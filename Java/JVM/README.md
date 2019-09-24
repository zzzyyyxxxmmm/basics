# data types

the Java Virtual Machine operates on two kinds of types: primitive types and reference types. 

The primitive data types supported by the Java Virtual Machine are the numeric types, the boolean type, and the returnAddress type. 

The numeric types consist of the integral types and the floating-point types. 

There are three kinds of reference types: class types, array types, and interface types. 

# Run-Time Data Areas 
The Java Virtual Machine defines various run-time data areas that are used during execution of a program. Some of these data areas are created on Java Virtual Machine start-up and are destroyed only when the Java Virtual Machine exits. Other data areas are per thread. Per-thread data areas are created when a thread is created and destroyed when the thread exits. 

## The pc Register 
The Java Virtual Machine can support many threads of execution at once. Each Java Virtual Machine thread has its own pc (program counter) register. At any point, each Java Virtual Machine thread is executing the code of a single method, namely the current method for that thread. If that method is not native, the pc register contains the address of the Java Virtual Machine instruction currently being executed. If the method currently being executed by the thread is native, the value of the Java Virtual Machine's pc register is undefined. The Java Virtual Machine's pc register is wide enough to hold a returnAddress or a native pointer on the specific platform. 

## Java Virtual Machine Stacks 
Each Java Virtual Machine thread has a private Java Virtual Machine stack, created at the same time as the thread. A Java Virtual Machine stack stores frames. A Java Virtual Machine stack is analogous to the stack of a conventional language such as C: it holds local variables and partial results and plays a part in method invocation and return. Because the Java Virtual Machine stack is never manipulated directly except to push and pop frames, frames may be heap allocated. The memory for a Java Virtual Machine stack does not need to be contiguous. 

This specification permits Java Virtual Machine stacks either to be of a fixed size or to dynamically expand and contract as required by the computation. If the Java Virtual Machine stacks are of a fixed size, the size of each Java Virtual Machine stack may be chosen independently when that stack is created. 
The following exceptional conditions are associated with Java Virtual Machine stacks: 

• If the computation in a thread requires a larger Java Virtual Machine stack than is permitted, the Java Virtual Machine throws a StackOverflowError. 

• If Java Virtual Machine stacks can be dynamically expanded, and expansion is attempted but insufficient memory can be made available to effect the expansion, or if insufficient memory can be made available to create the initial Java Virtual Machine stack for a new thread, the Java Virtual Machine throws an OutOfMemoryError. 

## Heap 
The Java Virtual Machine has a heap that is shared among all Java Virtual Machine threads. The heap is the run-time data area from which memory for all class instances and arrays is allocated. 

The heap is created on virtual machine start-up. Heap storage for objects is reclaimed by an automatic storage management system (known as a garbage collector); objects are never explicitly deallocated. The Java Virtual Machine assumes no particular type of automatic storage management system, and the storage management technique may be chosen according to the implementor's system requirements. The heap may be of a fixed size or may be expanded as required by the computation and may be contracted if a larger heap becomes unnecessary. The memory for the heap does not need to be contiguous. 
The following exceptional condition is associated with the heap: 

## Method Area 
The Java Virtual Machine has a method area that is shared among all Java Virtual Machine threads. It stores class info, constant variable, static variable, complied code.
Although the method area is logically part of the heap, simple implementations may choose not to either garbage collect or compact it. The method area may be of a fixed size or may be expanded as required by the computation and may be contracted if a larger method area becomes unnecessary. The memory for the method area does not need to be contiguous. 

In HotSpot, Method Area is called permanent generation and should be collected by GC. After java1.8, PG is replaced by meta space and use native memory.

## Run-Time Constant Pool 
It belongs to Method Aread. A run-time constant pool is a per-class or per-interface run-time representation of the constant_pool table in a class file. It contains several kinds of constants, ranging from numeric literals known at compile-time to method and field references that must be resolved at run-time. The run-time constant pool serves a function similar to that of a symbol table for a conventional programming language, although it contains a wider range of data than a typical symbol table. 
Each run-time constant pool is allocated from the Java Virtual Machine's method area (§2.5.4). The run-time constant pool for a class or interface is constructed when the class or interface is created (§5.3) by the Java Virtual Machine. 

