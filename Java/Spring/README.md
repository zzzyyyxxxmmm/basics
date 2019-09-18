# 一些配置

## Spring context
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.3.13.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>RELEASE</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## IOC
In software engineering, inversion of control is a programming principle. IoC inverts the flow of control as compared to traditional control flow. In IoC, custom-written portions of a computer program receive the flow of control from a generic framework.

## DI
In software engineering, dependency injection is a technique whereby one object supplies the dependencies of another object. A "dependency" is an object that can be used, for example as a service. Instead of a client specifying which service it will use, something tells the client what service to use.

# Wiring beans
In Spring, objects aren’t responsible for finding or creating the other objects that they need to do their jobs. Instead, the container gives them references to the objects that they collaborate with.

The act of creating these associations between application objects is the essence of dependency injection (DI) and is commonly referred to as wiring. 

When it comes to expressing a bean wiring specification, Spring is incredibly flexible, offering three primary wiring mechanisms:
* Explicit configuration in XML
* Explicit configuration in Java
* Implicit bean discovery and automatic wiring

## Automatically wiring beans

Spring attacks automatic wiring from two angles:
* Component scanning—Spring automatically discovers beans to be created in the application context.
* Autowiring—Spring automatically satisfies bean dependencies.

```java
public interface CompactDisc {
    void play();
}
```

```java
@Component
public class SgtPeppers implements CompactDisc {
    private String title = "Sgt. Pepper's Lonely Hearts Club Band";
    private String artist = "The Beatles";

    public void play() {
        System.out.println("Playing " + title + " by " + artist);
    }
}
```

What you should take note of is that SgtPeppers is annotated with @Component. This simple annotation identifies this class as a component class and serves as a clue to Spring that a bean should be created for the class. There’s no need to explicitly configure a SgtPeppers bean; Spring will do it for you because this class is annotated with @Component.

Component scanning isn’t turned on by default, however. You’ll still need to write an explicit configuration to tell Spring to seek out classes annotated with @Component and to create beans from them. The configuration class in the following listing shows the minimal configuration to make this possible.

```java
@Configuration
@ComponentScan
public class CDPlayerConfig {
}
```

With no further configuration, @ComponentScan will default to scanning the same package as the configuration class. Therefore, because CDPlayerConfig is in the soundsystem package, Spring will scan that package and any subpackages underneath it, looking for classes that are annotated with @Component. It should find the Compact- Disc class and automatically create a bean for it in Spring.

If you’d rather turn on component scanning via XML configuration, then you can use the <context:component-scan> element from Spring’s context namespace. Here is a minimal XML configuration to enable component scanning.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=CDPlayerConfig.class)
public class CDPlayerTest {
    @Autowired
    private CompactDisc cd;

    @Test
    public void cdShouldNotBeNull(){
       assertNotNull(cd);
    }
}

//or

@Test
public void cdShouldNotBeNull(){
    ApplicationContext applicationContext=new AnnotationConfigApplicationContext(CDPlayerConfig.class);
    CompactDisc compactDisc=applicationContext.getBean(SgtPeppers.class);
    assertNotNull(compactDisc);
}
```

CDPlayerTest takes advantage of Spring’s SpringJUnit4ClassRunner to have a Spring application context automatically created when the test starts. And the @Context- Configuration annotation tells it to load its configuration from the CDPlayerConfig class. Because that configuration class includes @ComponentScan, the resulting applica- tion context should include the CompactDisc bean.

### Naming a component-scanned bean
If you’d rather give the bean a different ID, all you have to do is pass the desired ID as a value to the @Component annotation. For example, if you wanted to identify the bean as lonelyHeartsClub, then you’d annotate the SgtPeppers class with @Component like this:
```java
@Component("lonelyHeartsClub")
public class SgtPeppers implements CompactDisc {
... 
}
```

### Setting a base package for component scanning

To specify a different base package, you only need to specify the pack- age in @ComponentScan’s value attribute:

```java
@Configuration
@ComponentScan("soundsystem")
public class CDPlayerConfig {}

//or

//string is not type-safe
@Configuration
@ComponentScan(basePackages={"soundsystem", "video"})
public class CDPlayerConfig {}

//or

@Configuration
@ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class})
public class CDPlayerConfig {}

