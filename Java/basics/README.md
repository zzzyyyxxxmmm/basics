# basics
一些Java基础

# 还不懂的问题

## 为什么接口不能是protected或者是default

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

## DI and IOC


# 一些小问题

# JVM

## GC

### What is GC
GC is used to collect objects which is marked as garbage.

### Where we do GC
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/Java-Garbage-Collection.png" width="700" height="200">
</div>

**Young Generation:** Newly created objects start in the Young Generation. The Young Generation is further subdivided into an Eden space, where all new objects start, and two Survivor spaces, where objects are moved from Eden after surviving one garbage collection cycle. When objects are garbage collected from the Young Generation, it is a minor garbage collection event.

**Old Generation:** Objects that are long-lived are eventually moved from the Young Generation to the Old Generation. When objects are garbage collected from the Old Generation, it is a major garbage collection event.

**Permanent Generation:**: Metadata such as classes and methods are stored in the Permanent Generation. Classes that are no longer in use may be garbage collected from the Permanent Generation.

### Which Object will be marked as garbage
根搜索算法
通过一系列名为GC Roots的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径成为引用链，当一个对象到GC Roots没有任何引用链相连，则证明此对象是不可用的。根搜索算法中不可达的对象也并非是‘非死不可’的，暂时是‘缓刑’阶段，要真正判断一个对象死亡要经历两次标记过程：如果对象在进行根搜索后发现对象不可达，那它将会进行被第一次标记并且进行筛选，筛选的条件是此对象是否有必要执行finalize()方法。当对象没有覆盖finalize()方法或者finalize()方法已经被虚拟机掉用过，这两种情况都视为‘没有必要执行’。
如果对象被认为有必要执行finalize()方法，那么这个方法会被放置在一个名为F-Queue的队列之中，并在稍后由一条由虚拟机自动建立的、低优先级的Finalizer线程去执行。这里的‘执行’也只是指虚拟机会触发这个方法，但并不承诺一定会执行。
finalize()方法是对象逃脱死亡命运的最后一次机会，稍后GC会对F-Queue中的对象进行第二次小规模的标记，如果对象在finalize()中重新与引用链上的任何一个对象建立了关联，就会被移出‘即将回收’集合，如果没有移出，那就真的离死亡不远了。
finalize()方法只会被系统自动调用一次。

**GC Roots**

* 虚拟机栈（栈帧中的本地变量表）中的引用的对象。
* 方法区中的类静态属性引用的对象。
* 方法区中的常量引用的对象。
* 本地方法栈（jni）即一般说的Native的引用对象。

* 强引用：普遍存在的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象
* 软引用：描述一些还有用，但并非必须的对象。对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中并进行第二次回收
* 弱引用：被弱引用关联的对象只能生存到下一次垃圾收集发生之前
* 虚引用：一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是希望能在这个对象被回收器回收时收到一个系统通知。

### How to collect - Algorithm

**标记清除**
标记-清除算法将垃圾回收分为两个阶段：标记阶段和清除阶段。在标记阶段首先通过根节点，标记所有从根节点开始的对象，未被标记的对象就是未被引用的垃圾对象。然后，在清除阶段，清除所有未被标记的对象。标记清除算法带来的一个问题是会存在大量的空间碎片，因为回收后的空间是不连续的，这样给大对象分配内存的时候可能会提前触发full gc。

**复制算法**
将现有的内存空间分为两快，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中，之后，清除正在使用的内存块中的所有对象，交换两个内存的角色，完成垃圾回收。
现在的商业虚拟机都采用这种收集算法来回收新生代，IBM研究表明新生代中的对象98%是朝夕生死的，所以并不需要按照1:1的比例划分内存空间，而是将内存分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中的一块Survivor。当回收时，将Eden和Survivor中还存活着的对象一次性地拷贝到另外一个Survivor空间上，最后清理掉Eden和刚才用过的Survivor的空间。HotSpot虚拟机默认Eden和Survivor的大小比例是8:1(可以通过-SurvivorRattio来配置)，也就是每次新生代中可用内存空间为整个新生代容量的90%，只有10%的内存会被“浪费”。当然，98%的对象可回收只是一般场景下的数据，我们没有办法保证回收都只有不多于10%的对象存活，当Survivor空间不够用时，需要依赖其他内存（这里指老年代）进行分配担保。

