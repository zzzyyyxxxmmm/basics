# JPA and ORM
Java Persistence API is the implementation of ORM

# What is hibernate
Hibernate ORM (Hibernate in short) is an ORM tool for the Java programming language. It provides a framework for mapping an object-oriented domain model to a relational database. 

# The paradigm mismatch

### The problem of granularity
The support for user-define type is limitted. Therefore it's not a good idea to create a UDT in your database as a type.

### The problem of subtypes
SQL doesn't offer an inheritance mechanism. So you can't creat a debit billing details inherited from billing details.

### The problem of identity
Think: Is username appropriate to be primary key if there aren't duplicate names in this word?

The username in billing table will refer to username in user table as a foreign key, it will make it hard to change the username because of the integrity constraints.

### Problems relating to associations
It's easy to implement many-to-one association in java and database:
```java
public class User {
    Set billingDetails;
}
public class BillingDetails {
    User user;
}
```

```sql
create table USERS (
    USERNAME varchar(15) not null primary key,
    ADDRESS varchar(255) not null
);
create table BILLINGDETAILS (
    ACCOUNT varchar(15) not null primary key,
    BANKNAME varchar(255) not null,
    USERNAME varchar(15) not null,
    foreign key (USERNAME) references USERS
);
```

What if bill can be shared by different people?

It's still easy to implement many-to-many relation in java:
```java
public class User {
    Set billingDetails;
}
public class BillingDetails {
    Set users;
}
```
But we will find it's hard to do it in sql because now we need a conjunction table.

#

## @Entity and @Table
@Entity采用的默认名字的形式，类名即表名，成员变量名即表字段名

如果加上@Entity(name="abc"), 则其对应的表名为abc。

这个name也是JPA查询时候Entity的名字,如果有多个User什么的,就需要给另一个User取一个别名用于JPA的查询

例如创建一个类Course，数据库有个表也叫course，这时候就不需要给Entity加那么，但如果数据库中的表叫"course1",则entity需要加上name="course1"。

如果数据库的column名字也改了呢,则这时候只能使用@Table了,同样table也可以指定对应的表明，并且优先级高于Entity。@Column关键字可以用来指定字段名

映射规则：

1. 实体类必须用 @javax.persistence.Entity 进行注解；
2. 必须使用 @javax.persistence.Id 来注解一个主键；
3. 实体类必须拥有一个 public 或者 protected 的无参构造函数，之外实体类还可以拥有其他的构造函数；
4. 实体类必须是一个顶级类（top-level class）。一个枚举（enum）或者一个接口（interface）不能被注解为一个实体
5. 实体类不能是 final 类型的，也不能有 final 类型的方法；
6. 如果实体类的一个实例需要用传值的方式调用（例如，远程调用），则这个实体类必须实现（implements）java.io.Serializable 接口。

## @Column
不是constrains，而是用于DDL
constraint可以用@NotNull 或者Size一类的

## hibernate.ddl-auto
1. create: 每次加载hibernate时都会删除上一次的生成的表，然后根据你的model类再重新来生成新表，哪怕两次没有任何改变也要这样执行，这就是导致数据库表数据丢失的一个重要原因。 
2. create-drop: 每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。 
3. update: 最常用的属性，第一次加载hibernate时根据model类会自动建立起表的结构（前提是先建立好数据库），以后加载hibernate时根据 model类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。要注意的是当部署到服务器后，表结构是不会被马上建立起来的，是要等应用第一次运行起来后才会。 
4. validate: 每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。

## @GeneratedValue 
* GenerationType.AUTO—Hibernate picks an appropriate strategy, asking the SQL dialect of your configured database what is best. This is equivalent to @GeneratedValue() without any settings.
会从其他三种里选择一个
* GenerationType.SEQUENCE—Hibernate expects (and creates, if you use the tools) a sequence named HIBERNATE_SEQUENCE in your database. The sequence will be called separately before every INSERT, producing sequential numeric values.

Sequence一般是用在oracle里,mysql不支持sequence
* GenerationType.IDENTITY—Hibernate expects (and creates in table DDL) a special auto-incremented primary key column that automatically generates a numeric value on INSERT, in the database.