```

### Annotating beans to be automatically wired
The @Autowired annotation’s use isn’t limited to constructors. It can also be used on a property’s setter method. 

```java
@Autowired
public void setCompactDisc(CompactDisc cd) {
  this.cd = cd;
}

@Autowired
public void insertDisc(CompactDisc cd) {
  this.cd = cd;
}
//any method is ok
```

If there are no matching beans, Spring will throw an exception as the application context is being created. To avoid that exception, you can set the required attribute on @Autowired to false

## Wiring beans with Java

Although automatic Spring configuration with component scanning and automatic wiring is preferable in many cases, there are times when automatic configuration isn’t an option and you must configure Spring explicitly. For instance, let’s say that you want to wire components from some third-party library into your application. Because you don’t have the source code for that library, there’s no opportunity to annotate its classes with @Component and @Autowired. Therefore, automatic configuration isn’t an option.

### Creating a configuration class
```java
@Configuration
public class CDPlayerConfig {
}
```

The key to creating a JavaConfig class is to annotate it with @Configuration. The @Configuration annotation identifies this as a configuration class,
 and it’s expected to contain details on beans that are to be created in the Spring application context.

So far, you’ve relied on component scanning to discover the beans that Spring should create. Although there’s no reason you can’t use 
component scanning and explicit configuration together, we’re focusing on explicit configuration in this sec- tion, so I’ve removed the @ComponentScan annotation from CDPlayerConfig.

With @ComponentScan gone, the CDPlayerConfig class is ineffective. If you were to run CDPlayerTest now, the test would fail with a BeanCreationException. The test expects to be injected with CDPlayer and CompactDisc, 
but those beans are never cre- ated because 
they’re never discovered by component scanning.

### Declaring a simple bean
To declare a bean in JavaConfig, you write a method that creates an instance of the desired type and annotate it with @Bean. For example, the following method declares the CompactDisc bean:
```java
@Bean
public CompactDisc sgtPeppers() {
  return new SgtPeppers();
}

//@Bean(name="lonelyHeartsClubBand")
```

The @Bean annotation tells Spring that this method will return an object that should be registered as a bean in the Spring application context. 

By default, the bean will be given an ID that is the same as the @Bean-annotated method’s name. In this case, the bean will be named compactDisc. If you’d rather it have a different name, you can either rename the method or prescribe a different name with the name attribute.

## Wiring beans with XML

### Creating an XML configuration specification
Create ApplicationContext.xml in /resources 

### Declaring a simple <bean>
```xml
<bean id="compactDisc" class="soundsystem.SgtPeppers" />
```

and call it:

```java
@Test
public void cdShouldNotBeNull(){
    ApplicationContext applicationContext=new ClassPathXmlApplicationContext("ApplicationContext.xml");
    CompactDisc compactDisc=applicationContext.getBean(SgtPeppers.class);
   assertNotNull(compactDisc);
}
```

### Initializing a bean with constructor injection
With specific regard to constructor injection, you have two basic options to choose from:
* The<constructor-arg>element
* Using the c-namespace introduced in Spring 3.0

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer">
     <constructor-arg ref="compactDisc" />
</bean>

<bean id="cdPlayer" class="soundsystem.CDPlayer"
      c:cd-ref="compactDisc" />
```

What if the parameters of constructor is some kinds of value like **string**?
```xml
<bean id="compactDisc"
      class="soundsystem.BlankDisc">
  <constructor-arg value="Sgt. Pepper's Lonely Hearts Club Band" />
  <constructor-arg value="The Beatles" />
</bean>

<bean id="compactDisc"
      class="soundsystem.BlankDisc"
      c:_title="Sgt. Pepper's Lonely Hearts Club Band"
      c:_artist="The Beatles" />
      
<bean id="compactDisc"
    class="soundsystem.BlankDisc"
    c:_0="Sgt. Pepper's Lonely Hearts Club Band"
    c:_1="The Beatles" />

<!-- 参数只有一个的情况下-->
<bean id="compactDisc" class="soundsystem.BlankDisc"
      c:_="Sgt. Pepper's Lonely Hearts Club Band" />
```

**Collections**

