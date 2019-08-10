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

## 后台线程
后台线程用于刷新内存池中的数据

后台线程类型：
* Master Thread
* IO Thread
* Purge Thread
* Page Cleaner Thread

## 内存

在数据库中进行读取页(每个页的大小是16KB)的操作，首先将从磁盘读到的页存放在缓冲池中，下一次读取相同的页时，首先判断该页是否在缓冲池中。若在缓冲池中，称该页在缓冲池中命中，直接读取该页。否则，读取磁盘上的页。

对于数据库中页的修改操作，则首先修改在缓冲池中的页，然后再以一定的频率刷新到磁盘上。这里需要注意的是，页从缓冲池刷新回磁盘的操作并不是在每次页发生更新时触发，而是通过一种checkpoint的机制刷新回磁盘。因此内存的大小显著印象数据库读取效率

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/db_buffer.png" width="500" height="300">
</div>

mysql允许多个缓冲池

### 缓冲池是怎么管理页的

1. 数据库刚启动，空闲页都在Free List当中
2. 需要缓冲页时，从Free List删除页，然后添加到LRU List；如果没有空闲页则将LRU尾端页淘汰，并分配给新页
3. 如果LRU List当中的页被修改了，则会产生脏页。此时缓冲池数据与硬盘不一致，该页会存在于Flush List当中（注意LRU和Flush都会持有该脏页）。

缓冲池是通过LRU进行管理，但是在插入新的页时，并不是直接放到LRU列表的首部，而是放到midpoint位置，通常是列表的5/8(hot list占5/8),那为什么不采用朴素的LRU算法，直接将读取的页放入到LRU列表的首部呢？这是因为若直接将读取到的页放入到LRU的首部，那么某些SQL操作可能会使缓冲池中的页被刷新出，从而影响缓冲池的效率。常见的这类操作为索引或数据的扫描操作。这类操作需要访问表中的许多页，甚至是全部的页，而这些页通常来说又仅在这次查询操作中需要，并不是活跃的热点数据。如果页被放入LRU列表的首部，那么非常可能将所需要的热点数据页从LRU列表中移除，而在下一次需要读取该页时，InnoDB存储引擎需要再次访问磁盘。

实际就是LRU被分成了hot list和warm list，数据第一次被访问会被插入到warm list的头部，如果在过了InnoDB_old_blocks_time时间后访问，则会被插入到hot list的头部。因此，如果有大量一次访问的数据，则全部都会被插入到warm list里而不是hot list。

如果在过了InnoDB_old_blocks_time之前读取warm list里的数据，那么则不会被添加到hot list里，可以通过下面语句修改：
```sql
show variables like 'innodb_old_blocks_pct'\G
show variables like 'innodb_old_blocks_time'\G
set global innodb_old_blocks_time=1000;
set global innodb_old_blocks_pct=20;

show engine innodb status\G 查看LRU和Free列表状态，单位都是一个页
```

另外，一些被压缩的页会被存储在unzip_LRU，LRU中的页包括了unzip_LRU

### redo log buffer
在通常情况下，8MB的重做日志缓冲池足以满足绝大部分的应用，因为重做日志在下列三种情况下会将重做日志缓冲中的内容刷新到外部磁盘的重做日志文件中。
1. Master Thread每一秒将重做日志缓冲刷新到重做日志文件；
2. 每个事务提交时会将重做日志缓冲刷新到重做日志文件；
3. 当重做日志缓冲池剩余空间小于1/2时，重做日志缓冲刷新到重做日志文件。
   
### 额外的内存池

### Checkpoint
缓冲池的设计目的为了协调CPU速度与磁盘速度的鸿沟。因此页的操作首先都是在缓冲池中完成的。如果一条DML语句，如Update或Delete改变了页中的记录，那么此时页是脏的，即缓冲池中的页的版本要比磁盘的新。数据库需要将新版本的页从缓冲池刷新到磁盘。

倘若每次一个页发生变化，就将新页的版本刷新到磁盘，那么这个开销是非常大的。若热点数据集中在某几个页中，那么数据库的性能将变得非常差。同时，如果在从缓冲池将页的新版本刷新到磁盘时发生了宕机，那么数据就不能恢复了。为了避免发生数据丢失的问题，当前事务数据库系统普遍都采用了Write Ahead Log策略，即当事务提交时，先写重做日志，再修改页。当由于发生宕机而导致数据丢失时，通过重做日志来完成数据的恢复。这也是事务ACID中D（Durability持久性）的要求。

思考下面的场景，如果重做日志可以无限地增大，同时缓冲池也足够大，能够缓冲所有数据库的数据，那么是不需要将缓冲池中页的新版本刷新回磁盘。因为当发生宕机时，完全可以通过重做日志来恢复整个数据库系统中的数据到宕机发生的时刻。但是这需要两个前提条件：
1. 缓冲池可以缓存数据库中所有的数据
2. 重做日志可以无限增大

