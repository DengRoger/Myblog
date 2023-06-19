---
title: CP Template
date: 2023-06-19 09:08:52
Categories:
    - CP
tags:
    - C++
    - CP
---

# CP Template
<!-- more -->

- [IO , PBDS](#IO,PBDS)
- [KMP](#KMP)
- [DAG](#DAG)
- [segmentTree](#SegmentTree)
- [ConvexHull](#ConvexHull)
- [MinSwaps](#MinSwaps)
- [Eratosthenes](#Eratosthenes)
- [3DLCS](#3DLCS)
- [EditDistance](#EditDistance)
- [LCS](#LCS)
- [LIS](#LIS)
- [Dijkstra](#Dijkstra)
- [*A](#*A)
- [Maximum s-t Flow](#MaximumS-tFlow)
- [BCC](#BCC)
- [LCA](#LCA)
- [Tarjan](#Tarjan)
- [BIT](#BIT)
- [擴展歐幾里德](#擴展歐幾里德)
- [Bellman Ford](#BellmanFord)
- [Hash](#hash)
- [ODT](#ODT) (又名科朵莉樹)
- [李超線段數](#李超線段數) 
- [AC自動機](#AC自動機)


## 

        - 重構大綱
            -  LCS , LIS , EditDistance , Dijkstra , bellman ford 、 KMP 
                等算法屬於簡單DP不該存於模板內
            - 基礎IO背起來
            - minSwaps O(n) 版本基本上完全用不到 
            - RMQ有莫隊算法的版本 
            - ODT、李超、AC automata、*A 都用不太到
            - segmentTree(包括 lazy tag 和 persistent) 、 Hash 、 Eratosthenes 
                等級的code需要立即寫出來 也不該存於模板中
            - BCC LCA 都可以用 Tarjan 改
            - Maximum Flow 準備一個 Dinic 即可
            - 學一下 NIM 
            - 3D LCS的時間複雜度須為 O(mn)
            - 新增條目 LCIS 

## IO,PBDS 
```cpp=
#include <bits/stdc++.h>
#pragma GCC optimize("O3","unroll-loops")
#define IO cin.tie(0), ios::sync_with_stdio(0)
#define All(x) x.begin(), x.end()
#define sz(x) (int)(x).size()
#define sort_unique(x) sort(All(x)); x.erase(unique(All(x)), x.end());
#define eb emplace_back 
#define pb push_back   
#define ne nth_element  
#define lb lower_bound  
#define ub upper_bound  
using namespace std;
typedef long long ll;
typedef pair<ll, ll> pii;
typedef vector<ll>   vl;

// #include <ext/pb_ds/assoc_container.hpp>
// #include <ext/pb_ds/tree_policy.hpp>
// using namespace __gnu_pbds;
// #define multiset tree<ll, null_type,less_equal<ll>, rb_tree_tag,tree_order_statistics_node_update>
/**
order_of_key(k) : nums strictly smaller than k
find_by_order(k): index from 0
**/
const int N = 50+5;
const ll INF = 1e9;
const int mod = 1e9+2e5+7;
const ll MXP = 1e10;
const int dir[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
```

## KMP 
```cpp=
bool match(string &s, string &t, vector<int> &F) {
    for (int i = 0, pos = -1 ; i < s.size() ; i++) {
        while (~pos && s[i] != t[pos + 1]) pos = F[pos];
        if (s[i] == t[pos + 1]) pos++;
        if (pos + 1 == (int)t.size()) return true;
    }
    return false;
}
```

## DAG
```cpp=
void DFS(map<int,queue<int>> &graph , map<int,bool> &visited , int place , stack<int> &ans){ 
    if(visited[place] == true) return; 
    visited[place]=true;
    while(!graph[place].empty())
        DFS(graph, visited, graph[place].front() , ans) , graph[place].pop() ; 
    ans.push(place);
}
for(auto&n:graph) DFS(graph , visited , n.first , ans) ;
while(!ans.empty()) cout << ans.top() << " " , ans.pop();
```

## SegmentTree
```cpp=
ll v[1e5] , seg[4e5+7] ; 
void StructSEG(int l , int r , ll node){
    if(l==r) seg[node] = v[r] ; 
    else{
        int mid = (l+r) >> 1 , tmp = node << 1;
        StructSEG(l,mid,tmp) ; 
        StructSEG(mid+1,r,tmp+1) ;
        seg[node] = seg[tmp] + seg[tmp+1] ; 
    }
}
int query(int tL , int tR , int nL , int nR , int node){
    int mid = (nL+nR)>>1 , ans = 0 ; 
    if(tL <= nL && nR <= tR) return seg[node] ;
    if(tL <= mid) ans += query(tL,tR,nL   ,mid,(node<<1)  );
    if(tR  > mid) ans += query(tL,tR,mid+1,nR ,(node<<1)+1);  
    return ans ; 
}
void updateNode(int idx , int val , int l , int r , int node){
    if(l == r) seg[node] = val , v[idx] = val ;
    else{
        int m = (l + r) >> 1 ; 
        int leftNode  = (node << 1) ; 
        int rightNode = (node << 1) + 1 ; 
        if(idx <= m && idx >= l) updateNode(idx , val , l , m , leftNode ) ;
        else updateNode(idx , val , m+1,r , rightNode ) ; 
        seg[node] = seg[leftNode] + seg[rightNode] ; 
    } 
}
```

## SegmentTree (lazy tag)
```cpp=
#define Ls(x) x << 1
#define Rs(x) x << 1 | 1

int N = 1e5+5;
vector<int> segTree(4*N), lazy(4*N) , arr(N);
// segment tree with lazy propagation 

void build(int l, int r, int idx = 1){
    if(l == r){
        segTree[idx] = arr[l];
        return;
    }
    int mid = (l+r) >> 1;
    build(l, mid, Ls(idx));
    build(mid+1, r, Rs(idx));
    segTree[idx] = segTree[Ls(idx)]+ segTree[Rs(idx)];
}

void pushDown(int l, int r, int idx){
    if(lazy[idx] == 0) return;
    // adding lazy[idx] to each element in range [l, r]
    segTree[idx] += (r-l+1)*lazy[idx]; 
    // set lazy for children
    if(l != r) lazy[Ls(idx)] += lazy[idx],lazy[Rs(idx)] += lazy[idx];
    // clear lazy[idx]
    lazy[idx] = 0; 
}

void update(int l, int r, int ql, int qr, int val, int idx = 1){
    // l : left index of current range
    // r : right index of current range
    // ql : left index of query range
    // qr : right index of query range
    // val : value to be added to each element in range [ql, qr]
    // idx : current index of segment tree
    pushDown(l, r, idx); // push down lazy value to children
    if(l > qr || r < ql) return; // out of range
    // completely in range case
    // if current range is not completely in range, we need to update its children
    if(l >= ql && r <= qr){
        segTree[idx] += (r-l+1)*val;
        if(l != r) lazy[Ls(idx)] += val, lazy[Rs(idx)] += val;
        return;
    }
    int mid = (l+r) >> 1;
    update(l, mid, ql, qr, val, Ls(idx)); // update left child
    update(mid+1, r, ql, qr, val, Rs(idx)); // update right child
    segTree[idx] = segTree[Ls(idx)]+ segTree[Rs(idx)];
}

int query(int l, int r, int ql, int qr, int idx = 1){
    pushDown(l, r, idx); // push down lazy value to children
    if(l > qr || r < ql) return 0; // out of range 
    if(l >= ql && r <= qr) return segTree[idx]; // completely in range
    int mid = (l+r) >> 1, sum = 0; 
    if(ql <= mid) sum += query(l, mid, ql, qr, Ls(idx));
    if(qr > mid) sum += query(mid+1, r, ql, qr, Rs(idx)); 
    return sum;
}
```

## ConvexHull
```cpp=
void Dhull(vector<pii> points , vector<pii> &hull){
    hull.push_back(points[0]) , hull.push_back(points[1]);
    for(int i = 2 ; i < points.size() ; i++){
        while(hull.size() >= 2){
            pair<int,int> p1 = hull[hull.size()-2] , p2 = hull[hull.size()-1] , p3 = points[i];
            int x1 = p2.first - p1.first , y1 = p2.second - p1.second;
            int x2 = p3.first - p2.first , y2 = p3.second - p2.second;
            if(x1*y2 - x2*y1 <= 0) break;
            hull.pop_back();
        }
        hull.push_back(points[i]);
    }
}
int main(){
    IO ; 
    vector<pair<int,int>> points,hull; 
    int n , x , y;
    cin >> n ;  
    for(int i = 0 ; i < n ; i++) cin>>x>>y , points.pb({x,y}); 
    sort(points.begin() , points.end());
    Dhull(points,hull) ; 
    reverse(points.begin(),points.end());
    Dhull(points,hull) ; 
    for(auto p : hull) cout << p.first << " " << p.second << endl;
}
```

## MinSwaps
```cpp=
int minSwaps(vector<int> &arr, int n) { 
    pii arrPos[n]; 
    for (int i = 0; i < n; i++) { 
        arrPos[i].first = arr[i]; 
        arrPos[i].second = i; 
    }
    sort(arrPos, arrPos + n); 
    vector<bool> vis(n, false); 
    int ans = 0; 
    for (int i = 0; i < n; i++) { 
        if (vis[i] or arrPos[i].second == i) continue; 
        int cycle_size = 0 , j = i ; 
        while (!vis[j]){ 
            vis[j] = 1; 
            j = arrPos[j].second; 
            cycle_size++; 
        } 
        if(cycle_size > 0) ans += (cycle_size - 1); 
    } 
    return ans; 
} 
```

## Eratosthenes
```cpp=
auto eratosthenes(int upperbound) {
    vector<bool> flag(upperbound + 1, true);
    flag[0] = flag[1] = false;
    for (int i = 2; i * i <= upperbound; ++i) {
        if (flag[i]) {
            for (int j = i * i; j <= upperbound; j += i)
                flag[j] = false;
        }
    }	
    return flag;
}
```

## 3DLCS 
(此條目需更新)
```cpp=
#include <bits/stdc++.h>
using namespace std;

int main(){
    string x,y,z ; cin >> x >> y >> z ; 
    cout << lcsOf3(x,y,z , x.size() , y.size() , z.size()) << endl;
}

int lcsOf3( string X, string Y, string Z, int m, int n, int o){
    int L[m+1][n+1][o+1];
    for (int i=0; i<=m; i++){
        for (int j=0; j<=n; j++){
            for (int k=0; k<=o; k++){
                if (i == 0 || j == 0||k==0) L[i][j][k] = 0;
                else if (X[i-1] == Y[j-1] && X[i-1]==Z[k-1]) L[i][j][k] = L[i-1][j-1][k-1] + 1;
                else L[i][j][k] = max(max(L[i-1][j][k],L[i][j-1][k]),L[i][j][k-1]);
            }
        }
    }
    return L[m][n][o];
}
```
## EditDistance 
```cpp=
int EditDistance2(string s1 , string s2){ // space complexity : O(n)
    vector<int> EditDistance(s2.size()+1, 0);
    for(int i = 0 ; i <= s2.size() ; i++) EditDistance[i] = i;
    for(int i = 1 ; i <= s1.size() ; i++){
        int prev = EditDistance[0];
        EditDistance[0] = i;
        for(int j = 1 ; j <= s2.size() ; j++){
            int temp = EditDistance[j];
            if(s1[i-1] == s2[j-1]) EditDistance[j] = prev;
            else EditDistance[j] = min(prev , min(EditDistance[j] , EditDistance[j-1])) + 1;
            prev = temp;
        }
    }
    return EditDistance[s2.size()];
}
```

## LCS 

## LIS 

## Dijkstra 

## *A 

## MaximumS-tFlow

## BCC

## LCA

## Tarjan