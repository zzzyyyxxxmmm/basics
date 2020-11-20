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
* RC下select不加锁, select会通过MVCC用snapshot read来读取结果, 但是，该级别下还是遗留了不可重复读和幻读问题： MVCC版本的生成时机: 是每次select时。这就意味着，如果我们在事务A中执行多次的select，在每次select之间有其他事务更新了我们读取的数据并提交了，那就出现了不可重复读
* RR下select也不加锁, 但是一次事务中只在第一次select时生成版本，后续的查询都是在这个版本上进行，从而实现了可重复读, 但是如果中间有写操作, 就会使用current read, 依旧会有幻读情况发生

# 分布式

## Introduce the distributed ID generator
