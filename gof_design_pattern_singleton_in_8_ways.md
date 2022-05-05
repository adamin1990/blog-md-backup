# Java 实现单例模式的8种方式  
单例模式是GOF23种设计模式中最简单的一种，属于创建型模式分类，Java大概有8种实现单例模式的方式  
## 饿汉模式（静态变量）
```java
public class SingleTon {
    private static final SingleTon INSTANCE =new SingleTon();
    private SingleTon() {
    }
    public static SingleTon getInstance(){
        return  INSTANCE;
    }
}
```
该方式为非懒加载方式，线程安全   
## 饿汉模式 （静态代码块）  
```java
public class SingleTon {
    private  static  SingleTon INSTANCE; 
    static {
        INSTANCE =new SingleTon();
    }
    private SingleTon(){
        
    }
    public static SingleTon getInstance(){
        return  INSTANCE;
    }
}
```
该方式为非懒加载方式，线程安全   
## 懒汉模式（非线程安全）  
```Java
public class SingleTon {
    private SingleTon(){
        
    }
    private static SingleTon INSTANCE;
    public  static SingleTon getInstance(){
        if(INSTANCE==null){
            INSTANCE=new SingleTon();
        }
        return  INSTANCE;
    }
}
```  
该方式为懒加载方式，但是无法保证线程安全  
## 懒汉模式（同步方法实现线程安全）  
```java
public class SingleTon {
    private static SingleTon INSTANCE;
    private SingleTon(){

    }
    public static synchronized SingleTon getInstance(){
        if(INSTANCE==null){
            INSTANCE=new SingleTon();
        }
        return  INSTANCE;
    }
}
```  
该方式为懒加载方式，线程安全，但是多线程访问下效率不高，不推荐使用 

## 懒汉模式（同步代码块）  
```java
public class SingleTon {
    private static SingleTon INSTANCE;
    private SingleTon(){

    }
    public static SingleTon getInstance(){
        if(INSTANCE==null){
            synchronized (SingleTon.class){
                INSTANCE=new SingleTon();

            }
        }
        return  INSTANCE;
    }
}
```  
该方式是懒汉模式，但是无法真正实现线程安全，不推荐使用  