```xml
<bean id="compactDisc" class="soundsystem.BlankDisc">
  <constructor-arg value="Sgt. Pepper's Lonely Hearts Club Band" />
  <constructor-arg value="The Beatles" />
  <constructor-arg><null/></constructor-arg>
</bean>

<bean id="compactDisc" class="soundsystem.BlankDisc">
  <constructor-arg value="Sgt. Pepper's Lonely Hearts Club Band" />
  <constructor-arg value="The Beatles" />
  <constructor-arg>
    <list>
      <value>Sgt. Pepper's Lonely Hearts Club Band</value>
      <value>With a Little Help from My Friends</value>
      <value>Lucy in the Sky with Diamonds</value>
      <value>Getting Better</value>
      <value>Fixing a Hole</value>
      <!-- ...other tracks omitted for brevity... -->
    </list>
  </constructor-arg>
</bean>

<constructor-arg>
    <list>
      <ref bean="sgtPeppers" />
      <ref bean="whiteAlbum" />
      <ref bean="hardDaysNight" />
      <ref bean="revolver" />
      ...
    </list>
```

### Setting properties
```xml
<bean id="cdPlayer"
      class="soundsystem.CDPlayer">
  <property name="compactDisc" ref="compactDisc" />
</bean>
```
如果里面是方法里面没提供constructor，只提供了set方法，那我们可以通过property注入

```xml

<bean id="cdPlayer"
      class="soundsystem.CDPlayer"
      p:compactDisc-ref="compactDisc" />
```

也可以自己独立创建一个Collection的bean
```xml
<util:list id="trackList">
  <value>Sgt. Pepper's Lonely Hearts Club Band</value>
  <value>With a Little Help from My Friends</value>
  <value>Lucy in the Sky with Diamonds</value>
  <value>Getting Better</value>
  <value>Fixing a Hole</value>
  <!-- ...other tracks omitted for brevity... -->
</util:list>


<bean id="compactDisc"
      class="soundsystem.BlankDisc"
      p:title="Sgt. Pepper's Lonely Hearts Club Band"
      p:artist="The Beatles"
      p:tracks-ref="trackList" />
```

## Importing and mixing configuration

### Referencing XML configuration in JavaConfig
Import 可以直接从其他config里导入相同的配置
```java
@Configuration
@Import({CDPlayerConfig.class, CDConfig.class})
public class SoundSystemConfig {
}
```

从xml里导入
```java
@Configuration
@Import(CDPlayerConfig.class)
@ImportResource("classpath:cd-config.xml")
public class SoundSystemConfig {
}
```

### Referencing JavaConfig in XML configuration
In XML, you can use the <import> element to split up the XML configuration. To import a JavaConfig class into an XML configuration, you declare it as a bean like this:

```xml
<bean class="soundsystem.CDConfig" />
<bean id="cdPlayer"
    class="soundsystem.CDPlayer"
    c:cd-ref="compactDisc" />
```

# Advanced wiring

## Environments and profile
1. Spring中的Profile 是什么?
Spring中的Profile功能其实早在Spring 3.1的版本就已经出来，它可以理解为我们在Spring容器中所定义的Bean的逻辑组名称，只有当这些Profile被激活的时候，才会将Profile中所对应的Bean注册到Spring容器中。举个更具体的例子，我们以前所定义的Bean，当Spring容器一启动的时候，就会一股脑的全部加载这些信息完成对Bean的创建；而使用了Profile之后，它会将Bean的定义进行更细粒度的划分，将这些定义的Bean划分为几个不同的组，当Spring容器加载配置信息的时候，首先查找激活的Profile，然后只会去加载被激活的组中所定义的Bean信息，而不被激活的Profile中所定义的Bean定义信息是不会加载用于创建Bean的。

2. 为什么要使用Profile
由于我们平时在开发中，通常会出现在开发的时候使用一个开发数据库，测试的时候使用一个测试的数据库，而实际部署的时候需要一个数据库。以前的做法是将这些信息写在一个配置文件中，当我把代码部署到测试的环境中，将配置文件改成测试环境；当测试完成，项目需要部署到现网了，又要将配置信息改成现网的，真的好烦。。。而使用了Profile之后，我们就可以分别定义3个配置文件，一个用于开发、一个用户测试、一个用户生产，其分别对应于3个Profile。当在实际运行的时候，只需给定一个参数来激活对应的Profile即可，那么容器就会只加载激活后的配置文件，这样就可以大大省去我们修改配置信息而带来的烦恼。

