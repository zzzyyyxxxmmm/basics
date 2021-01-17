# Dijstra
```c++
#define INF 0x3f3f3f3
int n;
struct qnode{
    int v;
    int c;
    qnode(int _v=0,int _c=0):v(_v),c(_c){} bool operator <(const qnode &r)const{
        return c>r.c; }
};
struct Edge{
int v,cost;
Edge(int _v=0,int _cost=0):v(_v),cost(_cost){} 
};

void Dijkstra(int n,int start, vector<int> &dist, vector<bool> &vis, vector<vector<Edge>>E){
   
    for(int i=0;i<=n;i++){
        vis[i]=false;
    }
    for(int i=1;i<=n;i++)   dist[i]=INF; 
    priority_queue<qnode>que; 
    while(!que.empty())que.pop();
    dist[start]=0; 
    que.push(qnode(start,0));
    qnode tmp; 
    while(!que.empty()){
        tmp=que.top();
        que.pop();
        int u=tmp.v; 
        if(vis[u]) continue; 
        vis[u]=true;
        for(int i=0;i<E[u].size();i++){
            int v=E[tmp.v][i].v;
            int cost=E[u][i].cost; 
            if(!vis[v]&&dist[v]>dist[u]+cost){
                dist[v]=dist[u]+cost;
                que.push(qnode(v,dist[v]));
            }
        } 
    }
}
void addedge(int u,int v,int w, vector<vector<Edge> >& E){
    E[u].push_back(Edge(v,w));
}

class Solution {
public:
    vector<bool> distanceLimitedPathsExist(int nn, vector<vector<int>>& edgeList, vector<vector<int>>& queries) {
         n=nn;
        vector<vector<Edge> >E(n+2);
        vector<vector<int> > vc(n+1, vector<int>(n+1, INF));
        for(int i=0;i<edgeList.size();i++){
            int u=edgeList[i][0]+1;
            int v=edgeList[i][1]+1;
            int d=edgeList[i][2];
            vc[u][v] = min(vc[u][v], d);
            vc[v][u]=min(vc[v][u], d);
        }

        for(int i=0;i<edgeList.size();i++){
            int u=edgeList[i][0]+1;
            int v=edgeList[i][1]+1;
            addedge(u, v, vc[u][v], E);
            addedge(v, u, vc[v][u], E);
        }

        vector<bool> ans;

        for(int i=0;i<queries.size();i++){
           int u=queries[i][0]+1;
            int v=queries[i][1]+1;
            int d=queries[i][2]; 
            
            vector<int> dist(n+1);
            vector<bool> vis(n+1);
            Dijkstra(n, u, dist,vis,  E); 
            int dis=dist[v];
            cout<<dis<<endl;
            ans.push_back(dis<d);
        }

        return ans;
        
    }
};
```



# BIT
```c++
vector<int> C;
int lowbit(int x){
    return x&(-x);
}
long long get(int x){
    long long ans=0;
    while(x>0){
        ans+=C[x];
        ans%=MOD;
        x-=lowbit(x);
    }
    return ans;
}

void add(int x, int d){
    while(x<=100000){
        C[x]+=d;
        x+=lowbit(x);
    }
}
```
# RMQ
```c++
vector<vector<int> > d;
    int n;
    void RMQ_init(vector<int> &A){
        for(int i=1;i<=A.size();i++){
            d[i][0]=A[i-1];
        }
        for(int j=1;(1<<j)<=n;j++){
            for(int i=1;i+j-1<=n;i++){
                d[i][j]=max(d[i][j-1],d[i+(1<<(j-1))][j-1]);
            }
        }
    }

    int RMQ(int L, int R){
        L++;
        R++;
        int k=0;
        while((1<<(k+1)) <= R-L+1)  k++;
        return max(d[L][k], d[R-(1<<k)+1][k]);
    }
```
# KMP
```c++
void kmp_pre(string x,vector<int> &Next){
    int i,j;
    j=Next[0]=-1;
    i=0;
    while(i<x.length()){
        while(j!=-1 && x[i]!=x[j])  j=Next[j];
        Next[++i]=++j;
    }
}

int kmp_count(string x,string y){
    int i,j;
    int ans=0;
    vector<int> Next(x.length()+1);
    kmp_pre(x,Next);
    i=j=0;
    while(i<y.length()){

        while(j!=-1 && y[i]!=x[j])  j=Next[j];
        i++;j++;
        if(j>=x.length()){
            ans++;
            j=Next[j];
        }
    }
    return ans;
}

```

