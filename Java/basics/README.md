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

# What is JDK? JRE? JVM?
* JDK(Java Development Tools): The JDK is a development environment for building applications, applets, and components using the Java programming language. 
* JRE(Java Runtime Environment): It’s an environment for you to run Java application. 
* JVM(Java Virtual Machine): Run java bytecode.

Basically, JDK contains JRE, but with compiler tools and other programming tools. JRE contains JVM and other component to run applets and application written in Java.

## String为什么是final的

String是final并不仅仅只是指这个class是final的，而是代表这个类是不可修改的，里面的char array同样也是用final修饰的

类上声明final也是为了不让其他类继承从而破坏其**immutable**，immutable带来的好处就是安全

例如，String本身的touppercase是直接新建一个大写的类，然后指向它，如果继承String，改写其touppercase，使其是mutable的，那会改变常量池所有原本字符串的引用

首先看下几个例子：
```java
class Solution {
    
    public static String addStr(String s){
        s+="abc";
        return s;
    }

    public static StringBuilder addSB(StringBuilder stringBuilder){
        stringBuilder.append("abc");
        return stringBuilder;
    }

    public static void main(String[] args) {
        String s="1";
        addStr(s);
        System.out.println(s);

        StringBuilder stringBuilder=new StringBuilder("1");
        addSB(stringBuilder);
        System.out.println(stringBuilder);

    }
}
```

由于String是final的，因此不可以修改，String s 传递到方法里拷贝的是String的引用，然后s+="abc"实际是将拷贝的引用指向了新的常量，而不是在原来的基础上修改

另一个严重的问题：
```java
class Solution {

    public static void main(String[] args) {
        HashSet<StringBuilder> hs=new HashSet<>();
        StringBuilder sb1=new StringBuilder("aaa");
        StringBuilder sb2=new StringBuilder("aaabbb");
        hs.add(sb1);
        hs.add(sb2);
        StringBuilder sb3=sb1;
        sb3.append("bbb");
        System.out.println(hs);
    }
}
```
这个例子直接破坏了set的性质，会出现两个aaabbb

## 为什么wait/notify要放在synchonized里

wait通常是写在一个无限循环中，如果条件不成立则wait

```java
while(!condition){      //Thread A
    wait();
}

condition=true;         //Thread B
notify();

```

如果线程A执行到wait前，B刚好执行到notify的时候，那么A就会一直死循环了，加上syncronized就可以解决这个问题

## Why Java doen't support multi inheritance

C++ supports multi inheritance, so why Java doen't support multi inheritance?

Let's first look at one problem called Diamond Problem:

Suppose we have a class called A, and A containes a method called fun();
There have two classes B,C who inherite from class A.
So, if a class D extends B and C at the same time and then call method fun(). whose method will be called. B or C? C++ use virtual inheritance to solve this problem.

That is one of the problem of multi inheritance. However, In most cases, we doen't always need to use multi inheritance, so java developer think we need to 
simplify the design of Java. Although we lose some features of multi intheritence, we gain the simplicity. And java can implement multi inheritance by interface.

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