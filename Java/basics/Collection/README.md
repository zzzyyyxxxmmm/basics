# HashMap

## 特点 
允许null key 和null value

## 如何Hash的
•	首先得到Key的Hashcode，并且将高16位和低16位亦或的到hash，加大了随机性，减少了碰撞, 实际上null的hash就是0
•	由于table长度的限制，最后实际在table中的位置是hash&(n-1)，保留后几位

## 如何putval
找到在table中的对应位置，插入node，如果发现大于树化的threshold的话，就进行树化。如果发现插入的元素重复了，则根据一个参数决定是否更新value的值，最后检查table的大小决定是否resize