### Configuring profile beans
```java
@Configuration
public class DataSourceConfig {
  @Bean(destroyMethod="shutdown")
  @Profile("dev")       //Wired for “dev” profile
  public DataSource embeddedDataSource() {
      return new EmbeddedDatabaseBuilder()
          .setType(EmbeddedDatabaseType.H2)
          .addScript("classpath:schema.sql")
          .addScript("classpath:test-data.sql")
          .build();
}
  @Bean
  @Profile("prod") //Wired for “prod” profile
  public DataSource jndiDataSource() {
    JndiObjectFactoryBean jndiObjectFactoryBean =
        new JndiObjectFactoryBean();
} }
```

```xml
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd"
profile="dev">
<jdbc:embedded-database id="dataSource">
<jdbc:script location="classpath:schema.sql" />
<jdbc:script location="classpath:test-data.sql" />
</jdbc:embedded-database>


<beans profile="dev">
  <jdbc:embedded-database id="dataSource">
    <jdbc:script location="classpath:schema.sql" />
    <jdbc:script location="classpath:test-data.sql" />
  </jdbc:embedded-database>
</beans>
```

### Active Profile
There are several ways to set these properties:
1. As initialization parameters on DispatcherServlet 
2. As context parameters of a web application
3. As JNDI entries
4. As environment variables
5. As JVM system properties
6. Using the @ActiveProfiles annotation on an integration test class

## Conditional beans
Suppose you want one or more beans to be configured if and only if some library is available in the application’s classpath. Or let’s say you want a bean to be created only if a certain other bean is also declared. Maybe you want a bean to be created if and only if a specific environment variable is set.
Until Spring 4, it was difficult to achieve this level of conditional configuration, but Spring 4 introduced a new @Conditional annotation that can be applied to @Bean methods. If the prescribed condition evaluates to true, then the bean is created. Oth- erwise the bean is ignored.

感觉用的不是特别多，暂时先不看

## Addressing ambiguity in autowiring

```java
@Autowired
public void setDessert(Dessert dessert) {
    this.dessert = dessert;
}
// In this example, Dessert is an interface and is implemented by three classes: Cake, Cookies, and IceCream:
@Component
public class Cake implements Dessert { ... }
@Component
public class Cookies implements Dessert { ... }
@Component
public class IceCream implements Dessert { ... }
```
Because all three implementations are annotated by @Component, 
they’re all picked up during component-scanning and created as beans in the Spring application context. 
Then, when Spring tries to autowire the Dessert parameter in setDessert(), it doesn’t have a single, unambiguous choice.

### Designating a primary bean

```java
@Component
@Primary
public class IceCream implements Dessert { ... }

//configuration
@Bean
@Primary
public Dessert iceCream() {
    return new IceCream();
}

//xml
<bean id="iceCream"
      class="com.desserteater.IceCream"
      primary="true" />
```

### Qualifying autowired beans
```java
@Autowired
@Qualifier("iceCream")
public void setDessert(Dessert dessert) {
    this.dessert = dessert;
}
```

这样很明显的问题就是iceCream的ID改了之后，并不会refract这里，因此需要给component也加上Qualifier

```java
@Component
@Qualifier("cold")
public class IceCream implements Dessert { ... }
```

如果又有其他类用了@Qualifier("cold")呢，我们需要更精确：
```java
@Component
@Qualifier("cold")
@Qualifier("creamy")
public class IceCream implements Dessert { ... }
```

但Java不允许重复的annotation那我们可以自己定义一个：

```java
@Target({ElementType.CONSTRUCTOR, ElementType.FIELD,
         ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Cold { }


@Target({ElementType.CONSTRUCTOR, ElementType.FIELD,
         ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Creamy { }


@Component
@Cold
@Creamy
public class IceCream implements Dessert { ... }

```

## Scoping beans
By default, all beans created in the Spring application context are created as single- tons. That is to say, no matter how many times a given bean is injected into other beans, it’s always the same instance that is injected each time.

