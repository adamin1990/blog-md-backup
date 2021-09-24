# Android Handler原理 -Handler

------

​	Handler允许发送和处理与线程的MessageQueue相关联Message和Runnable对象。每一个Handler实例都与一个单独的线程和该线程的消息队列相关联。当创建Handler的时候，它被它创建的线程绑定。它把Message或Runnable对象传递到消息队列，出队的时候执行它们。Handler有两个主要用处：1.调度消息和Runnable在未来的某个时间点执行。2.入队一个操作并在不同的线程上执行。

​	调度消息通过post，postAtTime(Runnable,long),postDelayed,sendEmptyMessage,sendMessage,sendMessageAtTime,sendMessageDelayed方法完成。post版本允许入队Runnable对象，sendMessage版本允许入队Message对象，Message对象包含的数据会被Handler的handleMessage方法处理（需要自己实现Handler的子类）。

​	post或send的时候可以允许item在消息队列准备好的时候立即处理，或者定义一个时间延迟或确切的时间处理。

​	当一个应用的进程创建完，它的主线程专注运行一个消息队列,负责管理高级别的应用对象（Activity，BroadcastReceiver等）和他们创建的window，我们可以创建自己的子线程，并通过Handler和主线程通信。这样是通过在子线程里调用post或sendMessage方法完成。然后Runnable或Message将在Handler的消息队列里调度，在合适的时候处理。

### 构造函数

Handler总共有7个构造函数

~~~
#无参构造函数，内部调用6，callback传null，async传false
1. public Handler() {
        this(null, false);
    }
#设置消息处理回调，内部调用6，async传false
2. public Handler(@Nullable Callback callback) {
        this(callback, false);
    }
  #设置Looper，这样就不用调用Looper.prepare方法，内部调用7
3. public Handler(@NonNull Looper looper) {
        this(looper, null, false);
    }
    #设置Looper和回调，内部调用7
4. public Handler(@NonNull Looper looper, @Nullable Callback callback) {
        this(looper, callback, false);
    }
    #设置消息是否异步，内部调用6
5. public Handler(boolean async) {
        this(null, async);
    }
    #设置回调和是否异步，内部检测looper是否为空，所以所有内部调用这个方法的构造函数都需要先调用Looper.prepare方法初始化Looper和MessageQueue
6. public Handler(@Nullable Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread " + Thread.currentThread()
                        + " that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
 #设置Looper，回调，是否异步   
7. public Handler(@NonNull Looper looper, @Nullable Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }


~~~

​	Handler总共有7个构造函数，分别应用在不同场景下，可以设置消息是否异步，设置Looper，回调。

### 消息处理 

消息处理使用dispatchMessage方法，在Looper的loop中，获取到Message以后，通过**msg.target.dispatchMessage**调用，msg.target是Handler实例。

~~~
   private static void handleCallback(Message message) {
        message.callback.run();
    }
        public void handleMessage(@NonNull Message msg) {
    }
    
    /**
     * Handle system messages here.
     */
    public void dispatchMessage(@NonNull Message msg) {
        if (msg.callback != null) { //优先处理Message的callback
            handleCallback(msg);
        } else {
            if (mCallback != null) {  //其次处理Handler的Callback对象
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            //msg.callback和Handler的Callback对象都是空的，走Handler的handleMessage方法
            handleMessage(msg);
        }
    }
~~~

从上面可以看出消息处理的优先级是**msg.callback>Handler的mCallback>Handler的handleMessage方法**。msg.callback可以在构造Message的时候设置，mCallback可以在上述构造函数中**2，4，6，7**中设置，handleMessage方法可以在我们继承Handler的时候重写该方法。

### 发送消息 

​	文章开头讲过，发送消息主要有两个版本，post和send。post版本会传递一个Runnable对象，封装成Message的callback，然后加入消息队列，这样post版本会在dispatchMessage的时候，优先执行msg.callback。send版本会直接传递一个Message对象。

​	不论是post版本还是send版本，都会调用Handler的**sendMesssageAtTime**方法：

~~~
 public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }

~~~

​	然后再调用Handler的enqueueMessage方法

~~~
    private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
            long uptimeMillis) {
        msg.target = this;
        msg.workSourceUid = ThreadLocalWorkSource.getUid();

        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
~~~

最终调用MessageQueue的enqueueMessage方法，把消息插入到消息队列。

### 移除消息

------