## Native Method Stacks 
Support Native Method

## Frames 
A frame is used to store data and partial results, as well as to perform dynamic linking, return values for methods, and dispatch exceptions. 

A new frame is created each time a method is invoked. A frame is destroyed when its method invocation completes, whether that completion is normal or abrupt (it throws an uncaught exception). Frames are allocated from the Java Virtual Machine stack of the thread creating the frame. Each frame has its own array of local variables, its own operand stack, and a reference to the run-time constant pool of the class of the current method. 

## Local Variables 
Each frame contains an array of variables known as its local variables. A single local variable can hold a value of type boolean, byte, char, short, int, float, reference, or returnAddress. A pair of local variables can hold a value of type long or double. 

A value of type long or type double occupies two consecutive local variables. Such a value may only be addressed using the lesser index. 

The Java Virtual Machine uses local variables to pass parameters on method invocation. On class method invocation, any parameters are passed in consecutive local variables starting from local variable 0. On instance method invocation, local variable 0 is always used to pass a reference to the object on which the instance method is being invoked (this in the Java programming language). Any parameters are subsequently passed in consecutive local variables starting from local variable.

## Operand Stacks 
Each frame contains a last-in-first-out (LIFO) stack known as its operand stack. 

The operand stack is empty when the frame that contains it is created. The Java Virtual Machine supplies instructions to load constants or values from local variables or fields onto the operand stack. Other Java Virtual Machine instructions take operands from the operand stack, operate on them, and push the result back onto the operand stack. The operand stack is also used to prepare parameters to be passed to methods and to receive method results. 

## Dynamic Linking 
Each frame contains a reference to the run-time constant pool for the type of the current method to support dynamic linking of the method code. The class file code for a method refers to methods to be invoked and variables to be accessed via symbolic references. Dynamic linking translates these symbolic method references into concrete method references, loading classes as necessary to resolve as-yet-undefined symbols, and translates variable accesses into appropriate offsets in storage structures associated with the run-time location of these variables. 

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

Minor GC: happen in young generation. Quick
Full GC: happen in old generation. Slow

**标记清除(Mark-Sweep)**
标记-清除算法将垃圾回收分为两个阶段：标记阶段和清除阶段。在标记阶段首先通过根节点，标记所有从根节点开始的对象，未被标记的对象就是未被引用的垃圾对象。然后，在清除阶段，清除所有未被标记的对象。标记清除算法带来的一个问题是会存在大量的空间碎片，因为回收后的空间是不连续的，这样给大对象分配内存的时候可能会提前触发full gc。

**复制算法(Copying)**
将现有的内存空间分为两快，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中，之后，清除正在使用的内存块中的所有对象，交换两个内存的角色，完成垃圾回收。
现在的商业虚拟机都采用这种收集算法来回收新生代，IBM研究表明新生代中的对象98%是朝夕生死的，所以并不需要按照1:1的比例划分内存空间，而是将内存分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中的一块Survivor。当回收时，将Eden和Survivor中还存活着的对象一次性地拷贝到另外一个Survivor空间上，最后清理掉Eden和刚才用过的Survivor的空间。HotSpot虚拟机默认Eden和Survivor的大小比例是8:1(可以通过-SurvivorRattio来配置)，也就是每次新生代中可用内存空间为整个新生代容量的90%，只有10%的内存会被“浪费”。当然，98%的对象可回收只是一般场景下的数据，我们没有办法保证回收都只有不多于10%的对象存活，当Survivor空间不够用时，需要依赖其他内存（这里指老年代）进行分配担保。

在被移动15次之后会进入老年代或者在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一般,年龄大于或等于该年龄的对象就可以直接进入老年代.