同样oracle也不支持自增
* GenerationType.TABLE—Hibernate will use an extra table in your database schema that holds the next numeric primary key value, one row for each entity class. This table will be read and updated accordingly, before INSERTs. The default table name is HIBERNATE_SEQUENCES with columns SEQUENCE_NAME and SEQUENCE_NEXT_HI_VALUE. (The internal implementation uses a more complex but efficient hi/lo generation algorithm; more on this later.)

这个需要自己建表来指定主键
## @GenericGenerator
一般用来生成UUID

## PhysicalNamingStrategy
如果表全部都是某个字段开头

# Mapping value types

## @Transient
entity和数据库的表如果对不上的话,entity里多的属性就会添加到数据库里, 如果不想添加,可以加上@Transient
如果数据库多的话,也不会报错,只是少显示了一个属性

## @Access
```java
@Access(AccessType.PROPERTY)
private String name;

public String getName() {
    return "***"+ name;
}
```
默认是通过field方式访问的, 注意保存的时候也会调用get

## @Formula
```java
@Formula("(select AVG(a.size) from Course a)")
protected BigDecimal averageBidAmount;

@Formula("substr(name,1,2)")
private String shortName;
```

## @ColumnTransformer
```java
@ColumnTransformer(read = "size/2",
write = "?*2" )
private Integer size;
```

## @Generated
```java
@Column(insertable = false)
@ColumnDefault("1.00") 
@Generated(GenerationTime.INSERT)
protected BigDecimal initialPrice;
```

## @Embeddable
假设User表的结构如下:
ID,USENAME,FIRSTNAME,STEET,ZIPCODE,CITY,BILLING_STREET,BILLING_ZIPCODE,BILLING_CITY
```java
@Embeddable
@ToString
public class Address {
    private String address;
}

@Embedded
@AttributeOverrides({
                @AttributeOverride(
                        name = "address",
                        column = @Column(name = "bill_address")
                )
        })
private Address billAddress;

private Address address;
```
在User表里的address里的annotation会覆盖address里的annotation
You can declare @AttributeOverrides at any level, as you do for the name property of the City class, mapping it to the CITY column. This can be achieved with either (as shown) an @AttributeOverride in Address or an override in the root entity class, User. Nested properties can be referenced with dot notation: for example, on User#address, @AttributeOveride(name = "city.name") references the Address #City#name attribute.

```java
@EmbeddedId
protected UserId id;
```
同样,这个也可以作为composite primary key.

## Creating custom JPA converters
```java
public class MonetaryAmount implements Serializable {
    protected final BigDecimal value;
    protected final Currency currency;

    public static MonetaryAmount fromString(String s) {
        String[] split = s.split(" ");
        return new MonetaryAmount(
        new BigDecimal(split[0]),
        Currency.getInstance(split[1])
       );
    }
}


@Converter(autoApply = true)
public class MonetaryAmountConverter implements AttributeConverter<MonetaryAmount, String> {
@Override
      public String convertToDatabaseColumn(MonetaryAmount monetaryAmount) {
        return monetaryAmount.toString();
    }
@Override
    public MonetaryAmount convertToEntityAttribute(String s) {
        return MonetaryAmount.fromString(s);
} 
}



@NotNull @Convert(converter = MonetaryAmountConverter.class, disableConversion = false) 
@Column(name = "PRICE", length = 63) 
protected MonetaryAmount buyNowPrice;
```

这段代码可以把11.23 USD or 99 EUR.转成类的形式

# Mapping inheritance

## @MappedSuperclass
```java
@MappedSuperclass
public abstract class BillingDetails { @NotNull
     protected String owner;
     // ...
}

@Entity @AttributeOverride(name = "owner", column = @Column(name = "CC_OWNER", nullable = false))   //当然也可以直接在父类上声明
public class CreditCard extends BillingDetails {

}
```

对于执行```select bd from BillingDetails bd```,
会执行下面查询:
```sql
select
    ID, OWNER, ACCOUNT, BANKNAME, SWIFT
from BANKACCOUNT
select
    ID, CC_OWNER, CARDNUMBER, EXPMONTH, EXPYEAR
from CREDITCARD
```
Hibernate uses a separate SQL query for each concrete subclass. On the other hand, queries against the concrete classes are trivial and perform well—Hibernate uses only one of the statements.

