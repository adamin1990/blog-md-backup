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
List
Queue
~~~



