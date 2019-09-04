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