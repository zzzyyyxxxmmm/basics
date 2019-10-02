# 每日练习

## [leetcode 208 Trie](https://leetcode.com/problems/implement-trie-prefix-tree/)

## [leetcode 399 union find](https://leetcode.com/problems/evaluate-division/)

## [lintcode 892 topo sort](https://www.lintcode.com/problem/alien-dictionary/description)

## [leetcode 84 monotonous stack](https://leetcode.com/problems/largest-rectangle-in-histogram/)

# 背包问题

### 01背包
典型的dp问题, 同其他dp问题一样, 抉择在于是否选择第i个物品
```java
dp[i][j]=max(dp[i-1][j],dp[i-1][j-C[i]]+W[i])
```
由于只能选择一个, 因此需要从V开始遍历, 因为这样的话dp[i-1][j-C[i]]将始终为0

**滚动数组**
由于只依赖左上方的, 因此可以省略一个loop

**如何初始化?**

如果不要求装满的话, 则初始化为0即可, 如果要求刚好装满, 则初始化为-INF

### 完全背包
由于可以无限选择, 因此需要从C[i]开始遍历, 新计算出来的值之后会继续用到

### 方案数
dp[0][0]初始化为1, max改成sum

# 常见算法

### 返回和
rangeSum还是以1开头比较好

## Interval
基本思路都是过滤掉不相交的, 相交的再求
### [Interval List Intersections] (https://leetcode.com/problems/interval-list-intersections/)

# 模板

## upperBound & lowerBound
```java
int upperBound(List<Integer> list,int x){
    int l=0;
    int r=list.size()-1;
    int ans=-1;
    while(l<=r){
        int mid=(l+r)>>1;
        if(list.get(mid)<=x){
            l=mid+1;
        } else if(list.get(mid)>x){
            r=mid-1;
            ans=mid;
        }
    }

    return ans;
}

int lowerBound(List<Integer> list,int x){
    int l=0;
    int r=list.size()-1;
    int ans=-1;
    while(l<=r){
        int mid=(l+r)>>1;
        if(list.get(mid)<x){
            l=mid+1;
            ans=mid;
        } else if(list.get(mid)>x){
            r=mid-1;
        } else {
            r=mid-1;
            ans=mid;
        }
    }

    return ans;
}
```