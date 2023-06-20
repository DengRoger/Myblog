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

## compile
```bash=
#!/bin/bash
g++ $1 && ./a.out < ipt.in > opt.out
```

## IO,PBDS 
```cpp=
#include <bits/stdc++.h>
#pragma GCC optimize("O3","unroll-loops")
#define IO cin.tie(0), ios::sync_with_stdio(0)
#define All(x) x.begin(), x.end()
#define sort_unique(x) sort(All(x)); x.erase(unique(All(x)), x.end());
#define ne nth_element  
using namespace std;
#include <bits/extc++.h>
#include <ext/pb_ds/assoc_container.hpp>
#include <ext/pb_ds/tree_policy.hpp>
using namespace __gnu_pbds;
#define multiset tree<ll, null_type,less_equal<ll>, rb_tree_tag,tree_order_statistics_node_update>
order_of_key(k) : nums strictly smaller than k
find_by_order(k): index from 0
/*
| Tag                  | push  | pop    | join | modify |
| paring_heap_tag      | O(1)  | O(lgN) | O(1) | O(lgN) |
| thin_heap_tag        | O(lgN)| O(lgN) |  慢  |   慢    |
| binomial_heap_tag    | O(1)  | O(lgN) | O(1) | O(lgN) |
| rc_binomial_heap_tag | O(1)  | O(lgN) | O(1) | O(lgN) |
| binary_heap_tag      | O(1)  | O(lgN) |  慢  | O(lgN) | 
*/
typedef __gnu_pbds::priority_queue<pair<int,ii>,less<pair<int,ii>>,rc_binomial_heap_tag> heap;
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

## SegmentTreeLazy
```cpp=
#define Ls(x) x << 1
#define Rs(x) x << 1 | 1
int N = 1e5+5;
vector<int> segTree(4*N), lazy(4*N) , arr(N);
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
    segTree[idx] += (r-l+1)*lazy[idx]; 
    if(l != r) lazy[Ls(idx)] += lazy[idx],lazy[Rs(idx)] += lazy[idx];
    lazy[idx] = 0; 
}

void update(int l, int r, int ql, int qr, int val, int idx = 1){
    pushDown(l, r, idx); 
    if(l > qr || r < ql) return; 
    if(l >= ql && r <= qr){
        segTree[idx] += (r-l+1)*val;
        if(l != r) lazy[Ls(idx)] += val, lazy[Rs(idx)] += val;
        return;
    }
    int mid = (l+r) >> 1;
    update(l, mid, ql, qr, val, Ls(idx));
    update(mid+1, r, ql, qr, val, Rs(idx));
    segTree[idx] = segTree[Ls(idx)]+ segTree[Rs(idx)];
}

int query(int l, int r, int ql, int qr, int idx = 1){
    pushDown(l, r, idx); 
    if(l > qr || r < ql) return 0;
    if(l >= ql && r <= qr) return segTree[idx]; 
    int mid = (l+r) >> 1, sum = 0; 
    if(ql <= mid) sum += query(l, mid, ql, qr, Ls(idx));
    if(qr > mid) sum += query(mid+1, r, ql, qr, Rs(idx)); 
    return sum;
}
```

## ConvexHull *** 
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

## 3DLCS *** (有加速版本)
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
// 問從 src 到 dst 的最小 edit distance
// ins 插入一個字元的成本
// del 刪除一個字元的成本
// sst 替換一個字元的成本
ll edd(string& src, string& dst, ll ins, ll del, ll sst) {
    ll dp[src.size() + 1][dst.size() + 1]; // 不 用 初 始 化
    for (int i = 0; i <= src.size(); i++) {
        for (int j = 0; j <= dst.size(); j++) {
            if (i == 0) dp[i][j] = ins * j;
            else if (j == 0) dp[i][j] = del * i;
            else if (src[i - 1] == dst[j - 1])
                dp[i][j] = dp[i - 1][j - 1];
            else
                dp[i][j] = min(dp[i][j - 1] + ins, 
                               min(dp[i - 1][j] + del, 
                                   dp[i - 1][j - 1] + sst));
        }
    }
    return dp[src.size()][dst.size()];
}
```

## dinc LCS
```cpp=
若一樣：左上+1
else max(左,上)
```

