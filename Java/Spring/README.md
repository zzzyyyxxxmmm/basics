#一些配置

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

Although automatic Spring configuration with component scanning and automatic wiring is preferable in many cases, there are times when automatic configuration isn’t an option and you must configure Spring explicitly. For instance, let’s say that you want to wire components from some third-party library into your application. Because you don’t have the source code for that library, there’s no opportunity to annotate its classes with @Component and @Autowired. Therefore, automatic configu- ration isn’t an option.

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

By default, the bean will be given an ID that is the same as the @Bean-annotated method’s name. In this case, the bean will be named compactDisc. If you’d rather it have a different name, you can either rename the method or prescribe a different name with the name attribute