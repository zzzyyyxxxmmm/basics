# 每日练习

## [leetcode 208 Trie](https://leetcode.com/problems/implement-trie-prefix-tree/)

## [leetcode 399 union find](https://leetcode.com/problems/evaluate-division/)

## [lintcode 892 topo sort](https://www.lintcode.com/problem/alien-dictionary/description)

## [leetcode 84 monotonous stack](https://leetcode.com/problems/largest-rectangle-in-histogram/)

## [LRU Cache] (https://leetcode.com/problems/lru-cache/)

## [leetcode 239. Sliding Window Maximum deque](https://leetcode.com/problems/sliding-window-maximum/)

## low_bound and upper_bound [leetcode 456. 132 Pattern](https://leetcode.com/problems/132-pattern/)
找到第一个大于等于 和 第一个大于

如果找到第一个小于等于的, 可以用upper_bound-1
小于则是low_bound-1
# 基础
# 构造问题
2. 思考如何自己check
3. 列出case, 尝试优化

# 递推
做递推不想递推表达式, 被自己蠢哭

# 数据结构
```c++
priority_queue<int> top是最大的
priority_queue<int, vector<int>, greater<int> > q; top是最小的

```
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

有序的方案数, 无序的方案数, 例如112,121是不同的方案

两个dp都是dp[j]=SUM(DP[i-nums[j]]), 其实就是交换两个loop, 第一个loop i 在外面表示选过的就不能再选了, 第二个loop i在里面, 可以重新再选

# 常见算法

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

## Find the Closest Palindrome
这个的核心就是建立一个set存储可能的结果, Palindrome就是根据前半个的数字建立. 另外对于类似于1000, 需要特殊判断, 不然会丢失999这样的结果

## 判断所有三角形组合
注意双point方向, l和r只能相加

# dp

## 记录路径
1. 边计算边记录, 例如求longestPalindrome
2. 记忆化搜索不一定总能搜到最后, 有可能出现circle, 出现这种情况还是直接搜索, 然后对当前状态建立答案数组, 逐渐优化, 不过这种通常也能用bfs做, 例如: [1284. Minimum Number of Flips to Convert Binary Matrix to Zero Matrix](https://leetcode.com/problems/minimum-number-of-flips-to-convert-binary-matrix-to-zero-matrix/)
3. 状态总数等于: 状态总数-所有不可能的状态及其衍生的状态
# 图论

## 基本术语
1. adjacent vertices. adjacent edges
2. isolated vertex, end vertex
3. cutset. A disconnecting set in a connected graph G is a set of edges whose removal disconnects G. We further define a cutset to be a disconnecting set, no proper subset of which is a disconnecting set. In the above example, only the second disconnecting set is a cutset. Note that the removal of the edges in a cutset always leaves a graph with exactly two components. If a cutset has only one edge e, we call e a **bridge** 
4. We also need the analogous concepts for the removal of vertices. A **separating set** in a connected graph G is a set of vertices whose deletion disconnects G; If a separating set contains only one vertex v, we call v a **cut-vertex**.
5. bfs. In breadth first search, we fan out to as many vertices as possible, before penetrat- ing deeper into the tree. 
6. dfs. In depth first search, we penetrate as deeply as possible into a tree before fanning out to other vertices. 

## 定理
1. Let G be a plane drawing of a connected planar graph, and let n, m andf denote respectively the number of vertices, edges and faces of'G. Then
n-m+f= 2.
2. Let G be a plane graph with n vertices, m edges, ffaces and k components. Then n - m + f = k + 1.

# LCA
[无任何算法的LCA](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree-ii/
## 二分图
染色法判断, KM求最大匹配.

### 例题
[hdu2444](http://acm.hdu.edu.cn/showproblem.php?pid=2444)

## cut vertex and bridge

* 定理: 在无向连通图G的DFS树中, 非根节点u是G的割顶当且仅当u存在一个子结点v, 使得v及其所有后代都没有反向边连回u的祖先(连回u不算).

方便起见, 设low(u)为u及其后代所能连回的最早祖先的pre值, 则定理中的条件就可以简写成u存在一个子结点v, 使得low(v)>=pre(u). 作为一种特殊情况, 如果v的后代只能连回v自己(即low(v)>pre(u)), 只需要删除(u,v)一条边就可以让图G非连通了, 满足这个条件的边称为桥.

### 例题
[leetcode 1192](https://leetcode.com/problems/critical-connections-in-a-network/)

# 几何

### 三点共线

# 模板

# STL


# 我犯傻过的题
1348. Tweet Counts Per Frequency 互相思考

1349. Min Stack 

[1298](https://leetcode.com/contest/weekly-contest-168/problems/maximum-candies-you-can-get-from-boxes/) bfs没想清楚状态

[1201] (https://leetcode.com/problems/ugly-number-iii/) 数学太差

[221] 最求大正方形, 最大矩形

[1424. Diagonal Traverse II](https://leetcode.com/contest/weekly-contest-186/problems/diagonal-traverse-ii/) 这个medium几乎让我变成一个智障, 首先范围没看仔细, 直接按最坏情况想, 结果没看到最底下的最坏情况有一个下限. 然后思考剪枝, 太复杂写了一个小时没写出来, 这种medium就不应该是剪枝, 当时应该直接放弃剪枝, 重新思考的

[1124. Longest Well-Performing Interval](https://leetcode.com/contest/weekly-contest-145/problems/longest-well-performing-interval/) 相对关系

# 绝世好题
[780. Reaching Points](https://leetcode.com/problems/reaching-points/)
[1593. Split a String Into the Max Number of Unique Substrings](https://leetcode.com/contest/weekly-contest-207/problems/split-a-string-into-the-max-number-of-unique-substrings/)
[1594. Maximum Non Negative Product in a Matrix](https://leetcode.com/contest/weekly-contest-207/problems/maximum-non-negative-product-in-a-matrix/)
[1681. Minimum Incompatibility 数位dp](https://leetcode.com/problems/minimum-incompatibility/)
[1692. Count Ways to Distribute Candies Button up 和Top Down区别](https://leetcode.com/problems/count-ways-to-distribute-candies/)


# 待做
[1632. Rank Transform of a Matrix](https://leetcode.com/problems/rank-transform-of-a-matrix/)
[1674. Minimum Moves to Make Array Complementary 数学题, 没意思](https://leetcode.com/problems/minimum-moves-to-make-array-complementary/)