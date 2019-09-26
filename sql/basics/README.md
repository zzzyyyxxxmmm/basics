# Basics for SQL
关于数据库的一些基础知识

# install (MAC)
1. download it
2. copy the temperory password
3. mysql -u root -p 
4. SET PASSWORD = PASSWORD('root');

### Relational Database vs noSQL

Relational Database is the database which uses relation model. Data is stored in the form of rows and columns in a table. The column in one table got some relations. Tables connect to each other by primary key and foreign key. There are some advantage of relational database: * It's easy to understand
* The SQL statement is easy to mantipulate the database. * It's easy to maintain and reduce redundancy of the data.

The data in nosql is stored by key-values pairs. It doesn't need to follow the ACID. They can easily scale out to many nodes in distribute system and handle big data access.

data model:

* Relationl
* key/value redis
* graph neo4j
* document mongo
* column-family HBase or BigTable
* array machine learning 

# How the DBMS represents the database in files on disk.
File Storage
Page Layout
Tuple Layout

## FILE STORAGE
The DBMS stores a database as one or more files
on disk. The OS doesn't know anything about these files.

The storage manager is responsible for
maintaining a database's files.

It organizes the files as a collection of pages.
1. Tracks data read/written to pages.
2. Tracks the available space.

## DATABASE PAGES
A page is a fixed-size block of data.
1. It can contain tuples, meta-data, indexes, log records…
2. Most systems do not mix page types.
3. Some systems require a page to be self-contained.

Each page is given a unique identifier.
* The DBMS uses an indirection layer to map page ids to
physical locations.

page size:
* sqlite 1KB
* oracle 4KB
* SQL server & postgreSQL 8KB
* MYSQL 16KB

### PAGE STORAGE ARCHITECTURE
Different DBMSs manage pages in files on disk in
different ways.
* Heap File Organization
* Sequential / Sorted File Organization
* Hashing File Organization

**DATABASE HEAP**

A heap file is an unordered collection of pages
where tuples that are stored in random order.
* Get / Delete Page
* Must also support iterating over all pages.

Need meta-data to keep track of what pages exist
and which ones have free space.

Two ways to represent a heap file:
* Linked List
* Page Directory

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/sql_heap_list.png" width="300" height="200">
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/sql_heap_directory.png" width="300" height="200">
</div>

### PAGE HEADER
Every page contains a header of metadata about the page's contents.
* Page Size
* Checksum
* DBMS Version
* Transaction Visibility
* Compression Information

### PAGE LAYOUT
Two approaches:
* Tuple-oriented
* Log-structured

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/sql_slotted_page.png" width="300" height="200">
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/sql_log.png" width="300" height="200">
</div>

## TUPLE LAYOUT
A tuple is essentially a sequence of bytes.
It's the job of the DBMS to interpret those bytes
into attribute types and values.

Each tuple is prefixed with a header
that contains meta-data about it.
* Visibility info (concurrency control)
* Bit Map for NULL values.
  
Attributes are typically stored in the
order that you specify them when you
create the table.

| Header | a | b | c | d |


## RECORD IDS
The DBMS needs a way to keep track
of individual tuples.
Each tuple is assigned a unique record
identifier.
* Most common: page_id + offset/slot
* Can also contain file location info. 

# ACCESS METHODS
An access method is a way that the
DBMS can access the data stored in a
table.
* Not defined in relational algebra.Three basic approaches:
* Sequential Scan
* Index Scan
* Multi-Index / "Bitmap" Scan

## SEQUENTIAL SCAN
For each page in the table:
* Retrieve it from the buffer pool.
* Iterate over each tuple and check whether
to include it.

The DBMS maintains an internal
cursor that tracks the last page / slot
it examined.

### Sequential Scan Optimizations
* Prefetching
* Parallelization
* Buffer Pool Bypass
* Zone Maps 
Pre-computed aggregates for the attribute values
in a page. DBMS checks the zone map first to
decide whether it wants to access the page.
提前存储一些例如平均值，最大值的信息，例如查找大于400的val，但最大值就是300，那么就可以直接返回了