## @Inheritance
[Inheritance的三种类型](!https://blog.csdn.net/songmaolin_csdn/article/details/78517534)
per_table 会为每个继承他的表创建table
single_table就指回合并所有的子类然后创建一个表
joined就是会创建子类表和所有表,然后关联

基于多台查询的时候会返回List,需要自己进行类型转换

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS) public abstract class BillingDetails {
@Id
@GeneratedValue(generator = Constants.ID_GENERATOR) protected Long id;
@NotNull
protected String owner;
// ...
}


@Entity
public class CreditCard extends BillingDetails { @NotNull
protected String cardNumber; @NotNull
protected String expMonth; @NotNull
    protected String expYear;
    // ...
}
```
对于查询```select bd from BillingDetails bd```

```sql
select
    ID, OWNER, EXPMONTH, EXPYEAR, CARDNUMBER,
    ACCOUNT, BANKNAME, SWIFT, CLAZZ_
from
( select
          ID, OWNER, EXPMONTH, EXPYEAR, CARDNUMBER,
          null as ACCOUNT,
          null as BANKNAME,
           null as SWIFT,
          1 as CLAZZ_
      from
          CREDITCARD
      union all
select
          id, OWNER,
          null as EXPMONTH,
          null as EXPYEAR,
          null as CARDNUMBER,
          ACCOUNT, BANKNAME, SWIFT,
          2 as CLAZZ_
from
          BANKACCOUNT
    ) as BILLINGDETAILS
```
虽然都是创建多个表,但后者执行的是union,比上面要好

### 如何选择
1. If you don’t require polymorphic associations or queries, lean toward table-per- concrete class—in other words, if you never or rarely select bd from Billing- Details bd and you have no class that has an association to BillingDetails. An explicit UNION-based mapping with InheritanceType.TABLE_PER_CLASS should be preferred, because (optimized) polymorphic queries and associa- tions will then be possible later.
   
2. If you do require polymorphic associations (an association to a superclass, hence to all classes in the hierarchy with dynamic resolution of the concrete class at runtime) or queries, and subclasses declare relatively few properties (particularly if the main difference between subclasses is in their behavior), lean toward InheritanceType.SINGLE_TABLE. Your goal is to minimize the number of nullable columns and to convince yourself (and your DBA) that a denormalized schema won’t create problems in the long run.
   
3. If you do require polymorphic associations or queries, and subclasses declare many (non-optional) properties (subclasses differ mainly by the data they hold), lean toward InheritanceType.JOINED. Alternatively, depending on the width and depth of your inheritance hierarchy and the possible cost of joins versus unions, use InheritanceType.TABLE_PER_CLASS. This decision might require evaluation of SQL execution plans with real data.

# Mapping collections and entity associations

The collection you could create is Item#images, referencing all images for a partic-
ular item. You create and map this collection property to do the following:
* Execute the SQL query SELECT * from IMAGE where ITEM_ID = ? automatically when you call someItem.getImages(). As long as your domain model instances are in a managed state (more later), you can read from the database on-demand while navigating the associations between your classes. You don’t have to manu- ally write and execute a query with the EntityManager to load data. On the other hand, the collection query when you start iterating the collection is always “all images for this item,” never “only images that match criteria XYZ.”
* Avoid saving each Image with entityManager.persist(). If you have a mapped collection, adding the Image to the collection with someItem.getImages() .add() will make it persistent automatically when the Item is saved. This cascad- ing persistence is convenient because you can save instances without calling EntityManager.
* Have a dependent life cycle of Images. When you delete an Item, Hibernate deletes all attached Images with an extra SQL DELETE. You don’t have to worry about the life cycle of images and cleaning up orphans (assuming your database foreign key constraint doesn’t ON DELETE CASCADE). The JPA provider handles the composition life cycle.

## mapping a list
```java
@Entity
public class Item {
    @ElementCollection 
    @CollectionTable(name = "IMAGE", joinColumns = @JoinColumn(name = "ITEM_ID")) 
    @Column(name = "FILENAME")
}
```
ITEM:

| Id | Name |
|----|------|
| 1  | Foo  |
| 2  | B    |
| 3  | C    |

IMAGE:

| Item_Id | FileName |
|---------|----------|
| 1       | Foo.jpg  |
| 1       | bar.jpg  |
| 1       | baz.jpg  |
| 2       | b.jpg    |



```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE") 
    @Column(name = "FILENAME") 
    @CollectionId(columns = @Column(name = "IMAGE_ID"), type = @Type(type = "long"), generator = Constants.ID_GENERATOR)
    protected Collection<String> images = new ArrayList<String>();
