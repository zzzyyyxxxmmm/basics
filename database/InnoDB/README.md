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
不支持事务，表锁，支持全文索引，也不支持外键，其优势是访问的速度快,主要面向一些OLAP，是5.5.8版本之前的默认存储引擎

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

## InnoDB关键特性
InnoDB存储引擎的关键特性包括：
* 插入缓冲（Insert Buffer）
* 两次写（Double Write）
* 自适应哈希索引（Adaptive Hash Index）
* 异步IO（Async IO）
* 刷新邻接页（Flush Neighbor Page）

### Insert Buffer

InnoDB存储引擎开创性地设计了Insert Buffer，对于非聚集索引的插入或更新操作，不是每一次直接插入到索引页中，而是先判断插入的非聚集索引页是否在缓冲池中，若在，则直接插入；若不在，则先放入到一个Insert Buffer对象中，好似欺骗。数据库这个非聚集的索引已经插到叶子节点，而实际并没有，只是存放在另一个位置。然后再以一定的频率和情况进行Insert Buffer和辅助索引页子节点的merge（合并）操作，这时通常能将多个插入合并到一个操作中（因为在一个索引页中），这就大大提高了对于非聚集索引插入的性能。

然而Insert Buffer的使用需要同时满足以下两个条件：
* 索引是辅助索引（secondary index）
* 索引不是唯一（unique）的。

当满足以上两个条件时，InnoDB存储引擎会使用Insert Buffer，这样就能提高插入操作的性能了。不过考虑这样一种情况：应用程序进行大量的插入操作，这些都涉及了不唯一的非聚集索引，也就是使用了Insert Buffer。若此时MySQL数据库发生了宕机，这时势必有大量的Insert Buffer并没有合并到实际的非聚集索引中去。因此这时恢复可能需要很长的时间，在极端情况下甚至需要几个小时。

Insert buffer是一个B+ tree

### double write
如果说Insert Buffer带给InnoDB存储引擎的是性能上的提升，那么doublewrite（两次写）带给InnoDB存储引擎的是数据页的可靠性。

当发生数据库宕机时，可能InnoDB存储引擎正在写入某个页到表中，而这个页只写了一部分，比如16KB的页，只写了前4KB，之后就发生了宕机，这种情况被称为部分写失效（partial page write）。在InnoDB存储引擎未使用doublewrite技术前，曾经出现过因为部分写失效而导致数据丢失的情况。

MySQL 可以根据redo log 进行恢复，而mysql在恢复的过程中是检查page的checksum，checksum就是pgae的最后事务号，发生partial page write 问题时，page已经损坏，找不到该page中的事务号，就无法恢复。这就是说，在应用（apply）重做日志前，用户需要一个页的副本，当写入失效发生时，先通过页的副本来还原该页，再进行重做，这就是doublewrite。在InnoDB存储引擎中doublewrite的体系架构如图2-5所示。

在InnoDB将BP中的Dirty Page刷（flush）到磁盘上时，首先会将（memcpy函数）Page刷到InnoDB tablespace的一个区域中(在内存)，我们称该区域为Double write Buffer（大小为2MB，每次写入1MB，128个页）。在向Double write Buffer写入成功后，第二步、再将数据拷贝到数据文件对应的位置

当第二步过程中发生故障，也就是发生partial page write的问题。恢复的时候先检查页内的checksum是否相同，不一致，则直接从doublewrite中恢复

### 自适应哈希索引

哈希（hash）是一种非常快的查找方法，在一般情况下这种查找的时间复杂度为O(1)，即一般仅需要一次查找就能定位数据。而B+树的查找次数，取决于B+树的高度，在生产环境中，B+树的高度一般为3～4层，故需要3～4次的查询。

InnoDB存储引擎会监控对表上各索引页的查询。如果观察到建立哈希索引可以带来速度提升，则建立哈希索引，称之为自适应哈希索引（Adaptive Hash Index，AHI）。AHI是通过缓冲池的B+树页构造而来，因此建立的速度很快，而且不需要对整张表构建哈希索引。InnoDB存储引擎会自动根据访问的频率和模式来自动地为某些热点页建立哈希索引。