**标记整理(Mark-Compact)**
复制算法的高效性是建立在存活对象少、垃圾对象多的前提下的。这种情况在新生代经常发生，但是在老年代更常见的情况是大部分对象都是存活对象。如果依然使用复制算法，由于存活的对象较多，复制的成本也将很高。
标记-压缩算法是一种老年代的回收算法，它在标记-清除算法的基础上做了一些优化。首先也需要从根节点开始对所有可达对象做一次标记，但之后，它并不简单地清理未标记的对象，而是将所有的存活对象压缩到内存的一端。之后，清理边界外所有的空间。这种方法既避免了碎片的产生，又不需要两块相同的内存空间，因此，其性价比比较高。

**增量算法**
增量算法的基本思想是，如果一次性将所有的垃圾进行处理，需要造成系统长时间的停顿，那么就可以让垃圾收集线程和应用程序线程交替执行。每次，垃圾收集线程只收集一小片区域的内存空间，接着切换到应用程序线程。依次反复，直到垃圾收集完成。使用这种方式，由于在垃圾回收过程中，间断性地还执行了应用程序代码，所以能减少系统的停顿时间。但是，因为线程切换和上下文转换的消耗，会使得垃圾回收的总体成本上升，造成系统吞吐量的下降。

**分代收集算法(Generational Collection)**
根据对象的存活周期的不同将内存分为几块。一是把Java堆分成新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高，没有额外空间对它进行分配担保，就必须使用标记清理或标记整理算法来进行回收

### How to collect - Collector
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/GC_Collector.png" width="400" height="400">
</div>

| Collector    | Range            | Algorithm             | Type         |
|--------------|------------------|-----------------------|--------------|
| Serial       | Young Generation | Copying               | Single       |
| ParNew       | Young Generation | Copying               | parallel     |
| Parallel     | Young Generation | Copying               | parallel     |
| Serial Old   | Old Generation   | Mark-Compact          | Single       |
| CMS          | Old Generation   | Mark-Sweep            | Concurrent   |
| Parallel Old | Old Generation   | Mark-Compact          | Muilt-Thread |
| G1           | All              | Copying, Mark-Compact | Muilt-Thread |

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

**CMS(Concurrent Mark Sweep)收集器**

CMS(Concurrent Mark Sweep)收集器是一个比较重要的回收器，现在应用非常广泛，我们重点来看一下，CMS一种获取最短回收停顿时间为目标的收集器，这使得它很适合用于和用户交互的业务。从名字(Mark Swep)就可以看出，CMS收集器是基于标记清除算法实现的。它的收集过程分为四个步骤：
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

**G1(Garbage First)收集器**

G1收集器是一款面向服务端应用的垃圾收集器。HotSpot团队赋予它的使命是在未来替换掉JDK1.5中发布的CMS收集器。与其他GC收集器相比，G1具备如下特点：
* 并行与并发：G1能更充分的利用CPU，多核环境下的硬件优势来缩短stop the world的停顿时间。
* 基于mark-compact, 不会产生空间碎片
* 分代收集：和其他收集器一样，分代的概念在G1中依然存在，不过G1不需要其他的垃圾回收器的配合就可以独自管理整个GC堆。
* 空间整合：G1收集器有利于程序长时间运行，分配大对象时不会无法得到连续的空间而提前触发一次GC。
可预测的非停顿：这是G1相对于CMS的另一大优势，降低停顿时间是G1和CMS共同的关注点，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。
* G1将整个JAVA堆(包括新生代老年代)划分为多个大小固定的独立区域(region), 并且跟踪这些区域的垃圾堆积程度, 在后台维护一个优先列表, 每次根据允许的收集时间, 优先回收垃圾最多的区域

# JVM调优

