# Anroid Handler原理-Message

​	Message翻译过来是消息，顾名思义，在Android中它定义了一个消息体，消息包含了描述和任意的数据对象，可以被发送给**Handler**。Message对象包含了两个int类型的额外字段，也包含一个object类型的额外字段，可以在多种情况下不分配。

​	尽管Message的构造器是public类型的，但最好的获取Message对象的方法是调用**Message.obtain()**方法或者**Handler.obtainMessage()**方法，这将会从对象回收池中拉取对象。

### 享元模式（FlyWeight）

享元模式是23种Gof设计模式结构化模式的一种，它在处理大量具有简单重复元素，单独存储会使用大量内存的时候非常有用，通常做法是把共享的数据存储在外部的数据结构里面，使用的时候把他们传递给对象。

**Handler.obtainMessage**方法内部也是调用的**Message.obtain**,所以只看后者：

~~~
    public static final Object sPoolSync = new Object();
    private static Message sPool;
    private static int sPoolSize = 0; #当前消息池的大小
    ....
    ....
    private static final int MAX_POOL_SIZE = 50;  #缓存消息池最大容量
  public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }
~~~

sPool如果是空，则new一个新的Message,非空的话从缓存池获取对象，并清空获取到的Message对象的flags,消息池当前大小减1。此外Message还有几个重载的obtain方法，分别对应不同场景对Message对象设置不同的属性，本质上都是从缓存池拿对象。

### 消息回收

Message回收共有两个方法，回收就是把Message对象放在全局的回收池，两个方法分别是：

##### recycleUnchecked()

~~~
 /**
     * Recycles a Message that may be in-use.回收一个可能在使用的消息
     * Used internally by the MessageQueue and Looper when disposing of queued Messages.处理完队列消息时在MessageQueue和Looper内部使用 
     */
    @UnsupportedAppUsage
    void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.清空其他的详细信息
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = UID_NONE;
        workSourceUid = UID_NONE;
        when = 0;
        target = null;
        callback = null;
        data = null;
   
        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
             #缓存池对象小于50，当前缓存池后移，当前对象放在链表首部，缓存池大小加一
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
~~~

recycleUnchecked方法可能回收一个正在使用的message，in-use表示在缓存池内，或者在消息队列里，只有在obtain的时候消息的状态才不是in-use。这个方法在Looper.loop循环获取到消息并dispatch以后会使用：

~~~
    public static void loop() {
        final Looper me = myLooper();
       .....
        me.mInLoop = true;
        final MessageQueue queue = me.mQueue;
       ....
        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
         ....
            try {
                msg.target.dispatchMessage(msg);
               ...
            } catch (Exception exception) {
             ...
            } finally {
               ...
            }
       
           ....
           #此处回收

            msg.recycleUnchecked();
        }
    }
~~~

在MessageQueue的多个remove方法中也会调用此方法。

##### recycle()

~~~
    private static boolean gCheckRecycle = true; #标识是否检测回收
    # Android版本低于Lollipop不检测回收 
    public static void updateCheckRecycle(int targetSdkVersion) {
        if (targetSdkVersion < Build.VERSION_CODES.LOLLIPOP) {
            gCheckRecycle = false;
        }
    }
    
       /*package*/ boolean isInUse() {
        return ((flags & FLAG_IN_USE) == FLAG_IN_USE);
    }

public void recycle() {
        if (isInUse()) { #检测时候在适使用 
            if (gCheckRecycle) {
                throw new IllegalStateException("This message cannot be recycled because it "
                        + "is still in use.");
            }
            return;
        }
        recycleUnchecked();
    }
~~~

recycle方法首先检测对象是否正在使用，是否在使用的状态上面已经提到了，入队正在处理，或者在回收池内。如果检测到正在使用，在判断gCheckRecycle是否检测回收，gCheckRecycle初始默认值是true，在updateCheckRecyle方法内判断修改其值，而updateCheckRecycle方法实在ActivityThread的handlebindApplication方法启动app时调用的：

~~~
    private void handleBindApplication(AppBindData data) {
        ....
        // send up app name; do this *before* waiting for debugger
        Process.setArgV0(data.processName);
        #这里对Message的gCheckRecycle修改
        Message.updateCheckRecycle(data.appInfo.targetSdkVersion);
    ...


    }

~~~

##### 异步消息

Message默认都是同步消息，在某些操作中，比如View的invalidate，会使用同步消息屏障，阻止后续消息达到某个条件才执行，异步消息能够让消息不受Looper的同步消息屏障的影响，免于消息屏障，异步消息的典型代表有中断、输入事件和其他即使有以他工作要被挂起也要单独执行的信号，如果消息被设置成异步的，会优先执行。Message中设置消息是否为异步的方法是**setAsynchronous（）**

~~~
public void setAsynchronous(boolean async) {
        if (async) {
            flags |= FLAG_ASYNCHRONOUS;
        } else {
            flags &= ~FLAG_ASYNCHRONOUS;
        }
    }
~~~

上面方法通过逻辑操作符修改flags，也提供了判断消息是否是异步的方法：

~~~
  public boolean isAsynchronous() {
        return (flags & FLAG_ASYNCHRONOUS) != 0;
    }
~~~

### 总结

Message是安卓消息传递机制信息的载体，他内部维护了一个链表，指向下一个，维护了一个缓存队列，最多缓存50个对象，避免重复创建对象，Message回收使用了享元模式的设计模式，Message回收会把flags标识成FLAG_IN_USE,消息入队也会标识成IN_USE,获取Message对象一般使用Message.obtain()方法，内部会清空flags标识。

Message默认是同步消息，遵循Looper的同步消息屏障，我们可以通过setAsynchronous方法把消息设置成异步，突破消息屏障，让它优先执行。