Spring defines several scopes under which a bean can be created, including the following:
* Singleton—One instance of the bean is created for the entire application.
* Prototype—One instance of the bean is created every time the bean is injected into or retrieved from the Spring application context.
* Session —In a web application, one instance of the bean is created for each session.
* Request—In a web application, one instance of the bean is created for each request.

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class Notepad { ... }


//xml
<bean id="notepad"
class="com.myapp.Notepad"
scope="prototype" />
```

## Runtime value injection

### Injecting external values

```java
@Configuration
@PropertySource("classpath:/com/soundsystem/app.properties")
public class ExpressiveConfig {
    @Autowired
    Environment env;
    @Bean
    public BlankDisc disc() {
        return new BlankDisc(
        env.getProperty("disc.title"),
        env.getProperty("disc.artist"));
    }    
}

//placeholder

//prerequisite


<context:property-placeholder />
<bean id="sgtPeppers"
      class="soundsystem.BlankDisc"
      c:_title="${disc.title}"
      c:_artist="${disc.artist}" />



@Bean
public
static PropertySourcesPlaceholderConfigurer placeholderConfigurer() {
  return new PropertySourcesPlaceholderConfigurer();
}

public BlankDisc(
      @Value("${disc.title}") String title,
      @Value("${disc.artist}") String artist) {
  this.title = title;
  this.artist = artist;
}
```

## Wiring with the Spring Expression Language
Spring 3 introduced Spring Expression Language (SpEL), a powerful yet succinct way of wiring values into a bean’s properties or constructor arguments using expressions that are evaluated at runtime. 

```java
public BlankDisc(
      @Value("#{systemProperties['disc.title']}") String title,
      @Value("#{systemProperties['disc.artist']}") String artist) {
  this.title = title;
  this.artist = artist;
}

#{3.14159}
#{false}
#{sgtPeppers.artist}
#{artistSelector.selectArtist()}
#{artistSelector.selectArtist()?.toUpperCase()}
T(java.lang.Math).PI
#{disc.title + ' by ' + disc.artist}
#{counter.total eq 100}
#{T(java.lang.Math).PI * circle.radius ^ 2}
#{scoreboard.score > 1000 ? "Winner!" : "Loser"}
#{jukebox.songs[4].title}
```

```xml
<bean id="sgtPeppers"
class="soundsystem.BlankDisc"
c:_title="#{systemProperties['disc.title']}"
c:_artist="#{systemProperties['disc.artist']}" />
```

# Aspect-oriented Spring
In software development, functions that span multiple points of an application are called cross-cutting concerns.

## What is aspect-oriented programming?
As stated earlier, aspects help to modularize cross-cutting concerns. In short, a cross- cutting concern can be described as any functionality that affects multiple points of an application. Security, for example, is a cross-cutting concern, in that many methods in an application can have security rules applied to them. Figure 4.1 gives a visual depic- tion of cross-cutting concerns.


Spring aspects can work with five kinds of advice:
* Before—The advice functionality takes place before the advised method is invoked.
* After—The advice functionality takes place after the advised method completes, regardless of the outcome.
* After-returning—The advice functionality takes place after the advised method successfully completes.
* After-throwing—The advice functionality takes place after the advised method throws an exception.
* Around—The advice wraps the advised method, providing some functionality before and after the advised method is invoked.

## Creating annotated aspects
```java
public interface Performance {
    public void perform();
}

@Aspect
public class Audience {
    @Before("execution(* concert.Performance.perform(..))")
    public void silenceCellPhones() {
        System.out.println("Silencing cell phones");
    }
    @Before("execution(* concert.Performance.perform(..))")
    public void takeSeats() {
        System.out.println("Taking seats");
    }
    @AfterReturning("execution(* concert.Performance.perform(..))")
    public void applause() {
        System.out.println("CLAP CLAP CLAP!!!");
    }
    @AfterThrowing("execution(* concert.Performance.perform(..))")
    public void demandRefund() {
        System.out.println("Demanding a refund");
    }
}
```

```java
@After
@AfterReturning
@AfterThrowing
@Around
@Before
```

可以看到上面excution四句都是相同的，想办法优化

```java
@Pointcut("execution(* concert.Performance.perform(..))")
public void performance() {}