### 异步IO

为了提高磁盘操作性能，当前的数据库系统都采用异步IO（Asynchronous IO，AIO）的方式来处理磁盘操作。InnoDB存储引擎亦是如此。

与AIO对应的是Sync IO，即每进行一次IO操作，需要等待此次操作结束才能继续接下来的操作。但是如果用户发出的是一条索引扫描的查询，那么这条SQL查询语句可能需要扫描多个索引页，也就是需要进行多次的IO操作。在每扫描一个页并等待其完成后再进行下一次的扫描，这是没有必要的。用户可以在发出一个IO请求后立即再发出另一个IO请求，当全部IO请求发送完毕后，等待所有IO操作的完成，这就是AIO。

AIO的另一个优势是可以进行IO Merge操作，也就是将多个IO合并为1个IO，这样可以提高IOPS的性能。

### 刷新邻接页

InnoDB存储引擎还提供了Flush Neighbor Page（刷新邻接页）的特性。其工作原理为：当刷新一个脏页时，InnoDB存储引擎会检测该页所在区（extent）的所有页，如果是脏页，那么一起进行刷新。这样做的好处显而易见，通过AIO可以将多个IO写入操作合并为一个IO操作，故该工作机制在传统机械磁盘下有着显著的优势。

### 启动，关闭与恢复
InnoDB存储引擎是MySql数据库的存储引擎之一，因此InnoDB存储引擎的启动和关闭，或者说MySql数据库服务器的启动和关闭过程对InnoDB存储引擎的处理过程。 
在关闭时，参数 innodb_fast_shutdown影响着表的InnoDB存储引擎的行为，该参数可以为0，1，2，默认值为1

1. 为0时，表示在MySql数据库关闭时，InnoDB需要完成所有的full purge和merge insert buffer，并且将所有的脏页刷新回磁盘。这需要一些时间，有时甚至需要几个小时来完成。如果在进行InnoDB升级时，必须将这个参数设置为0，然后再关闭数据库。 
2. 1是参数 innodb_fast_shutdown的默认值，表示不需要完成上述的full purge和merge insert buffer操作，但是在缓冲池中的一些数据脏页还是会刷新回磁盘。 
3. 2表示不完成full purge和merge insert buffer操作，也不将缓冲池中的数据脏页写回磁盘，而是将日志都写入日志文件。这样不会有任何事务的丢失，但是下次MySql数据库启动时，会进行恢复操作。

当正常关闭MySql数据库时，下次的启动应该会非常正常。但是如果没有正常地关闭数据库，如用kill命令关闭数据库，在MySql数据库运行中重启了服务器，或者在关闭数据库时，将参数innodb_fast_shutdown设置为了2，下次MySql数据库启动时都会对InnoDB存储引擎的表进行恢复操作。

参数innodb_force_recovery影响了整个InnoDB存储引擎恢复的状态。该参数默认值为0，表示当发生需要恢复时，进行所有的恢复操作，当不能进行有效恢复时，如数据页发生了corruption，MySql数据库可能发生宕机（crash)，并把错误写入错误日志中去。

某些情况下，可能并不需要进行完整的恢复操作，因为用户自己知道怎么进行恢复。比如在对一个表进行alter table操作时发生了意外，数据库重启时会对InnoDB表进行回滚操作，对于一个大表来说这需要很长时间，可能是几个小时。这时用户可以自行进行恢复，如可以把表删除，从备份中重新导入数据到表，可能这些操作的速度要远远快于回滚操作。