* Late Materialization
* Heap Clustering

# 四大特性和隔离级别
ACID

* 原子性 (Atomicity)
A transaction can not be interruptted. Either all the information is committed to save, or none is saved.
* 一致性（Consistency）
The data saved can’t violate any of the database’s integrity. Any operations should be logically correct. For example, AB have one thousand dollar in sum and A send B five hundred dollar. 
* 隔离性（Isolation）
The transactionis not affected by any other transactions taking place.
* 持久性（Durability）
Once a transaction is committed, it will remain so, regardless of a subsequent system failure.
  
  我们先看看如果不考虑事务的隔离性，会发生的几种问题：

* Dirty Read
A dirty read (aka uncommitted dependency) occurs when a transaction is allowed to read data from a row that has been modified by another running transaction and not yet committed.

* Non-repeatable reads
A non-repeatable read occurs, when during the course of a transaction, a row is retrieved twice and the values within the row differ between reads.
　　　　
* Phantom reads
A phantom read occurs when, in the course of a transaction, new rows are added or removed by another transaction to the records being read.
 
现在来看看MySQL数据库为我们提供的四种隔离级别：
1. Serializable (串行化)：可避免脏读、不可重复读、幻读的发生。 使用range lock
2. Repeatable read (可重复读)：可避免脏读、不可重复读的发生。 使用write read lock
3. Read committed (读已提交)：可避免脏读的发生。 使用write lock
4. Read uncommitted (读未提交)：最低级别，任何情况都无法保证。


# 数据库的基本操作

## Database

### 查看 创建 删除database
```sql
SHOW DATABASES;

CREATE DATABASE dbname;

DROP DATABASE dbname;
```

## Table
### 查看 创建 删除table
```sql
SHOW tables;


CREATE TABLE table_name
(  field1 data_type [ not null ],
   field2 data_type [ not null ], 
   field3 data_type [ not null ], 
   field4 data_type [ not null ], 
   field5 data_type [ not null ] 
);

DROP TABLE table_name;
```

### alter table
```sql
alter table table_name 
[modify] [column column_name][datatype|null not null] [restrict|cascade]
[drop] [constraint constraint_name] 
[add] [column] column definition
```

Common rules for modifing columns:
* The length of a column can be increased to the maximum length of the given data type.
* The length of a column can be decreased only if the largest value for that col- umn in the table is less than or equal to the new length of the column.
* The number of digits for a number data type can always be increased.
* The number of digits for a number data type can be decreased only if the value with the most number of digits for that column is less than or equal to the new number of digits specified for the column.
* The number of decimal places for a number data type can either be increased or decreased.
* The data type of a column can normally be changed.

### Creating a Table from an Existing Table
```sql
create table new_table_name as
select [ *|column1, column2 ]
from table_name
[ where ]
```

### Primary Key Constraints
Primary key is the term used to uniquely identify one or more columns in a table that make a row of data unique. Although the primary key typically consists of one column in a table, more than one column can comprise the primary key.

三种方式创建主键
```sql
CREATE TABLE EMPLOYEE_TBL
(
    EMP_ID CHAR(9) NOT NULL PRIMARY KEY
)

CREATE TABLE EMPLOYEE_TBL
(
    EMP_ID CHAR(9) NOT NULL,
    ...
    PRIMARY KEY (EMP_ID, ...))
);

Alter table tb add primary key(id);

```

### Foreign Key Constraints
A foreign key is a column in a child table that references a primary key in the parent table. A foreign key constraint is the main mechanism used to enforce **referential integrity** between tables in a relational database. There have another two advantages of fk: Easier Detective Work and Better Performance.

```sql
CONSTRAINT EMP_ID_FK FOREIGN KEY (EMP_ID) REFERENCES EMPLOYEE_TBL (EMP_ID));

alter table employee_pay_tbl
add constraint id_fk foreign key (emp_id) 
references employee_tbl (emp_id);
```

### Other Constraints

Unique Constraints

NOT NULL Constraints

