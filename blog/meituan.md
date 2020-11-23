# 以后再看
### [MySQL索引原理及慢查询优化](https://tech.meituan.com/2014/06/30/mysql-index.html)


### 基于Flume的美团日志收集系统(一)架构和设计(https://tech.meituan.com/2013/12/09/meituan-flume-log-system-architecture-and-design.html)

系统基于flume开发, 直接看架构图就明白了, 创新点在于DualChannel的使用, 根据流量大小可以使用不同的channel

Flume uses a transactional approach to guarantee the reliable delivery of the events. 

### [基于Flume的美团日志收集系统(二)改进和优化](https://tech.meituan.com/2013/12/09/meituan-flume-log-system-optimization.html)
调优经验还是比较值得一看的, 基本上都是因地制宜, 结合实际情况进行调优

### [OpenTSDB 造成 Hbase 整点压力过大问题的排查和解决](https://tech.meituan.com/2014/09/16/opentsdb-hbase-compaction-problem.html)
OpenTSDB在整点期间会出现高峰, 然后下降, 原因是OpenTSDB会在整点进行压缩, 修改压缩速度解决问题, 作者的分析思路:
1. 谁读取了openTSDB, 分析他们的流量
2. 逐渐缩小范围, 确定问题根源所在

### [Quartz应用与集群原理分析](https://tech.meituan.com/2014/08/31/mt-crm-quartz.html)
美团CRM系统的任务调度模块经历了以下历史方案:
1. Crontab+SQL
* 直接访问数据库，各系统业务接口没有重用。
* 完成复杂业务需求时，会引入过多中间表。
* 业务逻辑计算完全依赖SQL，增大数据库压力。
* 任务失败无法自动恢复。
2. Python+SQL
* 直接访问数据，需要理解各系统的数据结构，无法满足动态任务问题，各系统业务接口没有重用。
* 无负载均衡。
* 任务失败无法恢复。
* 在JAVA语言开发中出现异构，且很难统一到自动部署系统中。
3. Spring+JDK Timer
* 步骤复杂、分散，任务量增大的情况下，很难扩展
* 使用写死服务器Host的方式执行task，存在单点风险，负载均衡手动完成。
* 应用重启，任务无法自动恢复。

最主要的问题还是无法扩展

**Quartz集群原理分析**
Quartz的集群部署方案在架构上是分布式的，没有负责集中管理的节点，而是利用数据库锁的方式来实现集群环境下进行并发控制

在Quartz中有两类线程：Scheduler调度线程和任务执行线程。*任务执行线程*：Quartz不会在主线程(QuartzSchedulerThread)中处理用户的Job。Quartz把线程管理的职责委托给ThreadPool，一般的设置使用SimpleThreadPool。SimpleThreadPool创建了一定数量的WorkerThread实例来使得Job能够在线程中进行处理。WorkerThread是定义在SimpleThreadPool类中的内部类

每当要进行与某种业务相关的数据库操作时，先去QRTZ_LOCKS表中查询操作相关的业务对象所需要的锁，在select语句之后加for update来实现. 当一个线程使用上述的SQL对表中的数据执行查询操作时，若查询结果中包含相关的行，数据库就对该行进行ROW LOCK；若此时，另外一个线程使用相同的SQL对表的数据进行查询，由于查询出的数据行已经被数据库锁住了，此时这个线程就只能等待，直到拥有该行锁的线程完成了相关的业务操作，执行了commit动作后，数据库才会释放了相关行的锁，这个线程才能继续执行。

The SELECT FOR UPDATE statement is used to order transactions by controlling concurrent access to one or more rows of a table. It works by locking the rows returned by a selection query, such that other transactions trying to access those rows are forced to wait for the transaction that locked the rows to finish.

[Presto实现原理和美团的使用实践](https://tech.meituan.com/2014/06/16/presto.html)