参数innodb_force_recovery还可以设置为6个非零值：1-6，大的数字表示包含了前面所有小数字表示的影响。具体情况如下： 
1：忽略检查到的corrupt页。 
2：阻止Master Thread线程的运行，如Master Thread线程需要进行full purge，而这会导致crash。 
3：不进行事务的回滚操作。 
4：不进行插入缓冲的合并操作。 
5：不查看撤销日志（Undo Log），InnoDB存储引擎会将未提交的事务视为已提交。 
6：不进行前滚的操作。

参数innodb_force_recovery的值大于0时，可以对表进行select,creaete和drop操作，但是insert,update和delete这类DML操作是不允许的。

# 文件

## 日志文件

日志文件记录了影响MySQL数据库的各种类型活动。MySQL数据库中常见的日志文件有：
1. 错误日志（error log）
2. 二进制日志（binlog）
3. 慢查询日志（slow query log）
4. 查询日志（log）

这些日志文件可以帮助DBA对MySQL数据库的运行状态进行诊断，从而更好地进行数据库层面的优化。

### 错误日志

```sql
show variables like '%log_error%';
```
查看错误日志位置

### 慢查询日志
慢查询日志（slow log）可帮助DBA定位可能存在问题的SQL语句，从而进行SQL语句层面的优化。例如，可以在MySQL启动时设一个阈值，将运行时间超过该值的所有SQL语句都记录到慢查询日志文件中。DBA每天或每过一段时间对其进行检查，确认是否有SQL语句需要进行优化。该阈值可以通过参数long_query_time来设置，默认值为10，代表10秒。

在默认情况下，MySQL数据库并不启动慢查询日志，用户需要手工将这个参数设为ON

### 查询日志
查询日志记录了所有对MySQL数据库请求的信息，无论这些请求是否得到了正确的执行

### 二进制日志
binary log记录了对MySQL数据库执行更改的所有操作，但是不包括SELECT和SHOW这类操作。
总的来说，二进制日志主要有以下几种作用：

1. **恢复（recovery**）：某些数据的恢复需要二进制日志。例如，在一个数据库全备文件恢复后，用户可以通过二进制日志进行point-in-time的恢复。
2. **复制（replication）**：其原理与恢复类似，通过复制和执行二进制日志使一台远程的MySQL数据库（一般称为slave或者standby）与一台MySQL数据库（一般称为master或者primary）进行实时同步。
3. **审计（audit）**：用户可以通过二进制日志中的信息来进行审计，判断是否有对数据库进行注入攻击。

### 套接字文件
前面提到过，在UNIX系统下本地连接MySQL可以采用UNIX域套接字方式，这种方式需要一个套接字（socket）文件。

### pid文件
当MySQL实例启动时，会将自己的进程ID写入一个文件中——该文件即为pid文件。

### 表结构定义文件

因为MySQL插件式存储引擎的体系结构的关系，MySQL数据的存储是根据表进行的，每个表都会有与之对应的文件。但不论表采用何种存储引擎，MySQL都有一个以frm为后缀名的文件，这个文件记录了该表的表结构定义。
frm还用来存放视图的定义，如用户创建了一个v_a视图，那么对应地会产生一个v_a.frm文件，用来记录视图的定义，

## InnoDB存储引擎文件
之前介绍的文件都是MySQL数据库本身的文件，和存储引擎无关。除了这些文件外，每个表存储引擎还有其自己独有的文件。本节将具体介绍与InnoDB存储引擎密切相关的文件，这些文件包括重做日志文件、表空间文件。

### 表空间文件
InnoDB采用将存储的数据按表空间（tablespace）进行存放的设计。在默认配置下会有一个初始大小为10MB，名为ibdata1的文件。该文件就是默认的表空间文件（tablespace file）

### 重做日志文件
在默认情况下，在InnoDB存储引擎的数据目录下会有两个名为ib_logfile0和ib_logfile1的文件。在MySQL官方手册中将其称为InnoDB存储引擎的日志文件，不过更准确的定义应该是重做日志文件（redo log file）。为什么强调是重做日志文件呢？因为重做日志文件对于InnoDB存储引擎至关重要，它们记录了对于InnoDB存储引擎的事务日志。