Check Constraints
```sql
CONSTRAINT CHK_EMP_ZIP CHECK ( EMP_ZIP = ‘46234’);

CONSTRAINT CHK_EMP_ZIP CHECK ( EMP_ZIP in (‘46234’,’46227’,’46745’) );

CONSTRAINT CHK_PAY CHECK ( PAY_RATE > 12.50 );
```

## Manipulating Data

### insert
```sql
INSERT INTO TABLE_NAME
VALUES (‘value1’, ‘value2’, [ NULL ] );

INSERT INTO TABLE_NAME (‘COLUMN1’, ‘COLUMN2’) VALUES (‘VALUE1’, ‘VALUE2’);

insert into table_name [(‘column1’, ‘column2’)] select [*|(‘column1’, ‘column2’)]
from table_name
[where condition(s)];

insert into schema.table_name values
(‘column1’, NULL, ‘column3’);
```

### update
```sql
update table_name
set column_name = ‘value’ [where condition];

update table_name
set column1 = ‘value’,
[column2 = ‘value’,]
[column3 = ‘value’] [where condition];
```

### delete
```sql
delete from table_name [where condition];
```

## Database Query

### select
```sql
SELECT [ * | ALL | DISTINCT COLUMN1, COLUMN2 ]
FROM TABLE1 [ , TABLE2 ];
ORDER BY [DESC | 1]
```

## Operators

* IS NULL 
* BETWEEN
* Exception

等于的
* IN
* LIKE

The percent sign (%)
The underscore (_)
The percent sign represents zero, one, or multiple characters. The underscore represents a single number or character. The symbols can be used in combinations.

* EXISTS
* UNIQUE

上面都是可以加上NOT 修饰

* ALL and ANY
```sql
SELECT *
FROM PRODUCTS_TBL
WHERE COST > ALL ( SELECT COST
                   FROM PRODUCTS_TBL
                   WHERE COST < 10 );
```


## Summarizing Data Results from a Query

```sql
COUNT [ (*) | (DISTINCT | ALL) ] (COLUMN NAME)
count(*) --统计所有记录
count(column)   --统计该column下非null的
SUM ([ DISTINCT ] COLUMN NAME)
MAX([ DISTINCT ] COLUMN NAME)
MIN([ DISTINCT ] COLUMN NAME)
AVG ([ DISTINCT ] COLUMN NAME)
```

## Sorting and Grouping Data

```sql
GROUP BY [1]


SELECT COLUMN1, COLUMN2 FROM TABLE1, TABLE2
WHERE CONDITIONS
GROUP BY COLUMN1, COLUMN2 HAVING CONDITIONS
ORDER BY COLUMN1, COLUMN2
```

## Joining Tables in Queries

```sql
--正常合并一个主键外键相关联的两个表
SELECT EMPLOYEE_TBL.EMP_ID,
       EMPLOYEE_PAY_TBL.DATE_HIRE
FROM EMPLOYEE_TBL,
       EMPLOYEE_PAY_TBL
WHERE EMPLOYEE_TBL.EMP_ID = EMPLOYEE_PAY_TBL.EMP_ID;

--利用inner join实现上面的
SELECT EMPLOYEE_TBL.EMP_ID,
       EMPLOYEE_PAY_TBL.DATE_HIRE
FROM EMPLOYEE_TBL
INNER JOIN EMPLOYEE_PAY_TBL
ON EMPLOYEE_TBL.EMP_ID = EMPLOYEE_PAY_TBL.EMP_ID;

--natural join会删除重复的元素
```

## Union

union all have duplicate results.

# 数据库的基础概念

## 数据库中数据的类型

只包括三种：
* String types
* Numeric types
* Date and time types

### String types
```sql
CHAR(size) --保存固定长度的字符串（可包含字母、数字以及特殊字符）。在括号中指定字符串的长度。最多 255 个字符。
VARCHAR(size) --保存可变长度的字符串（可包含字母、数字以及特殊字符）。在括号中指定字符串的最大长度。最多 255 个字符。注释：如果值的长度大于 255，则被转换为 TEXT 类型。
TEXT --存放最大长度为 65,535 个字符的字符串。

```
**CHAR(n)和VARCHAR有什么区别？**

