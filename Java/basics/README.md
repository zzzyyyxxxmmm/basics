# basics
一些Java基础

# Object Oriented Programming

Object Oriented Programming (OOP) is a programming paradigm where the complete software operates as a bunch of objects talking to each other. An object is a collection of data and methods that operate on its data.

## Why OOP
For example, A car have an engine which correspond to variables in Java and a car is able to run which correspond to methods in Java. All of these characteristics increase the understanding of the software.

## Features of OOP
Encapsulation
Polymorphism
Inheritance

### Encapsulation
Encapsulation bundles data and methods together. For example, the car hava a engine, window and is able to run. These things are encapulated to make up a car. Besides, some components of car like enginee is hided by car to make it safe. In Java, access modifier restrict the access of component to guarantee the safety of class.

### Polymorphism
A car can take on different forms. It could be a taxi, a bus or a truck. But we don't need to care what the types of this car. All the things we need to know is this car is able to run. It has an engine, a window. In Java, we can create a databaseHelper to matipulate the database without knowing what type of the database is. If you want to change the database, the only thing you need to do is change the code where you create the database.

### Inheritance
The morden car inherits from old car like the carriage pulled by a horse. It reuse some features of carriage like morden car is able to run and have four wheels. But it also has some new features like engine to make it run faster than carriage. In java, inheritance is that a class is based on another class and uses data and implementation of the other class.
The purpose of inheritance is Code Reuse.

### SOLID

* Single Responsibility Principle: one class should have one and only responsibility.
* Open Closed Principle: Software components should be open for extension, but close for modification.
* Liskov's substitution principle: Drived types must be completely substitutable for their base types.
* Interface Segregation principle: Clients should not be forced to implement unneccessary methods which they will not use.
* Dependency Injection Principle: Depend on abstraction, not concretions.

## 使用回调有什么不好呢
1.	强引用，可能导致内存泄漏。
2.	多层回调的代码逻辑可读性差，调试时甚至使人抓狂。
3.	直接用匿名内部类去实现很容易就出现多级缩进，长长的屏幕都看不完一行代码。
4.	要跨线程必须要用Handler

## 抽象类和接口
general idea

1.	使用： Extend multiple class and implement one interface
2.	变量： interface: public static final var
3.  方法： 抽象方法都不可以被直接声明。接口中的的方法必须是public，为啥不能和abstract class一样可以是protected的呢？接口本身创建出来的作用之一就是给外部实现然后调用的。 抽象类的普通方法可以实现。 在Java7中不可以实现，Java8中可以通过default method实现。Java8中的接口静态方法只能通过接口名调用. Java9支持私有方法和私有静态方法

# 一些小问题

[Java 浮点类型 float 和 double 的表示方法和范围](http://www.runoob.com/w3cnote/java-the-different-float-double.html)