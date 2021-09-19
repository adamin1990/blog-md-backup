# Android Handler原理-MessageQueue(消息队列)

------

​	MessageQueue是一个低级别的类，持有一个由Looper传递的消息列表。Message不是直接添加到MessageQueue里的。而是由跟Looper关联的Handler对象添加的。

​	可以通过Looper.myQueue()方法提取跟当前线程关联的MessageQueue。

​	MessageQueue的构造函数：

~~~
  MessageQueue(boolean quitAllowed) {
        mQuitAllowed = quitAllowed;
        mPtr = nativeInit();
    }
~~~

**mQuitAllowed**表示是否允许退出，mPtr是navite函数的返回值，nativeInit位于C++层/frameworks/base/core/jni/android_os_MessageQueue.cpp文件内：

~~~
static jlong android_os_MessageQueue_nativeInit(JNIEnv* env, jclass clazz) {
    NativeMessageQueue* nativeMessageQueue = new NativeMessageQueue();
    if (!nativeMessageQueue) {
        jniThrowRuntimeException(env, "Unable to allocate native queue");
        return 0;
    }

   nativeMessageQueue->incStrong(env);
    return reinterpret_cast<jlong>(nativeMessageQueue);
}
~~~

首先实例化NaviteMessageQue，然后强制转换成long类型返回，那么mPtr就是一个指向NativeMessageQueue的指针，MessageQueue初始化的时候由nativeInit赋值，销毁的时候在finalizelize方法里调用dispose方法置零。

~~~
 private void dispose() {
        if (mPtr != 0) {
            nativeDestroy(mPtr);
            mPtr = 0;
        }
    }

~~~

回过头看构造函数，形参quitAllowed表示是否允许退出消息队列，主线程这个参数会传递成false，表示不允许退出，在ActivityThread的main方法里调用**Looper.prepareMainLooper()**,最终调用Lopper的构造函数，在构造函数里初始化MessageQueue，quitAllowed传参false。

~~~
  private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
~~~

我们平时调用Looper.prepare()，方法的时候，构造的MessageQueue默认是允许退出的。

### 消息入队

MessageQueue的消息入队方法是**enqueueMessage**,改方法在Handler里的**enqueueMessage**方法里调用。

~~~

    boolean enqueueMessage(Message msg, long when) {
        #判断入队消息对象的handler是否存在，不存在抛出异常
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        #判断消息是否正在使用（消息队列里或回收池里），使用的话抛出异常
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) { #正在退出，回首消息并返回false
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }
         #消息标记成使用中状态
            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            #当前队列头部消息是空，或者when是0（代表立即执行），或执行的时间小于当前队首执行的时间，则把该消息放到队首
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p; #队首消息放到当前消息的后面 
                mMessages = msg; #放到队首
                needWake = mBlocked; #阻塞的话需要唤醒
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                #在消息队列中插入消息，通常除了队首有同步平并且消息队列中最新消息是异步的情况下，我们不需要唤醒时间队队列
                needWake = mBlocked && p.target == null && msg.isAsynchronous();#先判断是否阻塞，再判断队首消息target是否是空，再判断当前消息是否异步
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {  #队尾或当前时间小于下条消息的时间，找到插入点 ，跳出循环
                        break;
                    }
                    if (needWake && p.isAsynchronous()) { #需要唤醒且下条消息是异步的，needWake复制false
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;  #插入消息 
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {  #需要唤醒
                nativeWake(mPtr); #native唤醒
            }
        }
        return true;
    }

~~~

消息入队根据when的先后顺序在队列的合适位置插入message。

### 消息出队 

~~~
 Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        #消息循环退出并disposed了立即返回
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) { #遇到同步屏障，循环读取，获取下一个异步消息，赋值给msg
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        #执行时间未到，计算下次获取超时时间
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message. 获取到和执行消息，阻塞状态清除，出队，并返回改消息
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.  没有消息了
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                #消息队列没有消息，或队首那条消息还没到处理时间（可能是一个同步屏障），处理闲时任务
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            #处理闲时任务
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {  #不保持，移除闲时任务 
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            #重置为0，不再次执行 
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            #处理闲时任务可能新的消息入队了，所以立即返回查看，不延迟等待
            nextPollTimeoutMillis = 0;
        }
    }

~~~

消息出队循环读取消息队列，碰到同步屏障后，优先读取异步消息并返回，当消息队列没有消息或当前消息还未到执行时间，会处理idleHandler任务。

### 同步屏障

同步屏障能够使异步消息优先执行，在上面的next方法已经分析过，MessageQueue的同步消息屏障的方法使postSyncBarrier.

~~~
 public int postSyncBarrier() {
        return postSyncBarrier(SystemClock.uptimeMillis());
    }

    private int postSyncBarrier(long when) {
        // Enqueue a new sync barrier token.
        // We don't need to wake the queue because the purpose of a barrier is to stall it.
        synchronized (this) {
            final int token = mNextBarrierToken++;
            final Message msg = Message.obtain();
            msg.markInUse();
            msg.when = when;
            msg.arg1 = token;

            Message prev = null;
            Message p = mMessages;
            if (when != 0) {
                while (p != null && p.when <= when) {
                    prev = p;
                    p = p.next;
                }
            }
            if (prev != null) { // invariant: p == prev.next
                msg.next = p;
                prev.next = msg;
            } else {
                msg.next = p;
                mMessages = msg;
            }
            return token;
        }
    }

~~~

postSyncBarrier方法首先构造一个Message对象，设置when和arg1属性，target属性为空，然后根据条件when入队，并返回自增的token。当遇到消息屏障的时候，后面的同步消息会被组织执行，直到调用removeSyncBarrier移除屏障。

同步消息屏障的典型使用场景是ViewRootImpl内，屏幕刷新时候的scheduleTravesals方法、unsheduleTraversals方法和doTravesal方法

~~~
void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            if (!mUnbufferedInputDispatch) {
                scheduleConsumeBatchedInput();
            }
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }

    void unscheduleTraversals() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
            mChoreographer.removeCallbacks(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        }
    }

    void doTraversal() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

            if (mProfile) {
                Debug.startMethodTracing("ViewAncestor");
            }

            performTraversals();

            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        }
    }
~~~

上面方法执行过程中添加和移除消息屏障。

​	

# 总结 

MessageQueue负责消息的入队（enQueueMessage）和出队（next），通过Handler间接操作入队，Looper.loop调用next方法循环出队，MessageQueue的初始化在Looper的prepare或prepareMainLooper方法中，mainLooper定义消息队列不可退出。MessageQueue内大部分操作是native层操作，基于epoll机制。同步屏障可以使异步消息优先执行。

MessageQueue内的IdleHandler可以注册为闲时执行，闲的意思使消息队列无消息，或者消息未达到执行时间。