The short answer is: VARCHAR is variable length, while CHAR is fixed length.

CHAR is a fixed length string data type, so any remaining space in the field is padded with blanks. CHAR takes up 1 byte per character. So, a CHAR(100) field (or variable) takes up 100 bytes on disk, regardless of the string it holds.

VARCHAR is a variable length string data type, so it holds only the characters you assign to it. VARCHAR takes up 1 byte per character, + 2 bytes to hold length information.  For example, if you set a VARCHAR(100) data type = ‘Jen’, then it would take up 3 bytes (for J, E, and N) plus 2 bytes, or 5 bytes in all.

### Numeric types
```sql
BIT(n)
BIT VARYING(n) 
DECIMAL(p,s) --DECIMAL速度会比FLOAT慢很多，但是精确，FLOAT你懂的 0.3=0.299999999
INTEGER 2^32
SMALLINT 2^16
BIGINT  2^64
FLOAT(p,s)  --1<=p<=21
DOUBLE PRECISION(p,s)  --22<=p<=53 建议直接使用FLOAT(p)来表示双精度
REAL(s)
```
**p** represents a number identifying the allocated or maximum length of the particu- lar field for each appropriate definition.

**s** is a number to the right of the decimal point, such as 34.ss.

注意：以上的 p 代表的并不是存储在数据库中的具体的长度，如 int(4) 并不是只能存储4个长度的数字。

实际上int(size)所占多少存储空间并无任何关系。int(3)、int(4)、int(8) 在磁盘上都是占用 4 btyes 的存储空间。就是在显示给用户的方式有点不同外，int(M) 跟 int 数据类型是相同的。

例如：

1、int的值为10 （指定zerofill）
```
int（9）显示结果为000000010
int（3）显示结果为010
```
就是显示的长度不一样而已 都是占用四个字节的空间

所以大小都一样, 只是显示的不一样

``NUMERIC(5)``

This example restricts the maximum value entered in a particular field to 99999.

``DECIMAL(4,2)``

2

12.4

12.44

12.449

The last numeric value, 12.449, is rounded off to 12.45 upon input into the col- umn. In this case, any numbers between 12.445 and 12.449 would be rounded to 12.45.

不满足的数会在插入前被round

### Date

DATE()	日期。格式：YYYY-MM-DD
注释：支持的范围是从 '1000-01-01' 到 '9999-12-31'

DATETIME()	*日期和时间的组合。格式：YYYY-MM-DD HH:MM:SS
注释：支持的范围是从 '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59'

TIMESTAMP()	*时间戳。TIMESTAMP 值使用 Unix 纪元('1970-01-01 00:00:00' UTC) 至今的秒数来存储。格式：YYYY-MM-DD HH:MM:SS
注释：支持的范围是从 '1970-01-01 00:00:01' UTC 到 '2038-01-09 03:14:07' UTC

TIME()	时间。格式：HH:MM:SS
注释：支持的范围是从 '-838:59:59' 到 '838:59:59'

YEAR()	2 位或 4 位格式的年。
注释：4 位格式所允许的值：1901 到 2155。2 位格式所允许的值：70 到 69，表示从 1970 到 2069。

## 范式

### What is Normalization ?
Normalization is a process of reducing redundancies of data in a database.

Normalization is a technique that is used when designing and redesigning a database. 

Normalization is a process or set of guidelines used to optimally design a database to reduce redundant data.

### What is normal form?
The actual guidelines of normalization, called normal forms

Normal form is a way of measuring the levels, or depth, to which a database has been normalized. A database’s level of normalization is determined by the normal form.

### The First Normal Form
每个column不可再分

The objective of the first normal form is to divide the base data into logical units called tables. When each table has been designed, a primary key is assigned to most or all tables. 

每个column不能在分，必须都是基本属性，比如一个列不能是Parent之类的，Parent必须提取出来单独作为一个表

### The Second Normal Form

