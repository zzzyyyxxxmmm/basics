
# 交替打印AB
```java
package com.test.p2;

/**
 * @author jikangwang
 */
public class ParentRobot {
    private final Object object = new Object();

    public void run() {
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                for (; ; ) {
                    synchronized (object) {
                        try {
                            System.out.println("A");
                            object.notify();
                            object.wait();

                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        });
        Thread t2 = new Thread(new Runnable() {
            public void run() {
                for (; ; ) {
                    synchronized (object) {
                        try {
                            System.out.println("B");
                            object.notify();
                            object.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        });

        t1.start();
        t2.start();
    }

    public static void main(String[] args) {
        new ParentRobot().run();
    }
}

```

# Java虚拟机对synchronized的优化
锁的状态总共有四种，无锁状态、偏向锁、轻量级锁和重量级锁。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁，但是锁的升级是单向的，也就是说只能从低到高升级，不会出现锁的降级

## 偏向锁
偏向锁是Java 6之后加入的新锁，它是一种针对加锁操作的优化手段，经过研究发现，在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，因此为了减少同一线程获取锁(会涉及到一些CAS操作,耗时)的代价而引入偏向锁。偏向锁的核心思想是，如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作，从而也就提供程序的性能。所以，对于没有锁竞争的场合，偏向锁有很好的优化效果，毕竟极有可能连续多次是同一个线程申请相同的锁。但是对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样场合极有可能每次申请锁的线程都是不相同的，因此这种场合下不应该使用偏向锁，否则会得不偿失，需要注意的是，偏向锁失败后，并不会立即膨胀为重量级锁，而是先升级为轻量级锁。下面我们接着了解轻量级锁。
## 轻量级锁
倘若偏向锁失败，虚拟机并不会立即升级为重量级锁，它还会尝试使用一种称为轻量级锁的优化手段(1.6之后加入的)，此时Mark Word 的结构也变为轻量级锁的结构。轻量级锁能够提升程序性能的依据是“对绝大部分的锁，在整个同步周期内都不存在竞争”，注意这是经验数据。需要了解的是，轻量级锁所适应的场景是线程交替执行同步块的场合，如果存在同一时间访问同一锁的场合，就会导致轻量级锁膨胀为重量级锁。
## 自旋锁
轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。这是基于在大多数情况下，线程持有锁的时间都不会太长，如果直接挂起操作系统层面的线程可能会得不偿失，毕竟操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，因此自旋锁会假设在不久将来，当前的线程可以获得锁，因此虚拟机会让当前想要获取锁的线程做几个空循环(这也是称为自旋的原因)，一般不会太久，可能是50个循环或100循环，在经过若干次循环后，如果得到锁，就顺利进入临界区。如果还不能获得锁，那就会将线程在操作系统层面挂起，这就是自旋锁的优化方式，这种方式确实也是可以提升效率的。最后没办法也就只能升级为重量级锁了。
## 锁消除
消除锁是虚拟机另外一种锁的优化，这种优化更彻底，Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间，如下StringBuffer的append是一个同步方法，但是在add方法中的StringBuffer属于一个局部变量，并且不会被其他线程所使用，因此StringBuffer不可能存在共享资源竞争的情景，JVM会自动将其锁消除。


# Volatile
volatile两大作用
1. 保证内存可见性
2. 防止指令重排
 
## 指令重排

指令重排序是JVM为了优化指令，提高程序运行效率，在不影响单线程程序执行结果的前提下，尽可能地提高并行度。编译器、处理器也遵循这样一个目标。注意是单线程。多线程的情况下指令重排序就会给程序员带来问题。

不同的指令间可能存在数据依赖。比如下面计算圆的面积的语句：
```java
double r = 2.3d;//(1)
double pi =3.1415926; //(2)
double area = pi* r * r; //(3)
```
area的计算依赖于r与pi两个变量的赋值指令。而r与pi无依赖关系。
as-if-serial语义是指：不管如何重排序（编译器与处理器为了提高并行度），（单线程）程序的结果不能被改变。这是编译器、Runtime、处理器必须遵守的语义。
虽然，（1） - happensbefore -> （2）,（2） - happens before -> （3），但是计算顺序(1)(2)(3)与(2)(1)(3) 对于r、pi、area变量的结果并无区别。编译器、Runtime在优化时可以根据情况重排序（1）与（2），而丝毫不影响程序的结果。
指令重排序包括编译器重排序和运行时重排序。

### 例子1：A线程指令重排导致B线程出错
对于在同一个线程内，这样的改变是不会对逻辑产生影响的，但是在多线程的情况下指令重排序会带来问题。看下面这个情景:
在线程A中:
```java
context = loadContext();
inited = true;
```

在线程B中:
```java
while(!inited ){ //根据线程A中对inited变量的修改决定是否使用context变量
   sleep(100);
}
doSomethingwithconfig(context);
```

假设线程A中发生了指令重排序:
```java
inited = true;
context = loadContext();
```
 
那么B中很可能就会拿到一个尚未初始化或尚未初始化完成的context,从而引发程序错误。
 
### 例子2：指令重排导致单例模式失效
我们都知道一个经典的懒加载方式的双重判断单例模式：
```java
public class Singleton {
  private static Singleton instance = null;
  private Singleton() { }
  public static Singleton getInstance() {
     if(instance == null) {
        synchronzied(Singleton.class) {
           if(instance == null) {
               instance = new Singleton();  //非原子操作
           }
        }
     }
     return instance;
   }
}
``` 
看似简单的一段赋值语句：instance= new Singleton()，但是很不幸它并不是一个原子操作，其实际上可以抽象为下面几条JVM指令：
```java
memory =allocate();    //1：分配对象的内存空间 
ctorInstance(memory);  //2：初始化对象 
instance =memory;     //3：设置instance指向刚分配的内存地址 
```
 
上面操作2依赖于操作1，但是操作3并不依赖于操作2，所以JVM是可以针对它们进行指令的优化重排序的，经过重排序后如下：
```java
memory =allocate();    //1：分配对象的内存空间 
instance =memory;     //3：instance指向刚分配的内存地址，此时对象还未初始化
ctorInstance(memory);  //2：初始化对象
```
 
可以看到指令重排之后，instance指向分配好的内存放在了前面，而这段内存的初始化被排在了后面。
在线程A执行这段赋值语句，在初始化分配对象之前就已经将其赋值给instance引用，恰好另一个线程进入方法判断instance引用不为null，然后就将其返回使用，导致出错。

当且仅当满足以下所有条件时，才应该使用volatile变量：
1、 对变量的写入操作不依赖变量的当前值，或者你能确保只有单个线程更新变量的值。
2、该变量没有包含在具有其他变量的不变式中。