# 线段树
```
#define lson l,mid,rt<<1
#define rson mid+1,r,rt<<1|1
#define root 1,nums.size(),1
#define mid ((l+r)>>1)
class Solution {
public:
    vector<int> sum;
    vector<int> nums;
    int tot=0;
    void pushup(int rt)
    {
        sum[rt]=sum[rt<<1]+sum[rt<<1|1];
    }

    void build(int l,int r,int rt)
    {
        if(l==r)
        {
            sum[rt]=nums[tot++];
            return;
        }
        build(lson);
        build(rson);
        pushup(rt);
    }

    void Update(int pos,int val,int l,int r,int rt)
    {
        if(l==r)
        {
            sum[rt]=val;
            return;
        }
        if(pos<=mid)    Update(pos,val,lson);
        else Update(pos,val,rson);
        pushup(rt);
    }

    int query(int L,int R,int l,int r,int rt)
    {
        if(l>=L&&r<=R)
        {
            return sum[rt];
        }
        int ans=0;
        if(L<=mid)  ans+=query(L,R,lson);
        if(R>mid)   ans+=query(L,R,rson);
        return ans;
    }

    NumArray(vector<int>& nums) {
        if(nums.size()==0)  return;
        vector<int> s(nums.size()<<4, 0);
        sum=s;
        this->nums=nums;
        build(root);
    }

    void update(int i, int val) {
        Update(i+1, val, root);
    }

    int sumRange(int i, int j) {
        return query(i+1, j+1, root);
    }
};

```
# 图论

## 二分图匹配+最大匹配
```c++
#include <cstdio>
#include <cstdlib>
#include <cmath>
#include <cstring>
#include <iostream>
#include <queue>
#include <algorithm>
#include <vector>
#include <set>
#include<map>
#include<stack>
#include<unordered_map>
#include<ctime>
#include<sstream>

using namespace std;
#define LL __int64
#define INF 0x3f3f3f3f
const int MAXN = 20;
#define mod 1000000007
const double eps = 1e-5;
int dir[4][2] = {-1, 0, 0, 1, 1, 0, 0, -1};
using namespace std;



bool dfs(int u,vector<vector<int> > g , vector<int> &linker, vector<int> used){
    for(int i=0;i<g[u].size();i++){
        int v=g[u][i];
        if(!used[v]){       //被尝试匹配过了,防止循环匹配
            used[v]=1;
            if (linker[v]==-1|| dfs(linker[v],g,linker, used)){
                linker[v]=u;
                return true;
            }
        }
    }
    return false;
}
int km(vector<vector<int> > g){
    int ans=0;
    vector<int> linker(g.size()+1,-1);
    for(int i=0;i<g.size();i++){
        vector<int> used(g.size()+1,0);
        if (dfs(i,g,linker,used)) ans++;
    }
    return ans;
}

bool bicolor(int u, vector<vector<int> > g, vector<int> &color){
    for(int i=0;i<g[u].size();i++){
        int v=g[u][i];
        if (color[u]==color[v]) return false;
        if (!color[v]){
            color[v]=-1*color[u];
            if (!bicolor(v,g,color))  return false;
        }
    }
    return true;
}

int fun(vector<vector<int> > g){
    int n=g.size();
    vector<int> color(n+1, 0);
    for(int i=0;i<n;i++){
        if (!color[i]&&g[i].size()>0){
            color[i]=1;
            if (!bicolor(i,g,color))    return false;
        }
    }
    return km(g)/2;
}

int main() {
    int n,m;
    while(scanf("%d%d",&n,&m)!=EOF){
        vector<vector<int> > G(n+1);
        int u,v;
        for(int i=0;i<m;i++){
            scanf("%d%d",&u,&v);
            u--;
            v--;
            G[u].push_back(v);
            G[v].push_back(u);
        }

        int result=fun(G);
        if (!result){
            cout<<"No"<<endl;
        } else {
            cout<<result<<endl;
        }
    }
    return 0;
}
```

## cut vetex and bridge
```c++
vector<int> pre;
vector<int> low;
vector<int> iscut;
vector<vector<int> > bridge;
int dfs_clock = 0;

int dfs(int u, int parent, vector<vector<int> > &g) {
    int lowu = pre[u] = ++dfs_clock;
    int child = 0;
    for (int i = 0; i < g[u].size(); i++) {
        int v = g[u][i];
        if (!pre[v]) {
            child++;
            int lowv = dfs(v, u, g);
            lowu = min(lowu, lowv);
            if (lowv >= pre[u]) {
                if (lowv > pre[u]) {       //桥
                    bridge.push_back({u, v});
                }
                iscut[u] = true;
            }

        } else if (pre[v] < pre[u] && v != parent) { //利用反向边来更新自己, 注意连回自己不算
            lowu = min(pre[v], lowu);
        }
    }

    if (parent == -1 && child == 1) iscut[u] = false;
    return low[u] = lowu;
}
```