消除部分依赖

The objective of the second normal form is to take data that is only partly dependent on the primary key and enter that data into another table. 

什么是部分依赖？

比如有个scoreTable(studentId,courseId,departmentName,departmentPresident,score)，主键是(studentId,courseId)

这个表有什么缺点？

1. 数据冗余，departmentName，departmentPresident大量重复
2. 如果我们删除所有的studentId信息，那么我们就会丢失所有的department信息
3. 该校新建了一个系，但是studentId又不能为空，因此无法插入
4. 如果一个学生转了系，那需要同时修改该表所有的departmentName,departmentPresident信息

通过主键(studentId,courseId)我们可以推出departmentName,并且studentId -> departmentName，这时候我们说departmentName**部分依赖**于主键. departmentPresident同理

对于score，我们单独通过studentId或courseId都不能得到score，必须要通过(studentId,courseId)才能得到score，因此score**完全依赖**于主键.

因此courseName应该被抽离出来变成:
scoreTable(studentId,couseId,score)
studentTable(studentId,departmentName,departmentPresident)

抽离出来的column怎么分配主要看这个column部分依赖哪个主键



### The Third Normal Form

消除传递依赖

The third normal form’s objective is to remove data in a table that is not dependent on the primary key. 

还有待改进的地方：2 ,3

对于上步得到的courseTable(studentId,departmentName,departmentPresident)

studentId -> courseName

courseName -> departmentName 

=> studentId -> departmentPresident 

因此departmentPresident**传递依赖**于studentId

因此departmentName,departmentPresident 应该再次被抽离出来:
scoreTable(studentId,couseId,score)
studentTable(studentId,departmentName)
departmentTable(departmentName,departmentPresident)

### Boyce–Codd normal form (or BCNF or 3.5NF)

上面都是消除非主属性对于key的依赖，这里我们需要消除主属性对key的依赖（key包含主属性），指导如何设计key

## Transction
Transactions are sequences of operations accomplished in a logical order on a shared database.

The following list describes the nature of transactions:
* All transactions have a beginning and an end.
* A transaction can be saved or undone.
* If a transaction fails in the middle, no part of the transaction can be saved to the database.

不可commit或者rollback一些操作，例如commit创建table，rollback truncate掉的表

commit 之后的信息会被暂时存世在rollback area，可以随时rollback，但一旦issued就会被正式提交并清空rollback area.

```sql
DELETE FROM PRODUCTS_TMP
WHERE COST < 14;
--8 rows deleted.
--A COMMIT statement is issued to save the changes to the database, completing the transaction.
SAVEPOINT sp1;
COMMIT;

rollback to sp1;

RELEASE SAVEPOINT savepoint_name;

--Commit complete.
```

## Index

### What is an index
Simply put, an index is a pointer to data in a table. An index in a database is very similar to an index in the back of a book. An index’s main purpose is to improve the performance of data retrieval.

Single-Column Indexes
Unique Indexes  column必须是unique的
Composite Indexes
Implicit Indexes 被自动创建的index

An index is usually implicitly created when you define a primary key for a table. 

### When Should Indexes Be Considered?
* Columns used as primary keys
* Columns used as foreign keys
* Columns frequently used to join tables
* Columns frequently used as conditions in a query
* Columns that have a high percentage of unique values

Unique indexes are implicitly used in conjunction with a primary key for the pri- mary key to work. Foreign keys are also excellent candidates for an index because they are often used to join the parent table. Most, if not all, columns used for table joins should be indexed.

Columns that are frequently referenced in the ORDER BY and GROUP BY clauses should be considered for indexes. 

Furthermore, indexes should be created on columns with a high number of unique values, or columns that when used as filter conditions in the WHERE clause return a low percentage of rows of data from a table. 

