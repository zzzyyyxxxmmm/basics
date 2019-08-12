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