**标记整理**
复制算法的高效性是建立在存活对象少、垃圾对象多的前提下的。这种情况在新生代经常发生，但是在老年代更常见的情况是大部分对象都是存活对象。如果依然使用复制算法，由于存活的对象较多，复制的成本也将很高。
标记-压缩算法是一种老年代的回收算法，它在标记-清除算法的基础上做了一些优化。首先也需要从根节点开始对所有可达对象做一次标记，但之后，它并不简单地清理未标记的对象，而是将所有的存活对象压缩到内存的一端。之后，清理边界外所有的空间。这种方法既避免了碎片的产生，又不需要两块相同的内存空间，因此，其性价比比较高。

**增量算法**
增量算法的基本思想是，如果一次性将所有的垃圾进行处理，需要造成系统长时间的停顿，那么就可以让垃圾收集线程和应用程序线程交替执行。每次，垃圾收集线程只收集一小片区域的内存空间，接着切换到应用程序线程。依次反复，直到垃圾收集完成。使用这种方式，由于在垃圾回收过程中，间断性地还执行了应用程序代码，所以能减少系统的停顿时间。但是，因为线程切换和上下文转换的消耗，会使得垃圾回收的总体成本上升，造成系统吞吐量的下降。

**分代收集算法**
根据对象的存活周期的不同将内存分为几块。一是把Java堆分成新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高，没有额外空间对它进行分配担保，就必须使用标记清理或标记整理算法来进行回收

### How to collect - Collector
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/GC_Collector.png" width="400" height="400">
</div>

**Serial Collector**
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/ParNew_SerialOld.png" width="400" height="150">
</div>

Serial收集器是最古老的收集器，它的缺点是当Serial收集器想进行垃圾回收的时候，必须暂停用户的所有进程，即stop the world。到现在为止，它依然是虚拟机运行在client模式下的默认新生代收集器，与其他收集器相比，对于限定在单个CPU的运行环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾回收自然可以获得最高的单线程收集效率。
Serial Old是Serial收集器的老年代版本，它同样是一个单线程收集器，使用标记－整理“算法。

**ParNew收集器**

ParNew收集器是Serial收集器新生代的多线程实现，注意在进行垃圾回收的时候依然会stop the world，只是相比较Serial收集器而言它会运行多条进程进行垃圾回收。
ParNew收集器在单CPU的环境中绝对不会有比Serial收集器更好的效果，甚至由于存在线程交互的开销，该收集器在通过超线程技术实现的两个CPU的环境中都不能百分之百的保证能超越Serial收集器。当然，随着可以使用的CPU的数量增加，它对于GC时系统资源的利用还是很有好处的。它默认开启的收集线程数与CPU的数量相同，在CPU非常多（譬如32个，现在CPU动辄4核加超线程，服务器超过32个逻辑CPU的情况越来越多了）的环境下，可以使用-XX:ParallelGCThreads参数来限制垃圾收集的线程数。

**Parallel Scavenge收集器**

Parallel是采用复制算法的多线程新生代垃圾回收器，似乎和ParNew收集器有很多的相似的地方。但是Parallel Scanvenge收集器的一个特点是它所关注的目标是吞吐量(Throughput)。所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量=运行用户代码时间 / (运行用户代码时间 + 垃圾收集时间)。停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能够提升用户的体验；而高吞吐量则可以最高效率地利用CPU时间，尽快地完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

Parallel Old收集器是Parallel Scavenge收集器的老年代版本，采用多线程和”标记－整理”算法。这个收集器是在jdk1.6中才开始提供的，在此之前，新生代的Parallel Scavenge收集器一直处于比较尴尬的状态。原因是如果新生代Parallel Scavenge收集器，那么老年代除了Serial Old(PS MarkSweep)收集器外别无选择。由于单线程的老年代Serial Old收集器在服务端应用性能上的”拖累“，即使使用了Parallel Scavenge收集器也未必能在整体应用上获得吞吐量最大化的效果，又因为老年代收集中无法充分利用服务器多CPU的处理能力，在老年代很大而且硬件比较高级的环境中，这种组合的吞吐量甚至还不一定有ParNew加CMS的组合”给力“。直到Parallel Old收集器出现后，”吞吐量优先“收集器终于有了比较名副其实的应用祝贺，在注重吞吐量及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge加Parallel Old收集器。

