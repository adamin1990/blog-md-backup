# 数据结构基础-栈（stack）

**栈**是一种简单的数据结构，添加和移除元素按照特定的顺序。每次添加元素，都会添加到栈的顶部，只有栈顶的元素才能被移除，这种表现叫**后进先出（LIFO）**，` last in first out `的英文缩写。

![image-stack](assets\image-stack.png)

向栈中添加新元素的操作叫**push**

从栈中移除元素的操作叫**pop**

### 实现栈

栈可以用**数组**或**链表**实现

### 栈数据结构实际应用中常见异常

使用栈我们经常碰到两种异常：

1. **stack overflow**

   向已满的栈中添加元素，造成栈溢出，top>=max-1

2. **stack underflow**

   从空栈中移除元素，造成堆栈下溢，top==-1

### java实现栈