当实例或介质失败（media failure）时，重做日志文件就能派上用场。例如，数据库由于所在主机掉电导致实例失败，InnoDB存储引擎会使用重做日志恢复到掉电前的时刻，以此来保证数据的完整性。

**也许有人会问，既然同样是记录事务日志，和之前介绍的二进制日志有什么区别？**

首先，二进制日志会记录所有与MySQL数据库有关的日志记录，包括InnoDB、MyISAM、Heap等其他存储引擎的日志。而InnoDB存储引擎的重做日志只记录有关该存储引擎本身的事务日志。

其次，记录的内容不同，无论用户将二进制日志文件记录的格式设为STATEMENT还是ROW，又或者是MIXED，其记录的都是关于一个事务的具体操作内容，即该日志是逻辑日志。而InnoDB存储引擎的重做日志文件记录的是关于每个页（Page）的更改的物理情况。

此外，写入的时间也不同，二进制日志文件仅在事务提交前进行提交，即只写磁盘一次，不论这时该事务多大。而在事务进行的过程中，却不断有重做日志条目（redo entry）被写入到重做日志文件中。

# 表

简单来说，表就是关于特定实体的数据集合，这也是关系型数据库模型的核心

## 索引组织表
在InnoDB存储引擎中，表都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表。在InnoDB存储引擎表中，每张表都有个主键，如果在创建表时没有显示地定义主键，则InnoDB存储引擎会按如下方式选择或创建主键。

首先判断表中是否有非空的唯一索引（Unique NOT NULL）,如果有，则该列即为主键。
如果不符合上述条件，InnoDB存储引擎自动创建一个6字节大小的指针。
当表中有多个非空唯一索引时，InnoDB存储引擎将选择建表时第一个定义的非空唯一索引为主键。这里需要注意的是，主键选择根据的是定义索引的顺序，而不是建表时列的顺序。

## InnoDB逻辑存储结构
从InnoDB存储引擎的逻辑存储结构看，所有数据都被逻辑地存放在一个空间中，称之为表空间（tablespace）。表空间又由段（segment）、区（extent）、页（page）组成。页在一些文档中有时也成为块（block），InnoDB存储引擎的逻辑存储结构大致如下图所示： 
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/innodb_stor.jpeg" width="500" height="500">
</div>

## InnoDB行记录格式
InnoDB存储引擎和大多数数据库一样，记录是以行的形式存储的。这意味着页中保存着表中一行行的数据。

数据库实例的作用之一就是读取页中存放的行记录。如果用户自己知道页中行记录的组织规则，也可以自行通过编写工具的方式来读取其中的记录，接下来将具体分析各格式存放数据的规则。

### Compact行记录格式
Compact行记录是在MySQL5.0中引入的，其设计目标是高校存储数据，简单来说，一个页中存放的行数据越多，其性能就越高。如下图所示： 

变长字段长度列表     NULL标志位      记录头信息    列1数据   列2数据

### 行溢出数据
InnoDB存储引擎可以将一条记录中的某些数据存储在真正的数据页面之外。一般认为BLOB、LOB这类的大对象列类型的存储会把数据存放在数据页面之外。但是，这个理解有点偏差，BLOB可以不将数据放在溢出页面，而且即便是VARCHAR列数据类型，依然有可能存放行溢出数据。

## InnoDB数据页结构
页是InnoDB存储引擎管理数据库的最小磁盘单位。页类型为B-tree Node的页存放的即是表中行的实际数据了。接下来，我们将从底层具体的探究InnoDB数据页的内部存储结构。

InnoDB数据页由一下7个部分组成：

File Header（文件头）
Page Header（页头）
Infimum和Supremum Records
user Records（用户记录，即行记录）
Free Space（空闲空间）
Page Directory（页目录）
File Trailer（文件结尾信息）
其中File Header、Page Header、File Trailer的大小是固定的，分别为38、56、8字节，这些空间用来标记该页的一些信息，如Checksum，数据页所在的B+树索引的层数等。User Records、Free Space、Page Directory 这些部分为实际的行记录存储空间，因此大小是动态的。 