**CMS收集器**

CMS(Concurrent Mark Swep)收集器是一个比较重要的回收器，现在应用非常广泛，我们重点来看一下，CMS一种获取最短回收停顿时间为目标的收集器，这使得它很适合用于和用户交互的业务。从名字(Mark Swep)就可以看出，CMS收集器是基于标记清除算法实现的。它的收集过程分为四个步骤：
* 初始标记(initial mark)
* 并发标记(concurrent mark)
* 重新标记(remark)
* 并发清除(concurrent sweep)
注意初始标记和重新标记还是会stop the world，但是在耗费时间更长的并发标记和并发清除两个阶段都可以和用户进程同时工作。

缺点：
1. CMS收集器对CPU资源 非常敏感（并发的缺点）
2. CMS收集器无法处理浮动垃圾(Floating Garbage)
CMS收集器无法处理浮动垃圾(Floating Garbage),可能出现“Concurrent Mode Failure”失败而导致另一次Full GC的产生。由于CMS并发清理阶段用户线程还在运行着,伴随程序运行自然就还会有新的垃圾不断产生,这一部分垃圾出现在标记过程之后,CMS无法在当次收集中处理掉它们,只好留待下一次GC时再清理掉。这一部分垃圾就称为“浮动垃圾”。也是由于在垃圾收集阶段用户线程还需要运行,那也就还需要预留有足够的内存空间给用户线程使用,因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集,需要预留一部分空间提供并发收集时的程序运作使用。在JDK 1.5的默认设置下,CMS收集器当老年代使用了68%的空间后就会被激活,这是一个偏保守的设置,如果在应用中老年代增长不是太快,可以适当调高参数-XX:CMSInitiatingOccupancyFraction的值来提高触发百分比,以便降低内存回收次数从而获取更好的性能,在JDK 1.6中,CMS收集器的启动阈值已经提升至92%。要是CMS运行期间预留的内存无法满足程序需要,就会出现一次“Concurrent Mode Failure”失败,这时虚拟机将启动后备预案:临时启用Serial Old收集器来重新进行老年代的垃圾收集,这样停顿时间就很长了。所以说参数-XX:CMSInitiatingOccupancyFraction设置得太高很容易导致大量“Concurrent Mode Failure”失败,性能反而降低。
3. CMS收集器会产生大量的空间碎片
CMS是一款基于“标记—清除”算法实现的收集器,这意味着收集结束时会有大量 空间碎片产生。空间碎片过多时,将会给大对象分配带来很大麻烦,往往会出现老年代还有很大空间剩余,但是无法找到足够大的连续空间来分配当前对象,不得不提前触发一次Full GC。为了解决这个问题,CMS收集器提供了一个-XX:+UseCMSCompactAtFullCollection开关参数(默认就是开启的),用于在CMS收集器顶不住要进行Full GC时开启内存碎片的合并整理过程,内存整理的过程是无法并发的,空间碎片问题没有了,但停顿时间不得不变长。虚拟机设计者还提供了另外一个参数-XX:CMSFullGCsBeforeCompaction,这个参数是用于设置执行多少次不压缩的Full GC后,跟着来一次带压缩的(默认值为0,表示每次进入Full GC时都进行碎片整理)。

**G1收集器**

G1收集器是一款面向服务端应用的垃圾收集器。HotSpot团队赋予它的使命是在未来替换掉JDK1.5中发布的CMS收集器。与其他GC收集器相比，G1具备如下特点：
并行与并发：G1能更充分的利用CPU，多核环境下的硬件优势来缩短stop the world的停顿时间。
分代收集：和其他收集器一样，分代的概念在G1中依然存在，不过G1不需要其他的垃圾回收器的配合就可以独自管理整个GC堆。
空间整合：G1收集器有利于程序长时间运行，分配大对象时不会无法得到连续的空间而提前触发一次GC。
可预测的非停顿：这是G1相对于CMS的另一大优势，降低停顿时间是G1和CMS共同的关注点，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。
在使用G1收集器时，Java堆的内存布局和其他收集器有很大的差别，它将这个Java堆分为多个大小相等的独立区域，虽然还保留新生代和老年代的概念，但是新生代和老年代不再是物理隔离的了，它们都是一部分Region（不需要连续）的集合。
虽然G1看起来有很多优点，实际上CMS还是主流。

