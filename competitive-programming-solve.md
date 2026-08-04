
#### cf template

```
// @author: Tejaansh
// you never fail until you stop trying

#pragma GCC optimize("O3,unroll-loops")
#include <bits/stdc++.h>
using namespace std;

// --- [ types ] ---
using ll  = long long;
using pii = pair<int,int>;
using vi  = vector<int>;

// --- [ macros ] ---
#define fr(i,a,b) for(int i=(a);i<(b);++i)
#define all(x)    (x).begin(),(x).end()
#define sz(x)     (int)(x).size()
#define pb        push_back
#define F         first
#define S         second

// --- [ constants ] ---
constexpr int INF  = 1e9;
constexpr ll  LINF = 4e18;
constexpr int MOD  = 1e9+7;

// --- [ rng ] ---
mt19937_64 rng(chrono::steady_clock::now().time_since_epoch().count());
auto rnd = [](ll l, ll r){ return uniform_int_distribution<ll>(l,r)(rng); };

// --- [ debug ] ---
#ifdef LOCAL
#define dbg(x) cerr<<" \033[33m"<<#x<<"\033[0m = "; _dbg(x); cerr<<"\n";
#define dbg2(x,y) dbg(x); dbg(y)
#else
#define dbg(x)
#define dbg2(x,y)
#endif

template<class T>          void _dbg(T x)                  { cerr<<x; }
void _dbg(__int128 x)                                       { if(x<0){cerr<<'-';x=-x;} if(x>9)_dbg(x/10); cerr<<(char)('0'+x%10); }
template<class A,class B>  void _dbg(const pair<A,B>& p)   { cerr<<"("<<p.F<<","<<p.S<<")"; }
template<class T>          void _dbg(const vector<T>& v)   { cerr<<"[ "; for(const auto&x:v){_dbg(x);cerr<<" ";} cerr<<"]"; }
template<class T>          void _dbg(const set<T>& s)      { cerr<<"{ "; for(const auto&x:s){_dbg(x);cerr<<" ";} cerr<<"}"; }
template<class T>          void _dbg(const multiset<T>& s) { cerr<<"{ "; for(const auto&x:s){_dbg(x);cerr<<" ";} cerr<<"}"; }
template<class K,class V>  void _dbg(const map<K,V>& m)    { cerr<<"{ "; for(const auto&[k,v]:m){_dbg(k);cerr<<":";_dbg(v);cerr<<" ";} cerr<<"}"; }

// --- [ math ] ---
ll pw(ll b, ll e, ll m=MOD) {
    ll r=1; b%=m;
    for(;e>0;e>>=1){ if(e&1) r=r*b%m; b=b*b%m; }
    return r;
}
ll inv(ll a, ll m=MOD) { return pw(a,m-2,m); }

// --- [ dsu ] ---
struct DSU {
    vi p, size_arr; int comps;
    DSU(int n): p(n), size_arr(n, 1), comps(n) { iota(all(p), 0); }
    int find(int x) { while(p[x] != x) { p[x] = p[p[x]]; x = p[x]; } return x; }
    bool same(int a, int b) { return find(a) == find(b); }
    int size(int x) { return size_arr[find(x)]; }
    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;
        if (size_arr[a] < size_arr[b]) swap(a, b);
        p[b] = a; size_arr[a] += size_arr[b];
        comps--; return true;
    }
};

// --- [ bit / fenwick (1-indexed) ] ---
struct BIT {
    int n; vector<ll> t;
    BIT(int n): n(n),t(n+1,0) {}
    void upd(int i, ll v) { for(;i<=n;i+=i&-i) t[i]+=v; }
    ll qry(int i) { ll s=0; for(;i>0;i-=i&-i) s+=t[i]; return s; }
    ll qry(int l, int r) { return qry(r)-qry(l-1); }
};

// --- [ compress ] ---
// auto c=compress(v); int idx=lower_bound(all(c),x)-c.begin();
template<class T>
vector<T> compress(vector<T> v) {
    sort(all(v)); v.erase(unique(all(v)),v.end()); return v;
}

// --- [ graph helpers ] ---
// weighted: vector<vector<pair<int,ll>>> g(n);
// unweighted: vector<vector<int>> g(n);
// edge: g[u].pb({v,w});

// --- [ solve ] ---
void solve() {
    
}

// --- [ main ] ---
int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr); // precompute();

    int t=1; cin>>t;
    while(t--) solve();
}
```