@Before("performance()")
public void silenceCellPhones() {
    System.out.println("Silencing cell phones");
}
@Before("performance()")
public void takeSeats() {
    System.out.println("Taking seats");
}
@AfterReturning("performance()")
public void applause() {
    System.out.println("CLAP CLAP CLAP!!!");
}
@AfterThrowing("performance()")
public void demandRefund() {
    System.out.println("Demanding a refund");
}
```

使用

```java
@Configuration
@EnableAspectJAutoProxy
@ComponentScan
public class ConcertConfig {
  @Bean
  public Audience audience() {
    return new Audience();
  }
}
```

```xml
<aop:aspectj-autoproxy />
<bean class="concert.Audience" />
```

### Around

```java
public class Audience {
    @Pointcut("execution(** concert.Performance.perform(..))")
    public void performance() {
    }

    @Around("performance()")
    public void watchPerformance(ProceedingJoinPoint jp) {
        try {
            System.out.println("Silencing cell phones");
            System.out.println("Taking seats");
            jp.proceed();
            System.out.println("CLAP CLAP CLAP!!!");
        } catch (Throwable e) {
            System.out.println("Demanding a refund");
        }
    }
}
```

### Handling parameters in advice
可以看到，perform是包含参数的，如果里面有参数，我们又想统计这个参数呢

```java
@Aspect
public class TrackCounter {
    private Map<Integer, Integer> trackCounts =
            new HashMap<Integer, Integer>();

    @Pointcut(
            "execution(* soundsystem.CompactDisc.playTrack(int)) " +
                    "&& args(trackNumber)")
    public void trackPlayed(int trackNumber) {
    }

    @Before("trackPlayed(trackNumber)")
    public void countTrack(int trackNumber) {
        int currentCount = getPlayCount(trackNumber);
        trackCounts.put(trackNumber, currentCount + 1);
    }

    public int getPlayCount(int trackNumber) {
        return trackCounts.containsKey(trackNumber)

    }
}
```

## Annotating introductions
With Spring AOP, you can introduce new methods to a bean. A proxy intercepts the calls and delegates to a different object that provides the implementation.

let’s say you want to introduce the following Encoreable interface to any implementation of Performance:

```java
public interface Encoreable {
    void performEncore();
}
```

```java
@Aspect
public class EncoreableIntroducer {
  @DeclareParents(value="concert.Performance+",
                  defaultImpl=DefaultEncoreable.class)
  public static Encoreable encoreable;
}
```

The @DeclareParents annotation is made up of three parts:
* The value attribute identifies the kinds of beans that should be introduced with the interface. In this case, that’s anything that implements the Performance interface. (The plus sign at the end specifies any subtype of Performance, as opposed to Performance itself.)
* The defaultImpl attribute identifies the class that will provide the implemen- tation for the introduction. Here you’re saying that DefaultEncoreable will provide that implementation.
* The static property that is annotated by @DeclareParents specifies the inter- face that’s to be introduced. In this case, you’re introducing the Encoreable interface.

## Declaring aspects in XML

```xml
<aop:config>
<aop:aspect ref="audience"> Reference audience bean
    <aop:before
      pointcut="execution(** concert.Performance.perform(..))"
      method="silenceCellPhones"/>
    <aop:before
      pointcut="execution(** concert.Performance.perform(..))"
      method="takeSeats"/>

     <aop:after-returning pointcut="execution(** concert.Performance.perform(..))" method="applause"/>
<aop:after-throwing pointcut="execution(** concert.Performance.perform(..))" method="demandRefund"/>
  </aop:aspect>
</aop:config>


<aop:config>
  <aop:aspect ref="audience">
<aop:pointcut id="performance"
expression="execution(** concert.Performance.perform(..))" />
 <aop:before
   pointcut-ref="performance"
   method="silenceCellPhones"/>
   </aop:aspect>
   </aop:config>
  
   
   
<!--Declaring around advice-->
<aop:config>
  <aop:aspect ref="audience">
    <aop:pointcut
        id="performance"
expression="execution(** concert.Performance.perform(..))" />
<aop:around advice pointcut-ref="performance"
method="watchPerformance"/>
  </aop:aspect>
</aop:config>


<!--Passing parameters to advice-->
<aop:config>
  <aop:aspect ref="trackCounter">
  <aop:pointcut id="trackPlayed" expression=
    "execution(* soundsystem.CompactDisc.playTrack(int))
        and args(trackNumber)" />


