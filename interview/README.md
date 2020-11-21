### What's the difference between stackoverflowError and outofmemoryError

### What is IOC and DI
IOC is an idea. Instead of creating an object by itself, IOC will revieve the object created by some other clients.

DI is one of the implementation of IOC, Spring provides Constructor Injection and Setter/Getter Injection to inject the object.

### What is AOP
AOP addresses the problem of cross-cutting concerns. It adds additional behavior to existing code (an advice) without modifying the code itself, instead they do so by a "pointcut" specification, in spring, it's called Aspect. We can use aspect to do like log, security check.

### ApplicationContext BeanFactory how bean is initialized
ApplicationContext extends from BeanFactory, so it includes all functionality of the BeanFactory. ApplcaitionContext support file access, event propagation. BeanFactory applies lazy loading, while AC applies eager loading.

### Json and XML
JSON (JavaScript Object Notation) is a lightweight data-interchange format and it completely language independent. It is based on the JavaScript programming language and easy to understand and generate.It supports array. It doesn’t supports comments.

XML (Extensible markup language) was designed to carry data, not to display data. It is a W3C recommendation. Extensible Markup Language (XML) is a markup language that defines a set of rules for encoding documents in a format that is both human-readable and machine-readable. The design goals of XML focus on simplicity, generality, and usability across the Internet. It is a textual data format with strong support via Unicode for different human languages. Although the design of XML focuses on documents, the language is widely used for the representation of arbitrary data structures such as those used in web services.

## Oauth

## CAS

# SQL

### What is Stored Procedures, Functions, views, triggers.

### 什么事务, 什么是四大隔离级别, 四大隔离级别是如何实现的
事务是由一系列操作组成的, 他们具有四大特性: ACID. 事务隔离由锁实现，原子性，一致性，持久性通过数据库的redo log和undo log来完成. 事务具有四种隔离级别, 这些都是通过锁和MVCC完成. 
* RU下select不加任何锁, 其他操作加上排他锁
* RC下select不加锁, select会通过MVCC用snapshot read来读取结果, 在每次select之间有其他事务更新了我们读取的数据并提交了，那就出现了不可重复读
* RR下select也不加锁, 但是一次事务中只在第一次select时生成版本，后续的查询都是在这个版本上进行，从而实现了可重复读, 但是如果中间有写操作, 就会使用current read, 依旧会有幻读情况发生

### MVCC的原理
InnoDB每个事务有个唯一事务ID - transaction id。在事务开始时向InnoDB事务系统申请的，按申请顺序严格递增。
一个row有多个版本, 里面有transaction id

在InnoDB中，会在每行数据后添加两个额外的隐藏的值来实现MVCC，这两个值一个记录这行数据何时被创建，另外一个记录这行数据何时过期（或者被删除）。 在实际操作中，存储的并不是时间，而是事务的版本号，每开启一个新事务，事务的版本号就会递增。

在 RC 隔离级别下，每个 SELECT 语句开始时，都会重新将当前系统中的所有的活跃事务拷贝到一个列表生成 ReadView。二者的区别就在于生成 ReadView 的时间点不同，一个是事务之后第一个 SELECT 语句开始、一个是事务中每条 SELECT 语句开始。

ReadView 中是当前活跃的事务 ID 列表，称之为 m_ids，其中最小值为 up_limit_id，最大值为 low_limit_id，事务 ID 是事务开启时 InnoDB 分配的，其大小决定了事务开启的先后顺序，因此我们可以通过 ID 的大小关系来决定版本记录的可见性，具体判断流程如下：

* 如果被访问版本的 trx_id 小于 m_ids 中的最小值 up_limit_id，说明生成该版本的事务在 ReadView 生成前就已经提交了，所以该版本可以被当前事务访问。
* 如果被访问版本的 trx_id 大于 m_ids 列表中的最大值 low_limit_id，说明生成该版本的事务在生成 ReadView 后才生成，所以该版本不可以被当前事务访问。需要根据 Undo Log 链找到前一个版本，然后根据该版本的 DB_TRX_ID 重新判断可见性。
* 如果被访问版本的 trx_id 属性值在 m_ids 列表中最大值和最小值之间（包含），那就需要判断一下 trx_id 的值是不是在 m_ids 列表中。如果在，说明创建 ReadView 时生成该版本所属事务还是活跃的，因此该版本不可以被访问，需要查找 Undo Log 链得到上一个版本，然后根据该版本的 DB_TRX_ID 再从头计算一次可见性；如果不在，说明创建 ReadView 时生成该版本的事务已经被提交，该版本可以被访问。
此时经过一系列判断我们已经得到了这条记录相对 ReadView 来说的可见结果。此时，如果这条记录的 delete_flag 为 true，说明这条记录已被删除，不返回。否则说明此记录可以安全返回给客户端。

在RR下生成readview的规则:

* SELECT时，读取创建版本号<=当前事务版本号，删除版本号为空或>当前事务版本号。
* INSERT时，保存当前事务版本号为行的创建版本号
* DELETE时，保存当前事务版本号为行的删除版本号
* UPDATE时，插入一条新纪录，保存当前事务版本号为行创建版本号，同时保存当前事务版本号到原来删除的行

RC下每次都会生成新的ReadView

[MySQL InnoDB MVCC 机制的原理及实现](https://zhuanlan.zhihu.com/p/64576887)

### 如何解决幻读
1. 设置隔离级别, 在S会给select加上共享锁, 读写互斥
2. Next-Key Locks 也叫间隙锁，它是Record Lock + Gap Lock形成的一个闭区间锁。比如select * from user where id>=1 and id<=10 for update，就会在id为[1,10]的索引闭区间上加Next-Key Lock。

### select什么情况下会加锁
1. 显示申明
* select: 即最常用的查询，是不加任何锁的
* select ... lock in share mode: 会加共享锁(Shared Locks)
* select ... for update: 会加排它锁
2. S隔离级别
# 分布式

## Introduce the distributed ID generator