## 常用工具
| name   | usage                                                                                                   |
|--------|---------------------------------------------------------------------------------------------------------|
| jps    | jvm process status tool，显示系统内所有的虚拟机进程                                                     |
| jstat  | jvm statistics monitoring tool，用于收集虚拟机各方面的运行数据                                          |
| jinfo  | configuration info for java，显示虚拟机参数配置信息                                                     |
| jmap   | memory map for java，生成虚拟机的内存转储快照（heapdump文件）                                           |
| jhat   | jvm heap dump browser，用于分析heapmap文件，它会建立一个http/html服务器让用户可以在浏览器上查看分析结果 |
| jstack | stack trace for java ，显示虚拟机的线程快照                                                             |
| jcmd   | java 命令行（stack trace for java），用于向正在运行的 JVM 发送诊断命令请求                              |

### JPS
列出所有虚拟机进程(LVMID + 名字)

jps [options] [hostid]

| Parameter | Explaination                                           |
|-----------|--------------------------------------------------------|
| -q        | 只输出进程ID的名称，省略主类的名称                     |
| -m        | 输出虚拟机进程启动时传递给主类main()函数的参数         |
| -l        | 输出主类的命名，如果进程执行的是Jar包，则输出Jar的路径 |
| -v        | 输出虚拟机进程启动时JVM参数                            |

### jstat
可以显示本地或远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行状况。

jstat option pid [interval [s|ms] [count]

| Parameter         | Explaination                                                                              |
|-------------------|-------------------------------------------------------------------------------------------|
| -class            | 监视类装载、卸载数量、总空间及类装载所耗费的时间                                          |
| -gc               | 监视Java堆状况，包括Eden区、2个survivor区、老年代、永久代等的容量、已用空间、GC时间等时间 |
| -gccapacity       | 监视内容与-gc相同，输出主要关注Java堆各个区域使用的最大空间和最小空间                     |
| -gcutil           | 监视内容与-gc基本相同，输出主要关注已使用空间占总空间的百分比                             |
| -gccause          | 与-gcutil功能一样，但会额外输出导致上一次GC产生的原因                                     |
| -gcnew            | 监视新生代GC的状况                                                                        |
| -gcnewcapacity    | 监视新生代GC的状况                                                                        |
| -gcold            | 监视老年代GC的状况                                                                        |
| -gcoldcapacity    | 监视内容与-gcold基本相同，输出主要关注使用的最大空间和最小空间                            |
| -gcpermcapacity   | 输出永久代使用的最大空间和最小空间                                                        |
| -compile          | 输出JIT编译器编译过的方法、耗时等信息                                                     |
| -printcompilation | 输出已经被JIT编译的方法                                                                   |
C:\Users\Administrator>jstat -gcutil 6716

  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT

  0.00  75.24  99.65  10.15  97.46  96.53      9    0.076     2    0.070    0.146

S0：Survivor1 区当前使用比例

S1：Survivor2 区当前使用比例 75.24

E：Eden区使用比例 99.65%

O：老年代使用比例 10.15%

M：元数据区使用比例

CCS：压缩使用比例

YGC：年轻代垃圾回收次数 9次

YGCT：年轻代垃圾回收总耗时

FGC：老年代垃圾回收次数 2次

FGCT：老年代垃圾回收消耗时间 0.070秒

GCT：垃圾回收消耗总时间 0.146秒

### jinfo

jinfo [option] pid

jinfo -flag CMSInitiatlingOccupancyFraction 1444

查询CMS..的信息
# 内存溢出的种类
1. OutOfMemoryError： PermGen(永久代) space
2. OutOfMemoryError：  Java heap space
3. OutOfMemoryError：unable to create new native thread

# 类什么时候初始化

加载完类后，类的初始化就会发生，意味着它会初始化所有类静态成员，以下情况一个类被初始化：
* 实例通过使用new()关键字创建或者使用class.forName()反射，但它有可能导致ClassNotFoundException。
* 类的静态方法被调用
* 类的静态域被赋值(读取或设置一个类的类的静态字段)
* 静态域被访问，而且它不是常量
* 在顶层类中执行assert语句
当然还有被动初始化

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