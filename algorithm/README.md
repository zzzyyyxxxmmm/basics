# 每日练习

## [leetcode 208 Trie](https://leetcode.com/problems/implement-trie-prefix-tree/)

## [leetcode 399 union find](https://leetcode.com/problems/evaluate-division/)

## [lintcode 892 topo sort](https://www.lintcode.com/problem/alien-dictionary/description)

## [leetcode 84 monotonous stack](https://leetcode.com/problems/largest-rectangle-in-histogram/)

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