好的，即使上述两个条件都满足，那么还有一个情况需要考虑：宕机后数据库的恢复时间。当数据库运行了几个月甚至几年时，这时发生宕机，重新应用重做日志的时间会非常久，此时恢复的代价也会非常大。

因此Checkpoint（检查点）技术的目的是解决以下几个问题：

* 缩短数据库的恢复时间
* 缓冲池不够用时，将脏页刷新到磁盘
* 重做日志不可用时，刷新脏页。

在InnoDB存储引擎中，Checkpoint发生的时间、条件及脏页的选择等都非常复杂。而Checkpoint所做的事情无外乎是将缓冲池中的脏页刷回到磁盘。不同之处在于每次刷新多少页到磁盘，每次从哪里取脏页，以及什么时间触发Checkpoint。在InnoDB存储引擎内部，有两种Checkpoint，分别为：
* Sharp Checkpoint
* Fuzzy Checkpoint
Sharp Checkpoint发生在数据库关闭时将所有的脏页都刷新回磁盘，这是默认的工作方式，

这里笔者进行了概括，在InnoDB存储引擎中可能发生如下几种情况的Fuzzy Checkpoint：

* Master Thread Checkpoint
* FLUSH_LRU_LIST Checkpoint
* Async/Sync Flush Checkpoint
* Dirty Page too much Checkpoint

## Master Thread

### 主loop

每秒一次的操作包括：
* 日志缓冲刷新到磁盘，即使这个事务还没有提交（总是）
* 合并插入缓冲（可能）
* 至多刷新100个InnoDB的缓冲池中的脏页到磁盘（可能）
* 如果当前没有用户活动，则切换到background loop（可能）

即使某个事务还没有提交，InnoDB存储引擎仍然每秒会将重做日志缓冲中的内容刷新到重做日志文件。这一点是必须要知道的，因为这可以很好地解释为什么再大的事务提交（commit）的时间也是很短的。

合并插入缓冲（Insert Buffer）并不是每秒都会发生的。InnoDB存储引擎会判断当前一秒内发生的IO次数是否小于5次，如果小于5次，InnoDB认为当前的IO压力很小，可以执行合并插入缓冲的操作。

同样，刷新100个脏页也不是每秒都会发生的。InnoDB存储引擎通过判断当前缓冲池中脏页的比例（buf_get_modified_ratio_pct）是否超过了配置文件中innodb_max_dirty_pages_pct这个参数（默认为90，代表90%），如果超过了这个阈值，InnoDB存储引擎认为需要做磁盘同步的操作，将100个脏页写入磁盘中。

接着来看每10秒的操作，包括如下内容：

* 刷新100个脏页到磁盘（可能的情况下）
* 合并至多5个插入缓冲（总是）
* 将日志缓冲刷新到磁盘（总是）
* 删除无用的Undo页（总是）
* 刷新100个或者10个脏页到磁盘（总是）

在以上的过程中，InnoDB存储引擎会先判断过去10秒之内磁盘的IO操作是否小于200次，如果是，InnoDB存储引擎认为当前有足够的磁盘IO操作能力，因此将100个脏页刷新到磁盘。接着，InnoDB存储引擎会合并插入缓冲。不同于每秒一次操作时可能发生的合并插入缓冲操作，这次的合并插入缓冲操作总会在这个阶段进行。之后，InnoDB存储引擎会再进行一次将日志缓冲刷新到磁盘的操作。这和每秒一次时发生的操作是一样的。

接着InnoDB存储引擎会进行一步执行full purge操作，即删除无用的Undo页。对表进行update、delete这类操作时，原先的行被标记为删除，但是因为一致性读（consistent read）的关系，需要保留这些行版本的信息。但是在full purge过程中，InnoDB存储引擎会判断当前事务系统中已被删除的行是否可以删除，比如有时候可能还有查询操作需要读取之前版本的undo信息，如果可以删除，InnoDB会立即将其删除。从源代码中可以发现，InnoDB存储引擎在执行full purge操作时，每次最多尝试回收20个undo页。

然后，InnoDB存储引擎会判断缓冲池中脏页的比例（buf_get_modified_ratio_pct），如果有超过70%的脏页，则刷新100个脏页到磁盘，如果脏页的比例小于70%，则只需刷新10%的脏页到磁盘。

### Background Loop
接着来看background loop，若当前没有用户活动（数据库空闲时）或者数据库关闭（shutdown），就会切换到这个循环。background loop会执行以下操作：
* 删除无用的Undo页（总是）；
* 合并20个插入缓冲（总是）；
* 跳回到主循环（总是）；
* 不断刷新100个页直到符合条件（可能，跳转到flush loop中完成）。

若flush loop中也没有什么事情可以做了，InnoDB存储引擎会切换到suspend__loop，将Master Thread挂起，等待事件的发生。若用户启用（enable）了InnoDB存储引擎，却没有使用任何InnoDB存储引擎的表，那么Master Thread总是处于挂起的状态。