<!--Introducing new functionality with aspects-->
<aop:aspect>
  <aop:declare-parents
    types-matching="concert.Performance+"
    implement-interface="concert.Encoreable"
    default-impl="concert.DefaultEncoreable"
    />
</aop:aspect>
```

# Building Spring web applications

## Getting started with Spring MVC

### Following the life of a request

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/web_request.png" width="500" height="500">
</div>


1. When the request leaves the browser, it carries information about what the user is asking for. At the least, the request will be carrying the requested URL. But it may also carry additional data, such as the information submitted in a form by the user.
The first stop in the request’s travels is at Spring’s DispatcherServlet. Like most Java- based web frameworks, Spring MVC funnels requests through a single front controller servlet. A front controller is a common web application pattern where a single servlet delegates responsibility for a request to other components of an application to per- form actual processing. In the case of Spring MVC, DispatcherServlet is the front controller.
2. The DispatcherServlet’s job is to send the request on to a Spring MVC controller. A controller is a Spring component that processes the request. But a typical application may have several controllers, and DispatcherServlet needs some help deciding which controller to send the request to. So the DispatcherServlet consults one or more handler mappings to figure out where the request’s next stop will be. The handler mapping pays particular attention to the URL carried by the request when making its decision.
3. Once an appropriate controller has been chosen, DispatcherServlet sends the request on its merry way to the chosen controller D. At the controller, the request drops off its payload (the information submitted by the user) and patiently waits while the controller processes that information. (Actually, a well-designed controller per- forms little or no processing itself and instead delegates responsibility for the business logic to one or more service objects.)
The logic performed by a controller often results in some information that needs to be carried back to the user and displayed in the browser. This information is referred to as the model. But sending raw information back to the user isn’t suffi- cient—it needs to be formatted in a user-friendly format, typically HTML. For that, the information needs to be given to a view, typically a JavaServer Page (JSP).
4. One of the last things a controller does is package up the model data and identify the name of a view that should render the output. It then sends the request, along with the model and view name, back to the DispatcherServlet E.
5. So that the controller doesn’t get coupled to a particular view, the view name passed back to DispatcherServlet doesn’t directly identify a specific JSP. It doesn’t even necessarily suggest that the view is a JSP. Instead, it only carries a logical name that will be used to look up the actual view that will produce the result. The DispatcherServlet consults a view resolver F to map the logical view name to a spe- cific view implementation, which may or may not be a JSP.
6. Now that DispatcherServlet knows which view will render the result, the request’s job is almost over. Its final stop is at the view implementation G, typically a JSP, where it delivers the model data. The request’s job is finally done. The view will use the model data to render output that will be carried back to the client by the (not- so-hardworking) response object H.

## Accepting request input

* Query parameters 
* Form parameters 
* Path variables

### Taking query parameters
```java
///spittles/show?spittle_id=12345.
@RequestMapping(method=RequestMethod.GET)
public List<Spittle> spittles(
    @RequestParam("max",defaultValue=MAX_LONG_AS_STRING) long max,
    @RequestParam("count") int count) {
  return spittleRepository.findSpittles(max, count);
}
```

### Taking input via path parameters
```java
///spittles/12345
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(
    @PathVariable("spittleId") long spittleId,
    Model model) {
  model.addAttribute(spittleRepository.findOne(spittleId));
  return "spittle";
}
```

# Hitting the database with Spring and JDBC

## Using JDBC driver-based data sources

* DriverManagerDataSource—Returns a new connection every time a connec- tion is requested. Unlike DBCP’s BasicDataSource, the connections provided by DriverManagerDataSource aren’t pooled.
* SimpleDriverDataSource—Works much the same as DriverManagerData- Source except that it works with the JDBC driver directly to overcome class load- ing issues that may arise in certain environments, such as in an OSGi container.
* SingleConnectionDataSource—Returns the same connection every time a connection is requested. Although SingleConnectionDataSource isn’t exactly a pooled data source, you can think of it as a data source with a pool of exactly one connection.

## object-relational mapping
* Integrated support for Spring declarative transactions  Transparent exception handling
* Thread-safe, lightweight template classes
* DAO support classes
* Resource management