# 内存溢出的种类
1. OutOfMemoryError： PermGen(永久代) space
2. OutOfMemoryError：  Java heap space
3. OutOfMemoryError：unable to create new native thread

# 类什么时候初始化

加载完类后，类的初始化就会发生，意味着它会初始化所有类静态成员，以下情况一个类被初始化：
* 实例通过使用new()关键字创建或者使用class.forName()反射，但它有可能导致ClassNotFoundException。
* 类的静态方法被调用
* 类的静态域被赋值
* 静态域被访问，而且它不是常量
* 在顶层类中执行assert语句

# Classloader

**JAVA类加载种类**

Java语言系统自带有三个类加载器: 
* Bootstrap ClassLoader 最顶层的加载类，主要加载核心类库，%JRE_HOME%\lib下的rt.jar、resources.jar、charsets.jar和class等。另外需要注意的是可以通过启动jvm时指定-Xbootclasspath和路径来改变Bootstrap ClassLoader的加载目录。比如java -Xbootclasspath/a:path被指定的文件追加到默认的bootstrap路径中。我们可以打开我的电脑，在上面的目录下查看，看看这些jar包是不是存在于这个目录。 
Int什么的都是由这个类加载的
* Extention ClassLoader 扩展的类加载器，加载目录%JRE_HOME%\lib\ext目录下的jar包和class文件。还可以加载-D java.ext.dirs选项指定的目录。 
* Appclass Loader也称为SystemAppClass 加载当前应用的classpath的所有类。是extension classloader的子类
我们上面简单介绍了3个ClassLoader。说明了它们加载的路径。并且还提到了-Xbootclasspath和-D java.ext.dirs这两个虚拟机参数选项。

**加载顺序**

我们看到了系统的3个类加载器，但我们可能不知道具体哪个先行呢？ 
我可以先告诉你答案 
1. Bootstrap CLassloder 
2. Extention ClassLoader 
3. AppClassLoader
   
**加载 流程**

1. 一个AppClassLoader查找资源时，先看看缓存是否有，缓存有从缓存中获取，否则委托给父加载器。 
2. 递归，重复第1部的操作。 
3. 如果ExtClassLoader也没有加载过，则由Bootstrap ClassLoader出面，它首先查找缓存，如果没有找到的话，就去找自己的规定的路径下，也就是sun.mic.boot.class下面的路径。找到就返回，没有找到，让子加载器自己去找。 
4. Bootstrap ClassLoader如果没有查找成功，则ExtClassLoader自己在java.ext.dirs路径中去查找，查找成功就返回，查找不成功，再向下让子加载器找。 
5. ExtClassLoader查找不成功，AppClassLoader就自己查找，在java.class.path路径下查找。找到就返回。如果没有找到就让子类找，如果没有子类会怎么样？抛出各种异常。

**为什么要使用classloader**
So, suppose a user calls his class java.lang.MyClass. Theoretically, it could get package access to all the fields and methods in the java.lang package and change the way they work. The language itself doesn't prevent this. But the JVM will block this, because all the real java.lang classes were loaded by bootstrap class loader. Not the same loader = no access.

There are other security features built into the class loaders that make it hard to do certain types of hacking.

So why three class loaders? Because they represent three levels of trust. The classes that are most trusted are the core API classes. Next are installed extensions, and then classes that appear in the classpath, which means they are local to your machine.

# static 加载顺序

如果类还没有被加载： 
1. 先执行父类的静态代码块和静态变量初始化，并且静态代码块和静态变量的执行顺序只跟代码中出现的顺序有关。 
2. 执行子类的静态代码块和静态变量初始化。 
3. 执行父类的实例变量初始化 
4. 执行父类的构造函数 
5. 执行子类的实例变量初始化 
6. 执行子类的构造函数 

# java8

## Stream

```java
filter
.distinct()
.takeWhile
.dropWhile
.limit(3) //select first 3 elements
.skip(2) //skip first 2 elements
```

