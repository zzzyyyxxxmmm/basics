# 安装与命令

```brew install redis```

## Mac

### run with and without backgroud service
```
brew services start redis
brew services stop redis
```

```
redis-server /usr/local/etc/redis.conf
```

redis-server /usr/local/etc/redis.conf
/etc/init.d/redis-server stop

### open client
```
redis-cli
```



# 什么是redis
Redis 是一个使用 C 语言写成的，开源的 key-value 数据库。。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。目前，Vmware在资助着redis项目的开发和维护。

# Commands

## Strings

| Command     | Example use and description                                                                                                                                                       |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| INCR        | INCR key-name—Increments the value stored at the key by 1                                                                                                                         |
| DECR        | DECR key-name—Decrements the value stored at the key by 1                                                                                                                         |
| INCRBY      | INCRBY key-name amount—Increments the value stored at the key by the provided integer value                                                                                       |
| DECRBY      | DECRBY key-name amount—Decrements the value stored at the key by the provided integer value                                                                                       |
| INCRBYFLOAT | INCRBYFLOAT key-name amount—Increments the value stored at the key by the provided float value (available in Redis 2.6 and later)                                                 |
| APPEND      | APPEND key-name value—Concatenates the provided value to the string already stored at the given key                                                                               |
| GETRANGE    | GETRANGE key-name start end—Fetches the substring, including all charac- ters from the start offset to the end offset, inclusive                                                  |
| SETRANGE    | SETRANGE key-name offset value—Sets the substring starting at the pro- vided offset to the given value                                                                            |
| GETBIT      | GETBIT key-name offset—Treats the byte string as a bit string, and returns the value of the bit in the string at the provided bit offset                                          |
| SETBIT      | SETBIT key-name offset value—Treats the byte string as a bit string, and sets the value of the bit in the string at the provided bit offset                                       |
| BITCOUNT    | BITCOUNT key-name [start end]—Counts the number of 1 bits in the string, optionally starting and finishing at the provided byte offsets                                           |
| BITOP       | BITOP operation dest-key key-name [key-name ...]—Performs one of the bitwise operations, AND, OR, XOR, or NOT, on the strings provided, storing the result in the destination key |

## List
| Command    | Example use and description                                                                                                                                                                                       |
|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| RPUSH      | RPUSHkey-namevalue[value...]—Pushes the value(s) onto the right end of the list                                                                                                                                   |
| LPUSH      | LPUSHkey-namevalue[value...]—Pushes the value(s) onto the left end of the list                                                                                                                                    |
| RPOP       | RPOPkey-name—Removes and returns the rightmost item from the list                                                                                                                                                 |
| LPOP       | LPOPkey-name—Removes and returns the leftmost item from the list                                                                                                                                                  |
| LINDEX     | LINDEX key-name offset—Returns the item at the given offset                                                                                                                                                       |
| LRANGE     | LRANGEkey-namestartend—Returns the items in the list at the offsets from start to end, inclusive                                                                                                                  |
| LTRIM      | LTRIMkey-namestartend—Trims the list to only include items at indices between start and end, inclusive                                                                                                            |
| BLPOP      | BLPOP key-name [key-name ...] timeout—Pops the leftmost item from the first non-empty LIST, or waits the timeout in seconds for an item                                                                           |
| BRPOP      | BRPOP key-name [key-name ...] timeout—Pops the rightmost item from the first non-empty LIST, or waits the timeout in seconds for an item                                                                          |
| RPOPLPUSH  | RPOPLPUSH source-key dest-key—Pops the rightmost item from the source and LPUSHes the item to the destination, also returning the item to the user                                                                |
| BRPOPLPUSH | BRPOPLPUSH source-key dest-key timeout—Pops the rightmost item from the source and LPUSHes the item to the destination, also returning the item to the user, and waiting up to the timeout if the source is empty |

