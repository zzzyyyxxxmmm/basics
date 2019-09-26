# 什么是微服务

As the project grows bigger, it becomes hard to maintain:
1. Slow to run
2. Slow to modify and deploy
3. Availibity problem. One small bug will shut down the whole project.

Microservice is an idea to devide the project into small services. The service expose API or use RPC to connect to each other. They normally deploy on VM or docker.

## Advantage of microservice
1. Solve complicated problem. It devide the project into small modules.
2. Easy to develop. Each module can be developed by a single team. They can choose tech stack they like without considering other modules. And refracting become feasible.
3. Easy to deploy. Deploy a single module without interfere other modules.
4. Scalable. Easy to add new service or add new server for some specific services.

## Disadvantage of microservice
1. MS is based on distribute system. Although a single module is easy, the whole system is hard to maintain. We need to take care of Exception handling, info-communicating.
2. Distribute database. CAP theory
3. Take care dependency. Modify one module may influence other modules.
4. hard to delopy. There so many small service to deploy. 一种自动化方案就是适用Paas, 例如AWS. 另一种就是kubernates

# API GateWay

load balancer, cache, access controll. (Nginx)

# Microservice design pattern

## Circuit Breaker Pattern
If a service shutdown because of some bugs, other service will try to visit the failed service again and again, so we need to prevent a network or service failure from cascading to other services.

A service client should invoke a remote service via a proxy that functions in a similar fashion to an electrical circuit breaker. When the number of consecutive failures crosses a threshold, the circuit breaker trips, and for the duration of a timeout period all attempts to invoke the remote service will fail immediately. After the timeout expires the circuit breaker allows a limited number of test requests to pass through. If those requests succeed the circuit breaker resumes normal operation. Otherwise, if there is a failure the timeout period begins again.

在分布式环境中，我们的应用可能会面临着各种各样的可恢复的异常（比如超时，网络环境异常），此时我们可以利用不断重试的方式来从异常中恢复(Retry Pattern)，使整个集群正常运行。

然而，也有一些异常比较顽固，突然发生，无法预测，而且很难恢复，并且还会导致级联失败（举个例子，假设一个服务集群的负载非常高，如果这时候集群的一部分挂掉了，还占了很大一部分资源，整个集群都有可能遭殃）。如果我们这时还是不断进行重试的话，结果大多都是失败的。因此，此时我们的应用需要立即进入失败状态(fast-fail)，并采取合适的方法进行恢复。

Circuit Breaker Pattern（熔断器模式）就是这样的一种设计思想。它可以防止一个应用不断地去尝试一个很可能失败的操作。一个Circuit Breaker相当于一个代理，用于监测某个操作对应的失败比率(fail / fail + success)。它会根据得到的数据来决定是否允许执行此操作，或者是立即抛出异常。

我们可以用状态机来实现 Circuit Breaker，它有以下三种状态：

**关闭(Closed)**：默认情况下Circuit Breaker是关闭的，此时允许操作执行。Circuit Breaker内部记录着最近失败的次数，如果对应的操作执行失败，次数就会续一次。如果在某个时间段内，失败次数（或者失败比率）达到阈值，Circuit Breaker会转换到开启(Open)状态。在开启状态中，Circuit Breaker会启用一个超时计时器，设这个计时器的目的是给集群相应的时间来恢复故障。当计时器时间到的时候，Circuit Breaker会转换到半开启(Half-Open)状态。

**开启(Open)**：在此状态下，执行对应的操作将会立即失败并且立即抛出异常。

**半开启(Half-Open)**：在此状态下，Circuit Breaker 会允许执行一定数量的操作。如果所有操作全部成功，Circuit Breaker就会假定故障已经恢复，它就会转换到关闭状态，并且重置失败次数。如果其中 任意一次 操作失败了，Circuit Breaker就会认为故障仍然存在，所以它会转换到开启状态并再次开启计时器（再给系统一些时间使其从失败中恢复）

## Graceful degradation


# Distribute Database

## Why
移动互联网时代，海量的用户每天产生海量的数量，比如：用户表、订单表、交易流水表。

以支付宝用户为例，8亿；微信用户更是10亿。订单表更夸张，比如美团外卖，每天都是几千万的订单。淘宝的历史订单总量应该百亿，甚至千亿级别，这些海量数据远不是一张表能Hold住的。

事实上MySQL单表可以存储1billion亿级数据，只是这时候性能比较差，业界公认MySQL单表容量在10 million以下是最佳状态，因为这时它的BTREE索引树高在3~5之间。

## How
### 分区 partition
Parition break the table into multiple entities, but it's still a unbroken table in a logic way. When a query comes, it will check a table who will decide which partition the query will go. Partions will share the same memeory, CPU resourses, which means they run in one machine. I think it's same as the sharind in a scale out way.
分区并没有将表拆分, 而只是数据**分段**划分在多个位置存放

什么时候用分区
1. sql经过优化
2. 数据量大
3. 表中的数据是分段的
4. 对数据的操作往往只涉及一部分数据，而不是所有的数据

**缺点**: 很多的资源都受到单机的限制，例如连接数，网络吞吐等！虽然每个分区可以独立存储，但是分区表的总入口还是一个MySQL示例。从而导致它的并发能力非常一般，远远达不到互联网高并发的要求！
  
### 分库分表 sharding  
**scale up** means break the table and database into different logic part or function part. For example, break the database into user database, order database. The table structure in different database is different.

**scale out** means break the table into different block based on time or id.


### NoSQL/NewSQL


## 读写分离(master/slave)
上面主要解决存储压力, 主从同步用于解决读写压力,

每个服务器上存储的数据相同, 读性能提升, 写性能不变, 需要进行数据同步
