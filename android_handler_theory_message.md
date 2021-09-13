# Anroid Handler原理-Message

​	Message翻译过来是消息，顾名思义，在Android中它定义了一个消息体，消息包含了描述和任意的数据对象，可以被发送给**Handler**。Message对象包含了两个int类型的额外字段，也包含一个object类型的额外字段，可以在多种情况下不分配。

​	尽管Message的构造器是public类型的，但最好的获取Message对象的方法是调用**Message.obtain()**方法或者**Handler.obtainMessage()**方法，这将会从对象回收池中拉取对象。

	### 享元模式（FlyWeight）

享元模式是23种Gof设计模式结构化模式的一种，它在处理大量具有简单重复元素，单独存储会使用大量内存的时候非常有用，通常做法是把共享的数据存储在外部的数据结构里面，使用的时候把他们传递给对象。