## LPS
```cpp=
int lps(string s){
  int N=2*s.size()+1;
  vector<int> dp(N);
  string s2="*";
  for(auto&c:s){
    s2.push_back(c);
    s2.push_back('*');
  }
  int C=0,R=0;
  for(int i = 1;i < N;i++){
    if(i>R)C=R=i;
    else{
      int mirrorI=C-(i-C);
      dp[i]=min(dp[mirrorI],R-i);
    }
    int j=dp[i]+1;
    while((i-j>=0)&&(i+j<N)&&(s2[i-j]==s2[i+j]))j++;
    dp[i]=j-1;
    if(i+dp[i]>R){
      C=i;
      R=i+dp[i];
    }
  }
  return *max_element(dp.begin(),dp.end());
}

string lps(string s){
  int N=2*s.size()+1;
  vector<int> dp(N);
  string s2="*";
  for(auto&c:s){
    s2.push_back(c);
    s2.push_back('*');
  }
  int C=0,R=0;
  for(int i = 1;i < N;i++){
    if(i>R)C=R=i;
    else{
      int mirrorI=C-(i-C);
      dp[i]=min(dp[mirrorI],R-i);
    }
    int j=dp[i]+1;
    while((i-j>=0)&&(i+j<N)&&(s2[i-j]==s2[i+j]))j++;
    dp[i]=j-1;
    if(i+dp[i]>R){
      C=i;
      R=i+dp[i];
    }
  }
  auto it=max_element(dp.begin(),dp.end());
  int maxLen=*it;
  int index=it-dp.begin();
  return s.substr((index-maxLen)/2,maxLen);
}
```

## LIS
```cpp=
void LIS(vector<int> &arr){
    vector<int> lis;
    for(int i = 0; i < arr.size(); i++){
        auto it = lower_bound(lis.begin(), lis.end(), arr[i]);
        if(it == lis.end()) lis.push_back(arr[i]);
        else *it = arr[i];
    }
    cout << lis.size() << endl;
}
```

## LCS
```cpp=
// LCS with O(nlogn) time complexity
int LCS(string text1, string text2) {
    // LCS -> LIS
    vector<int> alph[128];  // record text1's alphabet in text2 pos.
    int maps[128];
    memset(maps, 0, sizeof(maps));
    for(int i = 0; i < text1.size(); i++) maps[text1[i]] = 1;
    for(int j = text2.size(); j > -1; j--)
        if(maps[text2[j]] == 1) alph[text2[j]].push_back(j);
    vector<int> nums;
    for(int i = 0; i < text1.size(); i++) {
        if(alph[text1[i]].size() > 0)
            nums.insert(nums.end(), alph[text1[i]].begin(), alph[text1[i]].end());
    }
    // get LIS's length by monotone stack method : O(nlogn)
    vector<int> pool;
    for(int i = 0; i < nums.size(); i++) {
        if(i == 0 || nums[i] > pool.back() ) {
            pool.push_back(nums[i]);
        } else if(nums[i] == pool.back()) {
            continue;
        } else {
            int s = 0, e = pool.size() - 1, mid = 0;
            while(s < e) {
                mid = (s + e)/2;
                if(pool[mid] < nums[i]) s = mid + 1;
                else e = mid;
            }
            pool[e] = nums[i];
        }
    }
    return pool.size();
}
```
## Kurskal
```cpp=
struct Edge {
    int src;
    int dest;
    int weight; 
    Edge(int s, int d, int w) : src(s), dest(d), weight(w) {}
};
int findParent(const vector<int>& parent, int i) {
    if (parent[i] == i) return i;
    return findParent(parent, parent[i]);
}
vector<Edge> kruskalMST(const vector<vector<int>>& adjacencyMatrix) {
    int size = adjacencyMatrix.size();
    vector<Edge> mst, edges;
    for (int i = 0; i < size; ++i) {
        for (int j = i + 1; j < size; ++j) {
            if (adjacencyMatrix[i][j] > 0) edges.push_back(Edge(i, j, adjacencyMatrix[i][j]));
        }
    }
    sort(edges.begin(), edges.end(), [](const Edge& e1, const Edge& e2) {
        return e1.weight < e2.weight;
    });
    vector<int> parent(size);
    for (int i = 0; i < size; ++i) parent[i] = i;
    int count = 0; 
    for (const Edge& edge : edges) {
        if (count == size - 1) break;
        int srcParent = findParent(parent, edge.src) , destParent = findParent(parent, edge.dest);
        if (srcParent != destParent) {
            mst.push_back(edge);
            ++count;
            parent[srcParent] = destParent;
        }
    }
    return mst;
}
```