### Dynamic Programming:

Dynamic Programming is mostly just recursion with a few optimizations. 
- don't do the same thing multiple times
- discard useless info

Algebraically, `dp[i][j]` is simply a mathematical function $f(i, j)$ that returns a single numerical value (such as a count, minimum cost, or maximum profit) based on state variables $i$ and $j$.

Think of `dp[i][j]` as a **lookup table** for a subproblem:
* **`i` , `j` (The State):** Inputs defining  the subproblem we solve to build up to the main problem (e.g., $i$ = index of an array, $j$ = capacity). States represent the value(s) that the discarded info compress to. 
* **`dp[i][j]` (The Value):** Solution/optimal value for that specific subproblem.
- Transitions: represent the way the states interact with each other
- Base Case: similar to recursion

Push VS Pull dynamic programming: 

Pull DP: pull answers from prev DP values (our recursive solution)
Recursive DP can only be Pull DP; for iterative Pull DP the answers we pull need to be complete and the order of iteration always matters
```
for each state x:
	for each state y that can effect x
	    do some transition with dp[y] to dp[x]
```
Push DP: push answers to future DP values (our iterative solution)
```
for each state y
	for each state x that it effects
		do some transition with dp[y] to dp[x]
```
##### Common Algebraic Operations

A. Addition
* Meaning: The total number of ways to reach the current state is the sum of ways to reach state $A$ plus the sum of ways to reach state $B$.
* When to use: Counting problems or combining independent choices.
* Example:
  ```cpp
  dp[i][j] = dp[i-1][j] + dp[i][j-1]
  // Ways to reach cell (i, j) = (ways from top cell) + (ways from left cell)
  ```

B. Multiplication
* Meaning: If event $A$ happens in $X$ ways and event $B$ happens in $Y$ ways, both happen in $X \times Y$ ways.
* When to use: Combinatorics, independent sequential choices, or probability DP.
* Example:
  ```cpp
  dp[u][state] = dp[v1][state] * dp[v2][state]
  // Tree DP or combining independent combinatorial subtrees
  ```

C. Min / Max
* Meaning: Out of multiple valid decisions, select the optimal one.
* When to use: Optimization problems (maximizing profit, minimizing cost).
* Example:
  ```cpp
  dp[i][j] = max(dp[i-1][j], dp[i-1][j - w[i]] + v[i])
  // 0/1 Knapsack: max(exclude item i, include item i)
  ```

D. Division
* Meaning: Expected value or modular inverse calculations.
* When to use: Averaging outcomes (probability DP) or dividing out identical permutations (combinatorics).

> [!tip] Golden Rule of DP Transitions
> You can combine any states (`dp[i][j]`, `dp[k][m]` and so on) as long as:
> 1. There are no cycles (the underlying state graph is a **DAG**).
> 2. You satisfy subproblem independence.

- **`vector<int> arr(5):`** Creates 1 vector holding 5 plain integers (1D).
- **`vector<int> arr[5]:`** Creates a fixed-size C-array of 5 vectors stored on the stack (2D).
- **`vector<vector<int>> arr(5):`** Creates 1 dynamic vector holding 5 vectors stored on the heap (2D).


_Memoization_ is a computer optimization technique that stores the results of expensive function calls; The property where many different paths collapse into one reusable state is called _overlapping subproblems_ and it's the entire reason DP is faster than brute force. 


> [!tip] How to recognize and solve DP problems?
> intuition, start off by assuming every problem is DP
> 
> start by thinking how you would solve it recursively, like a brute force or backtracking
> Recursive + Memoization will work for most DP problems
> 
> break the problem into bits and solve one bit at a time; the recursive function can usually directly tell you the DP state and transitions
> 
> if recursion doesn't work, try to pull the state / transitions right out of the statement
> if neither of these tricks work, it's probably not an easy problem