## 分区表
### 分区概述
分区功能并不是在存储引擎层完成的，因此不是只有InnoDB存储引擎支持分区，常见的存储引擎MyISAM、DBA等都支持。但也并不是所有的存储引擎都支持，如CSV、FEDORATED、MERGE等就不支持。在使用分区功能前，应该对选择的存储引擎对分区的支持就有所了解。

分区的过程是将一个表或索引分解为多个更小、更可管理的的部分。就访问数据库的应用而言，从逻辑上讲，只有一个表或一个索引，但是在物理上这个表或索引可能由数十个物理分区组成。每个分区都是独立的对象，可以独自处理，也可以作为一个更大的对象的一部分进行处理。

MySQL支持的分区类型为水平分区，并不支持垂直分区。此外，MySQL数据库的分区是局部分区索引，一个分区中及存放了数据又存放了索引。而全局分区是指，数据存放在各个分区中，但是所有的数据的索引放在一个对象中。目前，MySQL数据库还不支持全局分区。

### 分区类型
1. RANGE分区 
RANGE分区主要用于日期列的分区，例如对于销售类的表，可以根据年来分区存放销售记录。 

2. LIST分区 
LIST分区和RANGE分区非常相似，只是分区列的值是离散的，而非连续的。

3. HASH分区 
HASHd的目的是将数据均匀地分布到预先定义的各个分区中，保证各分区的数据量大致都是一样的。在RANGE和LIST分区中，必须明确指定一个给定的列值或列值集合应该保存在哪个分区中；而在HASH分区中，MySQL自动完成这些工作，用户所要做的只是基于将要进行哈希分区的列值指定一个列值或表达式，以及指定被分区的表将要被分割成的分区数量。 

4. KEY分区 
KEY分区和HASH分区相似，不同之处在于HASH分区使用户定义的函数进行分区，KEY分区使用MySQL数据库提供的函数进行分区。对于NDB Cluster引擎，MySQL数据库使用MD5函数来分区；对于其他存储引擎，MySQL数据库使用其内部的哈希函数，这些函数基于与PASSWORD()一样的运算法则。 

5. COLUMNS分区 
COLUMNS分区可视为RANGE分区和LIST分区的一种进化。COLUMNS分区可以直接使用费整型的数据进行分区，分区根据分区类型直接比较而得，不需要转化为整型。

# 索引与算法

索引是应用程序设计和开发的一个重要方面。若索引太多，应用程序的性能可能会受到影响。而索引太少，对查询性能又会产生影响，要找到一个平衡点，这对应用程序的性能至关重要。

## InnoDB 存储引擎索引概述
InnoDB存储引擎支持以下几种常见的索引
* B+树索引
* 全文索引(Full-Text Search)
* 哈希索引

## B+ Tree索引和Hash索引区别 
* 哈希索引适合等值查询，但是不无法进行范围查询 哈希索引没办法利用索引完成排序 
* 哈希索引不支持多列联合索引的最左匹配规则 
* 如果有大量重复键值得情况下，哈希索引的效率会很低，因为存在哈希碰撞问题
  
## B+树
B+树索引有一个特点是高扇出性，因此在数据库中，B+树的高度一般在2到3层。也就是说查找某一键值的记录，最多只需要2到3次IO开销。
数据库中B+树索引分为聚集索引（clustered index)和非聚集索引（secondary index)，这两种索引的共同点是内部都是B+树，高度都是平衡的，叶节点存放着所有数据。不同点是叶节点是否存放着一整行数据。

### 聚集索引
聚集索引是指数据库表行中数据的物理顺序与键值的逻辑（索引）顺序相同。每张表只能有一个聚集索引（一个主键）。一个node包含一个row

聚集索引的好处是它对于主键的排序查找和范围的速度非常快。叶节点的数据就是我们要找的数据。聚集索引存的是node的实际的tuple