### flatMap

the flatMap method lets you replace each value of a stream with another stream and then concatenates all the generated streams into a single stream

[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/flatmap.png)


```java
List<String> list = new ArrayList<>();
list.add("abcd");
list.add("bcd");
// return a list of all the unique characters for a list of words a b c d

//wrong, the final result is String[], we want string
List<String[]> result1 = list.stream().map(word -> word.split("")).distinct().collect(toList());
List<Stream<String>> result2 = list.stream().map(word -> word.split("")).map(Arrays::stream).distinct().collect(toList());

//use flat map
List<String> result3 = list.stream().map(word -> word.split("")).flatMap(Arrays::stream).distinct().collect(toList());

/*
2. Given two lists of numbers, how would you return all pairs of numbers? For example,
given a list [1, 2, 3] and a list [3, 4] you should return [(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)].
For simplicity, you can represent a pair as an array with two elements.
*/

List<Integer> numbers1 = Arrays.asList(1, 2, 3);
List<Integer> numbers2 = Arrays.asList(3, 4);
List<int[]> pairs =
          numbers1.stream()
               .flatMap(i -> numbers2.stream()
                         .map(j -> new int[]{i, j})
               )
               .collect(toList());
```

## find and match
```java
if(menu.stream().anyMatch(Dish::isVegetarian)) {
     System.out.println("The menu is (somewhat) vegetarian friendly!!");
}

boolean isHealthy = menu.stream().allMatch(dish -> dish.getCalories() < 1000);

boolean isHealthy = menu.stream().noneMatch(d -> d.getCalories() >= 1000);

Optional<Dish> dish = menu.stream().filter(Dish::isVegetarian).findAny();

menu.stream().filter(Dish::isVegetarian).findAny().ifPresent(dish -> System.out.println(dish.getName());

.findFirst()
/*
* isPresent() returns true if Optional contains a value, false otherwise.
* ifPresent(Consumer<T> block) executes the given block if a value is present. We introduced the Consumer functional interface in chapter 3; it lets you pass a
lambda that takes an argument of type T and returns void.
* T get() returns the value if present; otherwise it throws a NoSuchElement-
Exception.
*T orElse(T other) returns the value if present; otherwise it returns a default
value.
*/
```

## Reduce
```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
int sum = numbers.stream().reduce(0, Integer::sum);
Optional<Integer> sum = numbers.stream().reduce((a, b) -> (a + b));
Optional<Integer> max = numbers.stream().reduce(Integer::max);
```

## Numeric streams
```java
int calories = menu.stream().mapToInt(Dish::getCalories).sum();

IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
Stream<Integer> stream = intStream.boxed();

//IntStream 避免的boxing
IntStream evenNumbers = IntStream.rangeClosed(1, 100) .filter(n -> n % 2 == 0);
System.out.println(evenNumbers.count());

//直角三角形
IntStream.rangeClosed(1,100).boxed().flatMap(a->IntStream.rangeClosed(a,100).mapToObj(b->new Double[]{Double.valueOf(a), (double) b,Math.sqrt(a*a+b*b)}).filter(t->t[2]%1==0))
                .forEach(t ->
                        System.out.println(t[0] + ", " + t[1] + ", " + t[2]));
```

## Building streams
```java

Stream<String> stream = Stream.of("Modern ", "Java ", "In ", "Action"); stream.map(String::toUpperCase).forEach(System.out::println);

Stream<String> emptyStream = Stream.empty();

Stream<String> homeValueStream
            = Stream.ofNullable(System.getProperty("home"));

int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = Arrays.stream(numbers).sum();


//看下面三句，iterate是可以加predicate的，filter虽然过滤了，但stream还是会发送，可以用takewhile停止.limit也可以
IntStream.iterate(0, n -> n < 100, n -> n + 4)
         .forEach(System.out::println);

IntStream.iterate(0, n -> n + 4)
         .filter(n -> n < 100)
         .forEach(System.out::println);

IntStream.iterate(0, n -> n + 4)
         .takeWhile(n -> n < 100)
         .forEach(System.out::println);


Stream.generate(Math::random)
.limit(5)
      .forEach(System.out::println);


IntStream ones = IntStream.generate(() -> 1);


IntStream twos = IntStream.generate(new IntSupplier(){
            public int getAsInt(){
return 2; }
});



IntSupplier fib = new IntSupplier(){
    private int previous = 0;
    private int current = 1;
    public int getAsInt(){
        int oldPrevious = this.previous;
        int nextValue = this.previous + this.current;
        this.previous = this.current;
        this.current = nextValue;
        return oldPrevious;
} };
IntStream.generate(fib).limit(10).forEach(System.out::println);
```