## BooleanForward
```cpp=
bool booleanForward(string s, vector<string> &dict) {
    int n = s.size();
    vector<bool> dp(n + 1, false);
    dp[0] = true;
    for (int i = 1; i <= n; i++) {
        for (auto word : dict) {
            int len = word.size();
            if (i >= len && s.substr(i - len, len) == word) {
                dp[i] = dp[i] || dp[i - len];
            }
        }
    }
    return dp[n];
}
```

## SPFA 
https://www.youtube.com/watch?v=Hy6IaVFFeqI
https://hackmd.io/@andy010629/Bellman-Ford


## qpw
```cpp=
ll qpw(ll base, ll exponent) {
    ll result = 1;
    while (exponent != 0) {
        if (exponent % 2 == 1)result *= base;
        base *= base;
        exponent /= 2;
    }
    return result;
}

long long POW(long long a, long long b) { 
    if (b == 0) return 1; 
    long long k = POW(a,b/2); 
    if (b % 2)return k * k * a; 
    return k * k;
}
```

## 轉換
```cpp=
stoi
stoll
to_string
```

## AC自動機
```cpp=
struct _AC{
	_AC *child[26];
	_AC *Fail;
	vector<pair<int,int>> out;
	_AC(){
		Fail = NULL;
		memset(child,0,sizeof(child));
	}
}*root;
void Insert_AC(string s){
	int n;
	_AC *p = root;
	for(auto&k:s){
		n = k - 'a';
		if(!p->child[n]) p->child[n] = new _AC();
		p = p->child[n];
	}
	p->out.push_back({s.size(),0});
}
void Construct_AC(){
	queue<_AC*> Q;
	for(int k=0;k<26;k++){
		if(root->child[k]){
			root->child[k]->Fail = root;
			Q.push(root->child[k]);
		}
		else root->child[k] = root;
	}
	_AC *p;
	while(!Q.empty()){
		p = Q.front();
		Q.pop();
		for(int k=0;k<26;k++){
			if(!p->child[k])p->child[k] = p->Fail->child[k];
			else {
				p->child[k]->Fail = p->Fail->child[k];
				for(auto&i:p->Fail->child[k]->out)p->child[k]->out.push_back({i.first,1});
				Q.push(p->child[k]);
			}
		}
	}
}
void Match_AC(string t){
	int n;
	_AC *p = root;
	for(int k=0;k<t.size();k++){
		n = t[k] - 'a';
		p = p->child[n];
		_AC *fail = p;
		while(fail != root && fail->out.size()){
			for(auto&i:fail->out)if(!i.second||fail->Fail->out.size())cout<<t.substr(k-i.first+1,i.first)<<'\n';
			fail->out.clear();
			fail->Fail->out.clear();
			fail = fail->Fail;
		}
	}
}

int main(){
	int n,m;string p,t;cin>>n;
	while(cin>>n>>m){
		root = new _AC();
		while(n--){
			cin>>p;
			Insert_AC(p);
		}
		Construct_AC();
		cin>>n>>m;
		cin>>t;
		Match_AC(t);
	}
}
```

## 離散化
```cpp=
vector<int> v(1000),b(1000);
for(auto&n:v)cin>>n;
sort(v.begin(),v.end());
auto len = unique(v.begin(),v.end())-v.begin();
v.resize(len);
for(int i = 0; i < v.size(); i++){
    b[i] = lower_bound(v.begin(),v.end(),v[i])-v.begin();
}
```

## 鞋帶
```cpp=
vector<pair<int,int>> v(n);
for(auto&n:v) cin >> n.first >> n.second;
int area = 0;
for(int i = 0; i < v.size(); i++){
    area += v[i].first * v[(i+1)%v.size()].second;
    area -= v[i].second * v[(i+1)%v.size()].first;
}
cout << abs(area)/2 << endl;
```

## 最小覆蓋圓 

## 最近點對

## else 
``` txt 
https://blog.csdn.net/weixin_42887391/article/details/84638883
https://peienwu.com/pair/
最近點對 

```