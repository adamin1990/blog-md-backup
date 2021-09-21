# Android Handler原理 -Handler

------

​	Handler允许发送和处理与线程的MessageQueue相关联Message和Runnable对象。每一个Handler实例都与一个单独的线程和该线程的消息队列相关联。当创建Handler的时候，它被它创建的线程绑定。它把Message或Runnable对象传递到消息队列，出队的时候执行它们。Handler有两个主要用处：1.调度消息和Runnable在未来的某个时间点执行。2.入队一个操作并在不同的线程上执行。

​	调度消息通过post，postAtTime(Runnable,long),postDelayed,sendEmptyMessage,sendMessage,sendMessageAtTime,sendMessageDelayed方法完成。post版本允许入队Runnable对象，sendMessage版本允许入队Message对象，Message对象包含的数据会被Handler的handleMessage方法处理（需要自己实现Handler的子类）。

​	post或send的时候可以允许item在消息队列准备好的时候立即处理，或者定义一个时间延迟或确切的时间处理。

​	当一个应用的进程创建完，它的主线程专注执行一个消息队列