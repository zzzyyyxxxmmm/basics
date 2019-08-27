# HashMap

## 特点 
允许null key 和null value

## 如何Hash的
•	首先得到Key的Hashcode，并且将高16位和低16位亦或的到hash，加大了随机性，减少了碰撞, 实际上null的hash就是0
•	由于table长度的限制，最后实际在table中的位置是hash&(n-1)，保留后几位
String 是每一位*31，左移，然后加起来
Integer的hashcode就是自己本身
如果自己写hashcode的话，直接自动生成就好了，他是调用了Objects.hash(),也是31左移加

## 如何putval
找到在table中的对应位置，插入node，如果发现大于树化的threshold的话，就进行树化。如果发现插入的元素重复了，则根据一个参数决定是否更新value的值，最后检查table的大小决定是否resize

## 如何getval
先比较hashcode,再用equals比较key，因为hashcode比较快

## 常见Hash Function

### CRC32

### SHA256

### MurmurHash

### Goolge CityHash

### Goolge FarmHash

### CLHash

## STATIC HASHING SCHEMES
* Linear Probe Hashing
* Robin Hood Hashing
* Cuckoo Hashing

### LINEAR PROBE HASHING
Single giant table of slots.
Resolve collisions by linearly searching for the
next free slot in the table.
* To determine whether an element is present, hash to a
location in the index and scan for it.
* Have to store the key in the index to know when to stop
scanning.
* Insertions and deletions are generalizations of lookups.

### ROBIN HOOD HASHING
Variant of linear probe hashing that steals slots
from "rich" keys and give them to "poor" keys.
* Each key tracks the number of positions they are from
where its optimal position in the table.
* On insert, a key takes the slot of another key if the first
key is farther away from its optimal position than the
second key.

A 0
B 0
C 1
最后一步C要插到B的位置，但由于B[1]>C[0],因此C需要向后插

| | | |  ->  |A[0]| | |  ->  |A[0]|B[1]| |  ->  |A[0]|B[1]|C[1]|

### CUCKOO HASHING
Use multiple hash tables with different hash
functions.
* On insert, check every table and pick anyone that has a
free slot.
* If no table has a free slot, evict the element from one of
them and then re-hash it find a new location.

If we find a cycle, then we can rebuild the entire
hash tables with new hash functions.
* With two hash functions, we (probably) won’t need to
rebuild the table until it is at about 50% full.
* With three hash functions, we (probably) won’t need to
rebuild the table until it is at about 90% full.

## Dynamic hash tables

### Chained Hashing
Maintain a linked list of buckets for each slot in
the hash table.
Resolve collisions by placing all elements with the
same hash key into the same bucket.
* To determine whether an element is present, hash to its
bucket and scan for it.
* Insertions and deletions are generalizations of lookups.

### EXTENDIBLE HASHING
* Chained-hashing approach with buckets.
* Instead of letting the linked list of buckets grow indefinitely, we’re going to split them incrementally.
* When a bucket is full, we split the bucket and reshuffle its elements.
* Uses global and local depths to determine buckets
* Hash table doubles in size to allow for more buckets

### Linear Hashing
* Maintain a pointer that tracks the next bucket to split.
* Overflow criterion is left up to the implementation.
* When any bucket overflows, split the bucket at the pointer location by adding a new slot entry, and create a new hash function.
* If hash function maps to slot that has previously been pointed to by pointer, apply the new hash function.
* When pointer reaches last slot, delete original hash function and replace it with new hash function.

# ConcurrentHashMap


# ArrayMap
ArrayMap是一种通用的key-value映射的数据结构，旨在提高内存效率，它与传统的HashMap有很大的不同。它将其映射保留在数组数据结构中：
两个数组（其中一个存放每个item的hash值的整数数组，以及key/value对的Object数组）。
这避免了它为放入映射的每个item创建额外的对象，并且它还积极地控制这些数组的增长。
数组的增长只需要复制数组中的item，而不是重建hash映射。

## 特点：
* ArrayMap是Android特有的api，用在移动端，所以它主要是提高内存效率。
* ArrayMap比传统的HashMap慢，所以ArrayMap不适合包含大数据的处理，因为添加和删除元素的时候需要使用二分搜索来查找元素。
* ArrayMap会在remove item的时候收缩数组。
* ArrayMap不是线程安全的。

# ArrayList

ArrayList满了才会扩容，扩容后的大小为原大小的1.5倍，即增加了50%
元素hash值第N+1位为0：不需要进行位置调整
元素hash值第N+1位为1：调整至原索引+oldcap处