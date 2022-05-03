# 面向对象软件开发的5大设计准的(SOLID)
# SOLID Priciple of Object-oriented software development Design 
------
## 单一职责原则 Single Responsibility Principle
A class should have one,and only one,reason to change.   
一个类应该有一个，且只有一个理由改变。  
It is one of the basic principles most developers apply to build robust and maintainable software.You can not only apply it to classes,buit also to software compoments and microservices.  
单一职责原色是大部分开发者应用构建健壮和可维护软件的基本准则之一。我们不仅可以把单一职责原则应用于类，并且可以应用到软件组件和微服务中。  
- Benefits of the Single Responsibility principle
- 单一职责原则的好处  
The argument for the single responsibility principle is relatively simple:it makes your software easier to implement and prevents unexpected sise-effects of future changes.
单一职责原则的论点相对简单：它使我们的软件更容易实现，并且防止软件未来变化的意外副作用。

## 开闭原则  Open/Closed Principle 
open/closed principle is the most important principle of object-oriented design.Software entities(classes,modules,functions,etc) should be open for extension,but closed for modification.  
开闭原则是面向对象设计最重要的原则，软件实体（类，模块，功能）应该对扩展开放，对修改闭合。  
It uses interfaces instead of superclasses to allow different implementations which you can easify substitude without changing the code that use them.The interfaces are closed for midifications,and you can provide new implementations to extend the functionality of your software.  
开闭原则使用接口而不是超类允许不同的实现，使我们不更改代码使用代码的情况下更容易的替换。接口对修改闭合，但是我们可以通过提供新的实现扩展软件的功能。这很好的体现了面向对象编程的多态性。  
The main benefit of this approach is that an interface introduces an additional level of abstraction which enables loose coupling. The implementations of an interface are independent of each other and don't need to share any code.  
开闭原则最大的好处是这些接口引入了一个额外的抽象级别，可以使松耦合，一个接口的实现独立于其他实现，不需要共享任何代码  

## 里氏替换原则（LSP）  Liskov Substitution Principle 

The Open/Closed Principle,is one of the key concepts in OOP that enables you to write robust,maintainable and reusable software components.But following the rules of that principle alone is not enough to ensure that you can change one part of your system without breaking other parts.your classes and interface also need to follow the liskov substitution principle to avoid and site-effects.  
开闭原则是面向对象的关键思想，使我们能够编写健硕的，可维护的可复用的软件组件。但是只遵循这条准则使我们改变软件系统中的部分而不破坏其他部分是不够的，我们的类和接口也需要遵循里氏替换原则。
It extends the Open/Closed Principle by focusing on the behavior of a superclass and its subtypes.
它通过聚焦超类和子类型的行为扩展了开闭原则。  
The principle defines that objects of a superclass shall be replaceable with objects of its subclassed without breaking the application.that requires the objects of your subclasses to behave in the same way as the objects of your superclass.  
里氏替换原则定义超类的对象应该可以在不破坏应用程序的前提下被其子类对象替代。这就需要子类的对象和超类对象表现方式相同。
- 遵循LSP案例 
```php 
<?php

interface Database 
{
    public function selectQuery(string $sql): array;
}

class SQLiteDatabase implements Database
{
    public function selectQuery(string $sql): array
    {
        // sqlite specific code

        return $result;
    }
}

class MySQLDatabase implements Database
{
    public function selectQuery(string $sql): array
    {
        // mysql specific code

        return $result; 
    }
}
```
- 违反LSP案例 
```PHP
<?php

interface Database 
{
    public function selectQuery(string $sql): array;
}

class SQLiteDatabase implements Database
{
    public function selectQuery(string $sql): array
    {
        // sqlite specific code

        return $result;
    }
}

class MySQLDatabase implements Database
{
    public function selectQuery(string $sql): array
    {
        // mysql specific code

        return ['result' => $result]; // This violates LSP !
    }
}
```

## 接口隔离原则  Interface Segration Principle  
Clients should not be forced to depend upon interfaces that they do not use.
客户端不应该被迫强制依赖他们不需要使用的接口。
By following this principle,you prevent bloated interfaces that define methods for multiple responsibilities.As explained in the single responsibility principle,you should avoid classes and interfaces with multiple responsibilities because they change ofen and make your software hard to maintain.  
通过遵循接口隔离原则，我们阻止了接口膨胀，避免了在同一接口中定义多个职责的方法。就像我们在单一职责原则中提到的，我们需要避免类和接口拥有多个职责，因为它们经常改变，会使得程序很难维护。  

## 依赖倒置原则  Dependency Inversion Principle 
DIP is based on Open/Closed Principle and Liskov substitution principle.The general idea of this principle is as simple as it is important:High-level modules,which provide complex logic,should be easily reusable and unaffected by changes in low-level modules,which provide utility features.To achieve that ,you need to introduce an abstraction that decouples the high-level and low-level modules from each other.  
依赖倒置原则基于开闭原则和里氏替换原则，他的只要思想是：提供复杂逻辑的高级别的模块应该容易复用，并且不受修改提供工具类特性的低级别模块的影响。我们需要提供一个抽象来解耦它们来实现。  
Dependency Inversion Principle consists of two parts:
- High-level modules should not depend on low-level modules.Both should depend on abstractions.
- 高级别模块不应该依赖低级别模块，它们都应依赖抽象  
- Abstractions should not depend on details,Details should depend on abstractions.
- 抽象不应该依赖具体，具体应该依赖抽象。  