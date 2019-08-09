# InnoDB 技术内幕

启动数据库实例时，会按照/etc/my.cnf -> /etc/mysql/my.cnf -> /usr/local/mysql/etc/my.cnf -> ~/.my.cnf，最终会以最后读到的参数为准，通常我们修改的是/etc/my.cnf

里面定义了data存放的目录

# MySQL体系结构和存储引擎

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/mysql_arc.png" width="700" height="500">
</div>

### InnoDB
InnoDB储存引擎支持事务，其设计目标主要面向在线事务处理(OLTP)。其特点是行锁设计，支持外键，并支持类似于Oracle的非锁定读，即默认读取操作不会产生锁。

InnoDB通过使用多版本并发控制(MVCC)来获得高并发性，并且实现了SQL标准的4种隔离级别，默认为REPEARTABLE级别。同时，使用一种被称为next-key locking的策略来避免幻读现象的产生。初次之外，InnoDB储存引擎还提供了插入缓冲（insert buffer），二次写（double write），自适应哈希索引（adaptive hash index），预读（read ahead）等高性能的功能。

对于表中数据的存储，InnoDB存储引擎采用了聚集（clustered的方式，因此每张表的存储都死按主键的顺序进行存放。如果没有显示的定义主键，则会默认每行生成一个ROWID，并以此作为主键）

```sql
show engines\G

create table mytest Engine=MyISAM --创建引擎
```
可以用来查看当前可以使用的存储引擎

### MyISAM
不支持事务，表锁，支持全文索引，主要面向一些OLAP，是5.5.8版本之前的默认存储引擎

### NDB
NDB将数据全部放在内存中，因此主键查找速度极快，可以通过添加NDB数据存储节点线性提高数据库性能。

另外，复杂的连接操作需要巨大的网络开销，因此查询速度很慢

### Memory存储引擎
顾名思义，就是放在内存里的，mysql使用这个引擎作为临时表来存放查询的中间结果集，如果数据过大或者包含Text或Blob字段，就会转化为MyISAM而存放到磁盘中

## 连接MySQL

### TCP/IPs

允许远程用户登录访问mysql的方法需要手动增加可以远程访问数据库的用户。

方法一、本地登入mysql，更改 "mysql" 数据库里的 "user" 表里的 "host" 项，将"localhost"改为"%"

```sql
use mysql;
update user set host = '%' where user = 'root';
select host, user from user;
```

方法二、直接授权(推荐)

从任何主机上使用root用户，密码：youpassword（你的root密码）连接到mysql服务器：

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;
```

操作完后切记执行以下命令刷新权限 

FLUSH PRIVILEGES 

host为%代表可以从任何IP段下连接，并且不需要密码

**远程连接命令:**
```
mysql -h192.168.0.101 -u david -p
```

# InnoDB存储引擎

## InnoDB架构

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/innodb_arc.png" width="500" height="300">
</div>

这个上面的矩形是内存块，用于：
* 维护所有进程/线程需要访问的多个内部数据结构
* 缓存数据
* redo log缓冲

### 后台线程
后台线程用于刷新内存池中的数据

后台线程类型：
* Master Thread
* IO Thread
* Purge Thread
* Page Cleaner Thread

### 内存

在数据库中进行读取页的操作，首先将从磁盘读到的页存放在缓冲池中，下一次读取相同的页时，首先判断该页是否在缓冲池中。若在缓冲池中，称该页在缓冲池中命中，直接读取该页。否则，读取磁盘上的页。

对于数据库中页的修改操作，则首先修改在缓冲池中的页，然后再以一定的频率刷新到磁盘上。这里需要注意的是，页从缓冲池刷新回磁盘的操作并不是在每次页发生更新时触发，而是通过一种checkpoint的机制刷新回磁盘。因此内存的大小显著印象数据库读取效率

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/db_buffer.png" width="500" height="300">
</div>