## Collecting data with streams
Here are some example queries of what you’ll be able to do using collect and collectors:

* Group a list of transactions by currency to obtain the sum of the values of all transactions with that currency (returning a Map<Currency, Integer>)
* Partition a list of transactions into two groups: expensive and not expensive (returning a Map<Boolean, List<Transaction>>)
* Create multilevel groupings, such as grouping transactions by cities and then further categorizing by whether they’re expensive or not (returning a Map<String, Map<Boolean, List<Transaction>>>)

```java
Map<Currency, List<Transaction>> transactionsByCurrencies =
        transactions.stream().collect(groupingBy(Transaction::getCurrency));

Map<Dish.Type, List<Dish>> caloricDishesByType =
                            menu.stream().filter(dish -> dish.getCalories() > 500)
                                         .collect(groupingBy(Dish::getType));
//{OTHER=[french fries, pizza], MEAT=[pork, beef]}

Map<Dish.Type, List<Dish>> caloricDishesByType =
              menu.stream()
                  .collect(groupingBy(Dish::getType,
                           filtering(dish -> dish.getCalories() > 500, toList())));

Map<Dish.Type, List<String>> dishNamesByType =
      menu.stream()
          .collect(groupingBy(Dish::getType,
                   mapping(Dish::getName, toList())));
//{OTHER=[french fries, pizza], MEAT=[pork, beef], FISH=[]}

Map<Dish.Type, Long> typesCount = menu.stream().collect(
                    groupingBy(Dish::getType, counting()));

//{MEAT=3, FISH=2, OTHER=4}
```

## Parallel data processing and performance

## Collection API enhancements

### Arrays.asList vs List.of

前者是mutable,允许 null，但是两个都不能添加元素

```java
Map<String, Integer> ageOfFriends
   = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
```

Consider the following code, which tries to remove transactions that have a reference code starting with a digit:
```java
for (Transaction transaction : transactions) {
     if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
          transactions.remove(transaction);
     }
}

// 等于下面的

for (Iterator<Transaction> iterator = transactions.iterator();terator.hasNext()) {
   Transaction transaction = iterator.next();
   if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
       transactions.remove(transaction);
   }
}
```
Notice that two separate objects manage the collection:
* The Iterator object, which is querying the source by using next() and has- Next()
* The Collection object itself, which is removing the element by calling remove()
As a result, the state of the iterator is no longer synced with the state of the collection, and vice versa. To solve this problem, you have to use the Iterator object explicitly and call its remove() method:
```java
for (Iterator<Transaction> iterator = transactions.iterator();
             iterator.hasNext(); ) {
           Transaction transaction = iterator.next();
           if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
               iterator.remove();
           }
}

transactions.removeIf(transaction ->
             Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

Sometimes, though, instead of removing an element, you want to replace it. For this purpose, Java 8 added replaceAll.

```java
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) +
             code.substring(1));
```

### Working with Map

Two new utilities let you sort the entries of a map by values or keys:
* Entry.comparingByValue 
* Entry.comparingByKey

**Compute patterns**
* computeIfAbsent—If there’s no specified value for the given key (it’s absent or its value is null), calculate a new value by using the key and add it to the Map.
* computeIfPresent—If the specified key is present, calculate a new value for it and add it to the Map.
* compute—This operation calculates a new value for a given key and stores it in
the Map.

```java
friendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>())
              .add("Star Wars");
```

**Remove patterns**
```java
favouriteMovies.remove(key, value);
```

**Replacement patterns**
```java
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

## Default Method

ArrayList里的sort就是default method, 通过default method避免在更新时，强制要求实现类实现新加的方法

### 
[Java 浮点类型 float 和 double 的表示方法和范围](http://www.runoob.com/w3cnote/java-the-different-float-double.html)