```

ITEM:

| Id | Name |
|----|------|
| 1  | Foo  |
| 2  | B    |
| 3  | C    |


IMAGE:

| Image_Id | ItemID | FileName |
|---------|--------|----------|
| 1       | 1      | Foo.jpg  |
| 2       | 1      | bar.jpg  |
| 3       | 1      | baz.jpg  |
| 4       | 1      | baz.jpg  |
| 5       | 2      | b.jpg    |

### @OrderColumn
暂时不知道有啥用

## mapping a map
```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE")
    @MapKeyColumn(name = "FILENAME")
    @Column(name = "IMAGENAME")
    protected Map<String, String> images = new HashMap<String, String>();
    // ...
}
```

ITEM:

| Id | Name |
|----|------|
| 1  | Foo  |
| 2  | B    |
| 3  | C    |


IMAGE:
| Item_Id | FileName | IMAGENAME |
|---------|----------|-----------|
| 1       | Foo.jpg  | Foo       |
| 1       | bar.jpg  | bar       |
| 1       | baz.jpg  | baz       |
| 2       | b1.jpg   | b1        |
| 2       | b2.jpg   | b2        |

这些都是可以进行有序化的


## @Parent 
用于address持有一个USer对象,不常用

## Mapping entity associations

上面讲的这些都存在严重的依赖关系,删一个就会级联删除

```java
@Entity
public class Bid {

@ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name = "ITEM_ID", nullable = false) protected Item item;
// ...
}
```
You don’t need to map the other side of this relationship; you can ignore the one-to- many association from Item to Bid. 

不过这是单向的,如果是双向的依然需要绑定
```java
@Entity
public class Item {
@OneToMany(mappedBy = "item", fetch = FetchType.LAZY)
    protected Set<Bid> bids = new HashSet<>();
// ...
}
```

# Managing data

[lifecycle of a object in hibernate](!https://github.com/zzzyyyxxxmmm/basics/blob/master/image/hibernate_life.png)

注意remove之后不一定立刻变成transient,而是经历flush/close之后才会transient

The persistence context acts as a first-level cache; it remembers all entity instances you’ve handled in a particular unit of work.

```java
Item item = em.getReference(Item.class, ITEM_ID);
```
If the persistence context already contains an Item with the given identifier, that Item instance is returned by getReference() without hitting the database. Furthermore, if no persistent instance with that identifier is currently managed, Hibernate produces a hollow placeholder: a proxy. This means getReference() won’t access the database, and it doesn’t return null, unlike find().

Hibernate, as a JPA implementation, synchronizes at the following times:
* When a joined JTA system transaction is committed
* Before a query is executed—we don’t mean lookup with find() but a query
with javax.persistence.Query or the similar Hibernate API
* When the application calls flush() explicitly

# Transactions and concurrency 重要啊

```java
EntityManager em = null;
UserTransaction tx = TM.getUserTransaction();
try {
    tx.begin();
    em = JPA.createEntityManager();
    tx.commit();
} catch (Exception ex) {
     try {
    if (tx.getStatus() == Status.STATUS_ACTIVE
        || tx.getStatus() == Status.STATUS_MARKED_ROLLBACK)
        tx.rollback();
} catch (Exception rbEx) {
 System.err.println("Rollback of transaction failed, trace follows!");
        rbEx.printStackTrace(System.err);
    }
    throw new RuntimeException(ex);
} finally {
    if (em != null && em.isOpen())
        em.close();
}
```

# Fetch plans, strategies, and profiles




# tips
1. 一个entity里的properties最好是不可分的,例如User里不要包含一个address,如果本身数据库就不包含address表的话,User里包含address会导致address的lifecycle脱离User. 多个User指向一个address也是不好的,例如修改一个address可能导致多个user的address发生改变