### 辅助索引
辅助索引（也称非聚集索引）。叶级别不包含行的全部数据，叶级别除了包含行的键值以外，每个索引行还包含了一个书签（bookmark），该书签告诉innodb存储引擎，哪里可以找到与索引对应的数据。
辅助索引的存在并不影响数据再聚集索引中的组织，因此一个表可以有多个辅助索引。the leaf node of the nonclustered index contains the clustered index keys.

### Covering Index
The nonclustered index contained all of the required information to resolve the query. No Key Lookups were required. An index that contains all information required to resolve the query is known as a “Covering Index”; it completely covers the query. 
例如,对A,B建立noncluster index。
```sql
select A,B from t where A = ''
```
这里，noncluster就不需要look up，因为本身key就已经包括了所有需要的值，因此，如果经常需要查询多个字段，多个字段就需要建立联合索引，避免触发look up

### Index Include Index
那么如果是
```sql
select A,B,C from t where A = ''
```
那么如果我就想要C，而且我也不想建立联合索引呢，那么可以使用Index Include Index，目前MS SQL Server，postgres11支持，mysql5.7不支持，sqlite不支持

Embed additional columns in index to support index-only queries.

### Function/Expression Indexes
Store the output of a function or expression as the key instead of the original
value. It is the DBMS’s job to recognize which queries can use that index.
```sql
SELECT * FROM users
WHERE EXTRACT(dow
FROM login) = 2;

CREATE INDEX idx_user_login --wrong
ON users (login);

CREATE INDEX idx_user_login
ON users (EXTRACT(dow FROM login));
```

### Single vs Composite Index
对于复合索引（多列b+tree，使用多列值组合而成的b+tree索引）。遵循最左侧原则，从左到右的使用索引中的字段，一个查询可以只使用索引中的一部份，但只能是最左侧部分。例如索引是key index (a,b,c). 可以支持a  a,b a,b,c 3种组合进行查找，但不支持 b,c进行查找。当使用最左侧字段时，索引就十分有效。

