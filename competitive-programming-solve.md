
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

	 