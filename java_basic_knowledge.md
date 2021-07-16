# Java必备基础知识备忘

###### 8大数据类型

~~~
byte/8 char/16 short/16 int/32 float/32 long/64 double/64 boolean/~
~~~

###### new Integer(2333)和 Integer.valueOf(2333)区别

~~~
使用new关键字创建每次都会创建一个新对象，而使用Integer.valueOf会从缓存池获取引用对象，没有才会去创建
~~~

###### String JDK1.8和1.9的区别 

~~~
Jdk1.8和之前版本String使用char数组存储，1.9及之后使用byte数组和coder标识编码类型实现
~~~

###### String不可变的好处

~~~
1.缓存hash值 2.String pool需要  3.安全性（网络传输） 4.线程安全
~~~

###### String StringBuffer StringBuilder对比

~~~
可变性方面：
String不可变，StringBuffer和StringBuilder可变 
线程安全方面：
String和StringBuffer安全,StringBuffer内部使用sychronized关键字同步，StringBuilder不安全
~~~

###### final关键字修饰的数据，方法，类

~~~
修饰数据：
对于基本数据类型，使用final关键字修饰后不可改变
对于引用类型，引用本身不能被改变，也就是不能指向其他对象，但是引用的对象本身内部可以改变。
修饰方法：
使用final修饰的方法不能被子类重写
修饰类：
类不允许被继承
~~~

###### 浅拷贝和深拷贝

~~~
浅拷贝拷贝的对象和原始对象引用同一个对象，拷贝对像改变也会改变原始对象
深拷贝拷贝对象和原始对象应用不同对象，
~~~

###### 泛型类型擦除

~~~
Java中泛型的工作原理是类型擦除，编译器在编译时会擦除类型相关信息，运行时不存在类型相关信息
~~~

###### 泛型中的限定通配符和非限定通配符

~~~
限定通配符有两种：
<? extend T> 指定泛型的上界，必须是T类型的子类
<? super T> 指定泛型下界，类型必须是T类型的父类
非限定通配符：
<?> 可以是任意类型
~~~

###### equals的五大特性

 ~~~
 自反性： x.equals(x) ==true;  x非空
 对称性： x.equals(y)==true 则 y.equals(x)==true x,y非空
 传递性： if(x.equals(y)&&y.equals(z)) 则 x.equals(z)==true  x,y,z非空
 一致性： 多次调用x.equals(y)返回值始终一致
 非空性： 对任意非空x，x.equals(null) 始终返回false
 ~~~

###### Java集合Collection

~~~
Set 无重复序列
	TreeSet 底层使用TreeMap存储，而TreeMap底层使用红黑树，有序，查找时间复杂度O(logN)
	HashSet 无需，可以实现快速查找，底层实现是哈希表,查找的时间复杂度O(1)
	LinkedHashSet，有HashSet的查找速度，内部使用双向链表维护顺序
List 可以有重复数据
	ArrayList基于动态数组实现，所以可以随机访问
	Vector，和ArrayList类似，但是线程安全
	LinkedList，基于双向链表实现，顺序访问，插入删除速度快，可以用作栈，队列，双向队列
Queue
	PriorityQueue，基于堆实现，可以实现优先级队列
	PriorityBlockingQueue，类似PriorityQueque，但是是线程安全的，操作使用ReentrantLock保持原子性。
~~~

###### Java映射Map

~~~
Map是具有键值对应关系的对象，不能重复
	HashMap:数据结构数组+链表（结合成哈希表），jdk1.8加入红黑树，默认桶大小是16，加载因子0.75，当单个链表超过8个元素的时候用链表转成红黑树
	HashTable：和HashMap类似，但是是线程安全的，使用synchronized
	ConcurrentHashMap:JDK1.7是分段锁，JDK1.8先使用CAS操作，如果CAS操作失败则使用系统自带的synchronized
	LinkedHashMap:继承自HashMap，内部维护双向链表维护插入顺序或LRU顺序
~~~

###### CAS原理，使用场景，缺点

~~~
compare and swap，比较和交换，是JDK提供的非阻塞原子性操作，首先获取预期值(当前变量最新值)A，然后进行CAS操作，和内存值V进行比较，如果A和V相等，就把当前变量改成B(更新值)，如果A和B不相等则不修改

CAS在java中的使用场景：
原子类，比如AtomicInteger，AtomicLong，AtomicBoolean
乐观锁
JDK1.8之后的ConcurrentHashMap


CAS缺点：
ABA问题，多线程需改同一个变量，线程1获取预期值A时候，线程2先把内存值改成了B,线程3又把内存值改成了A,这样线程1就认为A没有改变，解决ABA问题可以使用版本号比较
自旋时间过长占用资源
~~~