### Index失效的情况
[Index失效](https://blog.csdn.net/sc9018181134/article/details/78888022)

### Key Lookup 
SQL Server uses a Key Lookup to retrieve non-key data from the data page when a nonclustered index is used to resolve the query. That is, once SQL Server has used the nonclustered index to identify each row that matches the query criteria, it must then retrieve the column information for those rows from the data pages of the table.

## 全文检索(Full-Text Search)
全文检索是将储存于数据库中的整本书或整篇文章中的任意内容信息查找出来的技术。对每一个词建立一个索引， 指明该词在文中出现的次数和位置，当用户查询时，搜索程序就根据事先建立的索引进行查找，并将查找的结果反馈给用户。

### 倒排索引

其实就是inverted index，把每个单词在哪个部分出现储存，value可以表现为单词所在文档ID和在文档中的具体位置

# 锁

## Latches vs Locks
A page in SQL Server is 8KB and can store multiple rows. To increase concurrency and performance, buffer latches are held only for the duration of the physical operation on the page, unlike locks which are held for the duration of the logical transaction. Latches are internal to the SQL engine and are used to provide memory consistency, whereas locks are used by SQL Server to provide logical transactional consistency.

## LATCH MODES

### Read Mode
* Multiple threads are allowed to read the
same item at the same time.
* A thread can acquire the read latch if
another thread has it in read mode

### Write Mode
* Only one thread is allowed to access the
item.
* A thread cannot acquire a write latch if
another thread holds the latch in any mode.

## LATCH CRABBING/COUPLING
[reference](!https://15445.courses.cs.cmu.edu/fall2018/slides/09-indexconcurrency.pdf)
Protocol to allow multiple threads to
access/modify B+Tree at the same time.
Basic Idea:
* Get latch for parent.
* Get latch for child
* Release latch for parent if “safe”

A safe node is one that will not split or merge
when updated.
* Not full (on insertion)
* More than half-full (on deletion)

Search: Start at root and go down; repeatedly,
* Acquire R latch on child
* Then unlatch parent

Insert/Delete: Start at root and go down,
obtaining W latches as needed. Once child is
latched, check if it is safe:
* If child is safe, release all latches on ancestors.

## BETTER LATCHING ALGORITHM
Assume that the leaf node is safe.
Use read latches and crabbing to reach
it, and verify that it is safe.
If leaf is not safe, then do previous
algorithm using write latches.

Search: Same as before.

Insert/Delete:
* Set latches as if for search, get to leaf, and set W latch on
leaf.
* If leaf is not safe, release all latches, and restart thread
using previous insert/delete protocol with write latches.

This approach optimistically assumes that only leaf
node will be modified; if not, R latches set on the
first pass to leaf are wasteful.

## LEAF NODE SCAN
如果是在横向的叶子上找，例如查找val<4，可能就会到达叶子val为4节点，然后继续向左查找，这时候如果是search的话，两个thread会交换latch
如果一个是读，一个是写，则kill themselves

## 锁的类型
行锁
* 共享锁，允许事务读取一行数据
* 排他锁，允许事务删除或更新一行数据

# 行锁的3种算法
* Record Lock: 单个行记录上的锁
* Gap Lock：间隙锁，锁定一个范围，但不包含记录本身
* Next-Key Lock：Gap Lock + Record Lock，锁定一个范围，并且锁定记录本身

# 事务

## 事务类型
* 扁平事务（Flat Transactions）
* 带有保存点的扁平事务（Flat Transactions with Savepoints）
* 链事务（Chained Transactions）
* 嵌套事务（Nested Transactions）
* 分布式事务（Distributed Transactions）

## 事务的实现

事务隔离由锁实现，原子性，一致性，持久性通过数据库的redo log和undo log来完成

### MVCC
如果没有并发控制，那么如果同时有用户读写数据，那么可能出现读出的数据不一致的情况。比如说，进行银行账户A到B的转账，当A账户的钱被扣掉，而钱还没有加到B账户，此时用户查看自己的余额，会感觉钱凭空消失了。MySQL的隔离性就是用来解决这类问题的，而隔离性是通过不同的并发控制手段来实现的。对于刚才的问题，一种简单的并发控制方式，就是讲读写操作串行化，在账户间转账时，不允许查询账户，虽然这种方式可以解决问题，但无疑过于简单粗暴，效率极低。相比于串行化的并发控制，MVCC的优势在于读写影响，对于现代互联网读多写少的场景，这种方式性能明显更高。

MVCC是通过保存数据的多个版本来实现并发控制，当需要更新某条数据时，实现了MVCC的存储系统不会立即用新数据覆盖原始数据，而是创建该条记录的一个新的版本。对于多数数据库系统，存储会分为Data Part和Undo Log，Data Part用来存储事务已提交的数据，而Undo Log用来存储旧版本的数据。多版本的存在允许了读和写的分离，读操作是需要读取某个版本之前的数据即可，和写操作不冲突，大大提高了性能。

[innoDB在RR下解决了幻读?]https://juejin.im/post/6844903799534911496
[Innodb中的事务隔离级别和锁的关系](https://tech.meituan.com/2014/08/20/innodb-lock.html)

innoDB主要是通过MVCC进行隔离的, 在RR下会用Next-Key Lock解决幻读, 在S下会直接写用排他锁, 读用共享锁, 在Next-key Lock里

# sort and group by

## EXTERNAL MERGE SORT
**Sorting Phase**

Sort small chunks of data that fit in main-memory, and
then write back the sorted data to a file on disk.

**Merge Phase**

Combine sorted sub-files into a single larger file.

group by 可以用sort去重，或者hash去重，hash是把val映射到disk里，如果是相同的值，那value肯定相同，就相当于合并，最后再一起拿出来，主要用于处理大数据的