### When Should Indexes Be Avoided?
Although indexes are intended to enhance a database’s performance, there are
times of an when they should be avoided. The following guidelines indicate when the use index should be reconsidered:
* Indexes should not be used on small tables.
* Indexes should not be used on columns that return a high percentage of data rows when used as a filter condition in a query’s WHERE clause. For instance, you would not have an entry for the words the or and in the index of a book.
* Tables that have frequent, large batch update jobs run can be indexed. However, the batch job’s performance is slowed considerably by the index. The conflict of having an index on a table that is frequently loaded or manipulat- ed by a large batch process can be corrected by dropping the index before the batch job, and then re-creating the index after the job has completed. This is because the indexes are also updated as the data is inserted, causing addition- al overhead.
* Indexes should not be used on columns that contain a high number of NULL values.
* Columns that are frequently manipulated should not be indexed. Maintenance on the index can become excessive.

## Improving Database Performance
**SQL statement tuning** is the process of optimally building SQL statements to achieve results in the most effective and efficient manner. SQL tuning begins with the basic arrangement of the elements in a query. Simple formatting can play a rather large role in the optimiza- tion of a statement.

## Formatting Your SQL Statement
* Formatting SQL statements for readability
* The order of tables in the FROM clause
FROM SMALLEST TABLE,
     LARGEST TABLE
* The placement of the most restrictive conditions in the WHERE clause 
SQL 是从底部开始读where的，因此most restrictive conditions应该被放在最后面
* The placement of join conditions in the WHERE clause
被join的应该是最小的table

## Other Performance Considerations
* Using the LIKE operator and wildcards 
* Avoiding the OR operator
* Avoiding the HAVING clause
* Avoiding large sort operations
* Using stored procedures

## Managing Database Users

### How Does a User Differ from a Schema?
A database’s objects are associated with database user accounts, called schemas. A schema is a set of database objects that a database user owns. This database user is called the schema owner. The difference between a regular database user and a schema owner is that a schema owner owns objects within the database, whereas most users do not own objects. Most users are given database accounts to access data that is contained in other schemas. Because the schema owner actually owns these objects, he has complete control over them.

### Creating Users in MySQL

The steps for creating a user account in MySQL follow:
1. Create the user account within the database.
2. Grant the appropriate privileges to the user account.
The syntax for creating the user account is very similar to the syntax used in Oracle.
SELECT USER user [IDENTIFIED BY [PASSWORD] ‘password’]

The syntax for granting the user’s privileges is also similar to the Oracle version:

```sql
GRANT priv_type [(column_list)] [, priv_type [(column_list)]] ... ON [object_type]
{tbl_name | * | *.* | db_name.* | db_name.routine_name} TO user
# 一些小问题
```

### Creating Schemas
Schemas are created via the CREATE SCHEMA statement. The syntax is as follows:
```sql
CREATE SCHEMA [ SCHEMA_NAME ] [ USER_ID ]
[ DEFAULT CHARACTER SET CHARACTER_SET ]
[PATH SCHEMA NAME [,SCHEMA NAME] ] [ SCHEMA_ELEMENT_LIST ]
```

The following is an example:
```sql
CREATE SCHEMA USER1
CREATE TABLE TBL1
(COLUMN1 DATATYPE [NOT NULL],
(COLUMN2 DATATYPE [NOT NULL]...)
GRANT SELECT ON TBL1 TO USER2
```

### User sessions
Remember that the syntax varies between implementations. In addition, most database users do not manually issue the commands to connect or disconnect from the database. Most users access the database through a vendor-provided or third-party tool that prompts the user for a username and password, which in turn connects to the database and initiates a database user session.

## Managing Database Security
Generally, user management is the process of creating user accounts, removing user accounts, and keeping track of users’ actions within the database. Database security is going a step further by granting privileges for specific database access, revoking those privileges from users, and taking measures to protect other parts of the data- base, such as the underlying database files.

### Controlling Privileges Through Roles
A role is an object created in the database that contains group-like privileges. Roles can reduce security maintenance by not having to grant explicit privileges directly to a user. Group privilege management is much easier to handle with roles. A role’s privileges can be changed, and such a change is transparent to the user.

## Creating and Using Views and Synonyms
A view is a virtual table. That is, a view looks like a table and acts like a table as far as a user is concerned, but it doesn’t require physical storage. 

