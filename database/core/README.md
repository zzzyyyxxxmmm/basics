# Lock
## TWO-PHASE LOCKING

**Phase #1: Growing**

* Each txn requests the locks that it needs from the DBMS’s
lock manager.
* The lock manager grants/denies lock requests.
**Phase #2: Shrinking**

* The txn is allowed to only release locks that it previously
acquired. It cannot acquire new locks.

The txn is not allowed to acquire/upgrade locks
after the growing phase finishes.


## INTENTION LOCKS
An intention lock allows a higher level node to
be locked in shared or exclusive mode without
having to check all descendent nodes.
If a node is in an intention mode, then explicit
locking is being done at a lower level in the tree.

[COMPATIBILITY MATRIX](!https://github.com/zzzyyyxxxmmm/basics/blob/master/image/lock_compa.png)

# MULTI-VERSION CONCURRENCY CONTROL


# Optimization

分析查询语句:
* Postgres/SQLite: ANALYZE
* Oracle/MySQL: ANALYZE TABLE
* SQL Server: UPDATE STATISTICS
* DB2: RUNSTATS

## MYSQL 执行语句流程
[流程](!https://juejin.im/post/5b7036de6fb9a009c40997eb)
1. SQL statement enter to the server
2. check the cache (deprecated in 8.0)
3. enter parser and get Abstract Syntax Tree
4. enter to binder(Name→Internal ID), check the name is valid or not.
5. go to Optimizer

## Join Algorithm

### SIMPLE NESTED LOOP JOIN

这种两层的join, 并不是简单的n*m=m*n, 由于涉及到IO, 交换loop会导致完全不一样的结果, 最终的cost是以IO为单位

例如T1包含M page, m tuples, T2包含N pages, n tuples. 如果T1在外层, 那么T1要进行M次IO, 对于T1中的每个Tuple, 都要读取一次完整的T2, 总共需要M+(m*N)次IO, 如果反过来则是N+(n*M)

Example database:
* M = 1000, m = 100,000
* N = 500, n = 40,000 

### BLOCK NESTED LOOP JOIN

这个就是对于T1中的每个page查询T2, 因此总共是M+(M*N)

如果使用了buffer, 那么T1每次换block的时候,T2就不需要重新遍历一遍了, 因此所有的block都主要读一遍, 即为M+N, 注意这里的buffer就需要很大了, 通常不太可能

上面这两种时间复杂度都是(n*m)

### INDEX NESTED LOOP JOIN

由于扫描T2的时候都是sequential scan, 因此这里我们可以通过index加速, 由于使用B+tree, sequential scan可以默认为常数, 最后M+(m*C), 由结果可以看到, outer table的size应该尽量的小, 时间复杂度降低为 m\*logn.

### SORT-MERGE JOIN

**Phase #1: Sort**
* Sort both tables on the join key(s).
* Can use the external merge sort algorithm that we talked
about last class.

**Phase #2: Merge**
* Step through the two sorted tables in parallel, and emit
matching tuples.
* May need to backtrack depending on the join type.

复杂度: 

**Sort Cost (R):** 2M ∙ (log M / log B)
**Sort Cost (S):** 2N ∙ (log N / log B)
**Merge Cost:** (M + N)

The worst case for the merging phase is when the
join attribute of all of the tuples in both relations
contain the same value.

**Cost:** (M ∙ N) + (sort cost)

One or both tables are already sorted on join key.
Output must be sorted on join key.

### BASIC HASH JOIN ALGORITHM
就是对外表建一个hash, 然后就不用一个一个直接比较了

但明显一个问题就是hash不够大, 发生碰撞, 这时候就需要用到grace hash join, 大概就是发生碰撞的再hash第二次.

**Partitioning Phase:**
* Read+Write both tables
* 2(M+N) IOs

**Probing Phase:**
* Read both tables
* M+N IOs

| Algorithm               | IO Cost             | Example      |
|-------------------------|---------------------|--------------|
| Simple Nested Loop Join | M+(m*N)             | 1.3 hours    |
| Block Nested Loop Join  | M + (M ∙ N)         | 50 seconds   |
| Index Nested Loop Join  | M + (m ∙ C)         | ~20 seconds  |
| Sort-Merge Join         | M + N + (sort cost) | 0.59 seconds |
| Hash Join               | 3(M + N)            | 0.45 seconds |

# Cache

show variables like '%query_cache%'; 

mysql从5.6开始就已经默认关闭cache了, mysql8.0目前已经放弃支持cache, 主要原因是因为缓存命中难度大,命中率低,扩展性低. 修改一个表会导致整个缓存失效. 当前的替代方案为第三方缓存例如sqlproxy. [详情](!https://mysqlserverteam.com/mysql-8-0-retiring-support-for-the-query-cache/)