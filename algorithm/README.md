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

## 单调栈
```java
for(int i=0;i<n;i++){
    while(!st.empty()&&A[st.top()]>=A[i]) st.pop();
    if(st.empty())   left.push_back(0);
    else left.push_back(st.top()+1);
    st.push(i);
}
```
核心是求从某个元素开始, 最多能向两边扩展多少, 整个延展应该是一个波谷形

栈应该是单调递增的, 去掉所有左边比他大的,那么栈顶就是比他小的元素, 再+1就是能扩展到的地方

由于类似于[1,1]计算的时候会出现重复, 因此从右往左遍历的时候就不需要相等的情况了

## sudo
比较naive的思想就是dfs, 然后比较当前行,当前列, 当前所在box, 很麻烦. 所以不如把行列box都拆开, 判断当前行列box是否包含了某个数字.

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