If a table that was used to create a view is dropped, the view becomes inaccessible. You receive an error when trying to query against the view.

### Creating Views

```sql
CREATE VIEW CUSTOMERS AS
SELECT *
FROM CUSTOMER_TBL;
```

### Synonym
A synonym is merely another name for a table or a view. Synonyms are usually created so a user can avoid having to qualify another user’s table or view to access the table or view.

## Working with the System Catalog
The system catalog is a collection of tables and views that contain important information about a database. A system catalog is available for each database. Information in the system catalog defines the structure of the database and also information on the data con- tained therein. For example, the data dictionary language (DDL) for all tables in the database is stored in the system catalog. See Figure 21.1 for an illustration of the system cata- log within the database.

The system catalog is basically a group of objects that contain information that defines other objects in the database, the structure of the database itself, and various other significant information.

### What Is Contained in the System Catalog?
* User accounts and default settings
* Privileges and other security information
* Performance statistics
* Object sizing
* Object growth
* Table structure and storage
* Index structure and storage
* Information on other database objects, such as views, synonyms, triggers, and stored procedures
* Table constraints and referential integrity information
* User sessions
* Auditing information
* Internal database settings
* Locations of database files

## Advanced SQL Topics

### Cursors
An SQL cursor is an area in database memory where the last SQL statement is stored. If the current SQL statement is a database query, a row from the query is also stored in memory. This row is the cursor’s current value or cur- rent row.

A cursor is typically used to retrieve a subset of data from the database.

```sql
DECLARE CURSOR EMP_CURSOR IS
SELECT * FROM EMPLOYEE_TBL
{ OTHER PROGRAM STATEMENTS }
```

According to the ANSI standard, the following operations are used to access a cursor after it has been defined:
* OPEN Opens a defined cursor
When a cursor is opened, the specified cursor’s SELECT statement is executed and the results of the query are stored in a staging area in memory.

* FETCH Fetches rows from a cursor into a program variable
The contents of the cursor (results from the query) can be retrieved through the use of the FETCH statement after a cursor has been opened.
```sql
FETCH EMP_CURSOR INTO EMP_RECORD
```

* CLOSE Closes the cursor when operations against the cursor are complete

## Stored Procedures and Functions
A stored procedure is a group of one or more SQL statements or functions that are stored in the database, compiled, and ready to be executed by a database user. A stored function is the same as a stored procedure, but a function is used to return a value.

Functions are not allowed to change anything, must have at least one parameter, and they must return a value. A function can be used inline in SQL statements if it returns a scalar value, or can be joined upon if it returns a result set. Stored procs do not have to have a parameter, can change database objects, and do not have to return a value.

```sql
CREATE PROCEDURE NEW_PRODUCT
(PROD_ID IN VARCHAR2, PROD_DESC IN VARCHAR2, COST IN NUMBER)
AS
BEGIN
  INSERT INTO PRODUCTS_TBL
  VALUES (PROD_ID, PROD_DESC, COST);
  COMMIT;
END;

CALL NEW_PRODUCT (‘9999’,’INDIAN CORN’,1.99);
```

## Triggers
A trigger is a compiled SQL procedure in the database used to perform actions based on other actions that occur within the database. The trigger can be executed before or after an INSERT, DELETE, or UPDATE statement. Triggers can also be used to check data integrity before an INSERT, DELETE, or UPDATE statement. Triggers can roll back transactions, and they can modify data in one table and read from another table in another database.

创建只有一个执行语句的触发器
创建了一个名为trig1的触发器，一旦在work表中有插入动作，就会自动往time表里插入当前时间

```sql
CREATE TRIGGER trig1 AFTER INSERT
    ON work FOR EACH ROW
    BEGIN
      INSERT INTO time VALUES(NOW());
    END
```

## 如何连接数据库
`` mysql -u root -p``

然后输入密码

# install (MAC)
1. download it
2. copy the temperory password
3. mysql -u root -p 
4. SET PASSWORD = PASSWORD('root');

## MongoDB和MySQL区别
