# Android Handler原理-Looper

------



​	Looper在Handler机制中主要在线程中进行消息循环。线程中默认没有关联的Looper。在线程调用**Looper.prepare**方法创建关联的looper，调用**Looper.loop**方法处理消息，直到loop停止。大部分与消息循环的交互是通过**Handler**类。官方文档给出了一个简单实例实现了一个Looper线程，使用prepare和loop常见一个Handler与Looper通信。

~~~
 class LooperThread extends Thread {
         public Handler mHandler;
   
         public void run() {
             Looper.prepare();
   
             mHandler = new Handler() {
                 public void handleMessage(Message msg) {
                     // process incoming messages here
                 }
             };
   
             Looper.loop();
         }
     }
~~~

### Looper.prepare

通过ThreadLocal存储当前线程的Looper实例

~~~
#线程本地变量存储Looper实例
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
final MessageQueue mQueue;#关联的消息队列
final Thread mThread; #创建Looper的线程 
public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) { 
        #prepare只允许调用一次 
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed)); #新实例创建，存入本地线程变量副本
    }
      private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed); #消息队列实例化
        mThread = Thread.currentThread(); #创建的线程
    }

~~~

preprare参数quitAllowed代表跟Looper关联的MessageQueue是否允许退出，默认允许，Looper实例化的同时，也实例化了关联的MessageQueue，给线程变量mThread赋值，主线程的MessageQueue是不允许退出的，在ActivityThread的main方法中调用**Looper.prepareMainLooper**方法，内部prepare传参为false，表示不允许退出。

### 消息循环，Looper.loop

~~~
public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }
public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            #没有调用prepare方法，线程本地变量获取不到looper实例，直接抛出异常 
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;
       ....
        boolean slowDeliveryDetected = false;

        for (;;) {
           #从MessageQueue取消息，
            Message msg = queue.next(); // might block
            if (msg == null) {
                // 没有消息，消息队列正在退出
                return;
            }

            try {
               #调用handler.dispathMessage方法传递消息
                msg.target.dispatchMessage(msg);
               ...
            } catch (Exception exception) {
               ...
                throw exception;
            } finally {
               ...
            }
        
            #回收到回收池内
            msg.recycleUnchecked();
        }
    }

~~~

​	loop方法首先从线程本地变量获取Looper实例，如果获取不到直接抛出异常，所以一定要先调用Looper.prepare 方法实例化Looper和MessageQueue.接着通过for循环不断从消息队列里面通过next方法取Message，next方法取不到消息的时候可能会阻塞。取到消息后通过msg的target(handler实例)的dispatchMessage方法处理消息。最后把消息回首到缓存池。

### 总结

Looper主要通过loop方法从MessageQueue中循环获取消息，然后交给Handler处理。

在线程中要先使用Looper.prepare方法创建Looper，再调用Looper.loop方法获取和处理消息。

Android主线程的Looper实在ActivitiyThread的main方法中通过Looper.prepareMainLooper初始化的。

 