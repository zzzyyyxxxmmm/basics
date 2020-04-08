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