## Set
| Sets        | Example use and description                                                                                                                                                                                                                                       |
|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SADD        | SADD key-name item [item ...]—Adds the items to the set and returns the number of items added that weren’t already present                                                                                                                                        |
| SREM        | SREM key-name item [item ...]—Removes the items and returns the number of items that were removed                                                                                                                                                                 |
| SISMEMBER   | SISMEMBER key-name item—Returns whether the item is in the SET                                                                                                                                                                                                    |
| SCARD       | SCARD key-name—Returns the number of items in the SET                                                                                                                                                                                                             |
| SMEMBERS    | SMEMBERS key-name—Returns all of the items in the SET as a Python set                                                                                                                                                                                             |
| SRANDMEMBER | SRANDMEMBER key-name [count]—Returns one or more random items from the SET. When count is positive, Redis will return count distinct randomly cho- sen items, and when count is negative, Redis will return count randomly chosen items that may not be distinct. |
| SPOP        | SPOP key-name—Removes and returns a random item from the SET                                                                                                                                                                                                      |
| SMOVE       | SMOVE source-key dest-key item—If the item is in the source, removes the item from the source and adds it to the destination, returning if the item was moved                                                                                                     |
| SDIFF       | SDIFF key-name [key-name ...]—Returns the items in the first SET that weren’t in any of the other SETs (mathematical set difference operation)                                                                                                                    |
| SDIFFSTORE  | SDIFFSTORE dest-key key-name [key-name ...]—Stores at the dest-key the items in the first SET that weren’t in any of the other SETs (math- ematical set difference operation)                                                                                     |
| SINTER      | SINTER key-name [key-name ...]—Returns the items that are in all of the SETs (mathematical set intersection operation)                                                                                                                                            |
| SINTERSTORE | SINTERSTORE dest-key key-name [key-name ...]—Stores at the dest-key the items that are in all of the SETs (mathematical set intersection operation)                                                                                                               |
| SUNION      | SUNION key-name [key-name ...]—Returns the items that are in at least one of the SETs (mathematical set union operation)                                                                                                                                          |
| SUNIONSTORE | SUNIONSTORE dest-key key-name [key-name ...]—Stores at the dest-key the items that are in at least one of the SETs (mathematical set union operation)                                                                                                             |

## Hash
| Command      | Example use and description                                                                                                    |
|--------------|--------------------------------------------------------------------------------------------------------------------------------|
| HMGET        | HMGET key-name key [key ...]—Fetches the values at the fields in the HASH                                                      |
| HMSET        | HMSET key-name key value [key value ...]—Sets the values of the,fields in the HASH                                             |
| HDEL         | HDEL key-name key [key ...]—Deletes the key-value pairs in the HASH,,returning the number of pairs that were found and deleted |
| HLEN         | HLEN key-name—Returns the number of key-value pairs in the HASH                                                                |
| HEXISTS      | HEXISTS key-name key—Returns whether the given key exists in the HASH                                                          |
| HKEYS        | HKEYS key-name—Fetches the keys in the HASH                                                                                    |
| HVALS        | HVALS key-name—Fetches the values in the HASH                                                                                  |
| HGETALL      | HGETALL key-name—Fetches all key-value pairs from the HASH                                                                     |
| HINCRBY      | HINCRBY key-name key increment—Increments the value stored at the given key by the integer increment                           |
| HINCRBYFLOAT | HINCRBYFLOAT key-name key increment—Increments the value stored at the given key by the float increment                        |

## Sorted sets

| Command          | Example use and description                                                                                                                           |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| ZADD             | ZADD key-name score member [score member ...]—Adds members with the given scores to the ZSET                                                          |
| ZREM             | ZREM key-name member [member ...]—Removes the members from the ZSET, returning the number of members that were removed                                |
| ZCARD            | ZCARD key-name—Returns the number of members in the ZSET                                                                                              |
| ZINCRBY          | ZINCRBY key-name increment member—Increments the member in the ZSET                                                                                   |
| ZCOUNT           | ZCOUNT key-name min max—Returns the number of members with scores between the provided minimum and maximum                                            |
| ZRANK            | ZRANK key-name member—Returns the position of the given member in the ZSET                                                                            |
| ZSCORE           | ZSCORE key-name member—Returns the score of the member in the ZSET                                                                                    |
| ZRANGE           | ZRANGE key-name start stop [WITHSCORES]—Returns the members and optionally the scores for the members with ranks between start and stop               |
| ZREVRANK         | ZREVRANK key-name member—Returns the position of the member in the ZSET, with members ordered in reverse                                              |
| ZREVRANGE        | ZREVRANGE key-name start stop [WITHSCORES]—Fetches the given members from the ZSET by rank, with members in reverse order                             |
| ZRANGEBYSCORE    | ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]—Fetches the members between min and max                                                   |
| ZREVRANGEBYSCORE | ZREVRANGEBYSCOREkeymaxmin[WITHSCORES][LIMIToffset count]—Fetches the members in reverse order between min and max                                     |
| ZREMRANGEBYRANK  | ZREMRANGEBYRANK key-name start stop—Removes the items from the ZSET with ranks between start and stop                                                 |
| ZREMRANGEBYSCORE | ZREMRANGEBYSCOREkey-nameminmax—Removes the items from the ZSET with scores between min and max                                                        |
| ZINTERSTORE      | ZINTERSTORE dest-key key-count key [key ...] [WEIGHTS weight[weight...]][AGGREGATESUM|MIN|MAX]—Performs a SET-like intersection of the provided ZSETs |
| ZUNIONSTORE      | ZUNIONSTORE dest-key key-count key [key ...] [WEIGHTS weight[weight...]][AGGREGATESUM|MIN|MAX]—Performs a SET-like union of the provided ZSETs        |



# 使用redis实现分布式锁

(redis blog)[https://redis.io/topics/distlock]