## 图论

### 链式前向星

```cpp
int ecnt, mp[MAXN];

struct Edge {
    int to, nxt;
    Edge(int to = 0, int nxt = 0) : to(to), nxt(nxt) {}
} es[MAXM];

void mp_init() {
    memset(mp, -1, (n + 2) * sizeof(int));
    ecnt = 0;
}

void mp_link(int u, int v) {
    es[ecnt] = Edge(v, mp[u]);
    mp[u] = ecnt++;
}

for (int i = mp[u]; i != -1; i = es[i].nxt)
```

### 最短路

+ Dijkstra

```cpp
struct Edge {
    int to, val;
    Edge(int to = 0, int val = 0) : to(to), val(val) {}
};

vector<Edge> G[MAXN];
ll dis[MAXN];

void dijkstra(int s) {
    using pii = pair<ll, int>;
    memset(dis, 0x3f, sizeof(dis));
    priority_queue<pii, vector<pii>, greater<pii> > q;
    dis[s] = 0;
    q.emplace(0, s);
    while (!q.empty()) {
        pii p = q.top();
        q.pop();
        int u = p.second;
        if (dis[u] < p.first) continue;
        for (Edge& e : G[u]) {
            int v = e.to;
            if (updmin(dis[v], dis[u] + e.val)) {
                q.emplace(dis[v], v);
            }
        }
    }
}
```

+ SPFA

```cpp
void spfa(int s) {
    queue<int> q;
    q.push(s);
    memset(dis, 0x3f, sizeof(dis));
    memset(in, 0, sizeof(in));
    in[s] = 1;
    dis[s] = 0;
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        for (Edge& e : G[u]) {
            int v = e.to;
            if (dis[v] > dis[u] + e.val) {
                dis[v] = dis[u] + e.val;
                if (!in[v]) {
                    in[v] = 1;
                    q.push(v);
                }
            }
        }
        in[u] = 0;
    }
}
```

+ Floyd 最小环

```cpp
// 注意 INF 不能超过 1/3 LLONG_MAX
for (int k = 0; k < n; k++) {
    for (int i = 0; i < k; i++) {
        for (int j = 0; j < i; j++) {
            ans = min(ans, G[i][k] + G[k][j] + dis[i][j]);
        }
    }
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            dis[i][j] = min(dis[i][j], dis[i][k] + dis[k][j]);
        }
    }
}
```

### 拓扑排序

```cpp
int n, deg[MAXN], dis[MAXN];
vector<int> G[MAXN];

bool topo(vector<int>& ans) {
    queue<int> q;
    for (int i = 1; i <= n; i++) {
        if (deg[i] == 0) {
            q.push(i);
            dis[i] = 1;
        }
    }
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        ans.push_back(u);
        for (int v : G[u]) {
            deg[v]--;
            dis[v] = max(dis[v], dis[u] + 1);
            if (deg[v] == 0) q.push(v);
        }
    }
    return ans.size() == n;
}
```

### 最小生成树

```cpp
// 前置：并查集
struct Edge {
    int from, to, val;
    Edge(int from = 0, int to = 0, int val = 0) : from(from), to(to), val(val) {}
};

vector<Edge> es;

ll kruskal() {
    sort(es.begin(), es.end(), [](Edge& x, Edge& y) { return x.val < y.val; });
    iota(pa, pa + n + 1, 0);
    ll ans = 0;
    for (Edge& e : es) {
        if (find(e.from) != find(e.to)) {
            merge(e.from, e.to);
            ans += e.val;
        }
    }
    return ans;
}
```

### LCA

```cpp
int dep[MAXN], up[MAXN][22]; // 22 = ((int)log2(MAXN) + 1)

void dfs(int u, int pa) {
    dep[u] = dep[pa] + 1;
    up[u][0] = pa;
    for (int i = 1; i < 22; i++) {
        up[u][i] = up[up[u][i - 1]][i - 1];
    }
    for (int i = 0; i < G[u].size(); i++) {
        if (G[u][i] != pa) {
            dfs(G[u][i], u);
        }
    }
}

int lca(int u, int v) {
    if (dep[u] > dep[v]) swap(u, v);
    int t = dep[v] - dep[u];
    for (int i = 0; i < 22; i++) {
        if ((t >> i) & 1) v = up[v][i];
    }
    if (u == v) return u;
    for (int i = 21; i >= 0; i--) {
        if (up[u][i] != up[v][i]) {
            u = up[u][i];
            v = up[v][i];
        }
    }
    return up[u][0];
}
```

### 网络流

+ 最大流

```cpp
const int INF = 0x7fffffff;

struct DEdge {
    int to, cap;
    DEdge(int to, int cap) : to(to), cap(cap) {}
};

struct Dinic {
    int n, s, t;
    vector<DEdge> es;
    vector<vector<int> > G;
    vector<int> dis, cur;

    Dinic(int n, int s, int t) : n(n), s(s), t(t), G(n + 1), dis(n + 1), cur(n + 1) {}

    void add_edge(int u, int v, int cap) {
        G[u].push_back(es.size());
        es.emplace_back(v, cap);
        G[v].push_back(es.size());
        es.emplace_back(u, 0);
    }

    bool bfs() {
        dis.assign(n + 1, 0);
        queue<int> q;
        q.push(s);
        dis[s] = 1;
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            for (int i : G[u]) {
                DEdge& e = es[i];
                if (!dis[e.to] && e.cap > 0) {
                    dis[e.to] = dis[u] + 1;
                    q.push(e.to);
                }
            }
        }
        return dis[t];
    }

    int dfs(int u, int cap) {
        if (u == t || cap == 0) return cap;
        int tmp = cap, f;
        for (int& i = cur[u]; i < G[u].size(); i++) {
            DEdge& e = es[G[u][i]];
            if (dis[e.to] == dis[u] + 1) {
                f = dfs(e.to, min(cap, e.cap));
                e.cap -= f;
                es[G[u][i] ^ 1].cap += f;
                cap -= f;
                if (cap == 0) break;
            }
        }
        return tmp - cap;
    }

    ll solve() {
        ll flow = 0;
        while (bfs()) {
            cur.assign(n + 1, 0);
            flow += dfs(s, INF);
        }
        return flow;
    }
};
```

+ 最小费用流

```cpp
const int INF = 0x7fffffff;

struct MEdge {
    int from, to, cap, cost;
    MEdge(int from, int to, int cap, int cost) : from(from), to(to), cap(cap), cost(cost) {}
};

struct MCMF {
    int n, s, t, flow, cost;
    vector<MEdge> es;
    vector<vector<int> > G;
    vector<int> d, p, a;  // dis, prev, add
    deque<bool> in;

    MCMF(int n, int s, int t) : n(n), s(s), t(t), flow(0), cost(0), G(n + 1), p(n + 1), a(n + 1) {}

    void add_edge(int u, int v, int cap, int cost) {
        G[u].push_back(es.size());
        es.emplace_back(u, v, cap, cost);
        G[v].push_back(es.size());
        es.emplace_back(v, u, 0, -cost);
    }

    bool spfa() {
        d.assign(n + 1, INF);
        in.assign(n + 1, false);
        d[s] = 0;
        in[s] = 1;
        a[s] = INF;
        queue<int> q;
        q.push(s);
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            in[u] = false;
            for (int& i : G[u]) {
                MEdge& e = es[i];
                if (e.cap && d[e.to] > d[u] + e.cost) {
                    d[e.to] = d[u] + e.cost;
                    p[e.to] = i;
                    a[e.to] = min(a[u], e.cap);
                    if (!in[e.to]) {
                        q.push(e.to);
                        in[e.to] = true;
                    }
                }
            }
        }
        return d[t] != INF;
    }

    void solve() {
        while (spfa()) {
            flow += a[t];
            cost += a[t] * d[t];
            int u = t;
            while (u != s) {
                es[p[u]].cap -= a[t];
                es[p[u] ^ 1].cap += a[t];
                u = es[p[u]].from;
            }
        }
    }
};
```

### 无向图最小割

```cpp
namespace stoer_wagner {
    bool vis[MAXN], in[MAXN];
    int G[MAXN][MAXN], w[MAXN];

    void init() {
        memset(G, 0, sizeof(G));
        memset(in, 0, sizeof(in));
    }

    void add_edge(int u, int v, int w) {
        G[u][v] += w;
        G[v][u] += w;
    }

    int search(int& s, int& t) {
        memset(vis, 0, sizeof(vis));
        memset(w, 0, sizeof(w));
        int maxw, tt = n + 1;
        for (int i = 0; i < n; i++) {
            maxw = -INF;
            for (int j = 0; j < n; j++) {
                if (!in[j] && !vis[j] && w[j] > maxw) {
                    maxw = w[j];
                    tt = j;
                }
            }
            if (t == tt) return w[t];
            s = t; t = tt;
            vis[tt] = true;
            for (int j = 0; j < n; j++) {
                if (!in[j] && !vis[j]) {
                    w[j] += G[tt][j];
                }
            }
        }
        return w[t];
    }

    int go() {
        int s, t, ans = INF;
        for (int i = 0; i < n - 1; i++) {
            s = t = -1;
            ans = min(ans, search(s, t));
            if (ans == 0) return 0;
            in[t] = true;
            for (int j = 0; j < n; j++) {
                if (!in[j]) {
                    G[s][j] += G[t][j];
                    G[j][s] += G[j][t];
                }
            }
        }
        return ans;
    }
}
```

### 树链剖分

```cpp
// 点权
vector<int> G[MAXN];
int pa[MAXN], sz[MAXN], dep[MAXN], dfn[MAXN], maxc[MAXN], top[MAXN];

void dfs1(int u) {
    sz[u] = 1;
    maxc[u] = -1;
    int maxs = 0;
    for (int& v : G[u]) {
        if (v != pa[u]) {
            pa[v] = u;
            dep[v] = dep[u] + 1;
            dfs1(v);
            sz[u] += sz[v];
            if (updmax(maxs, sz[v])) maxc[u] = v;
        }
    }
}

void dfs2(int u, int tp) {
    static int cnt = 0;
    top[u] = tp;
    dfn[u] = ++cnt;
    if (maxc[u] != -1) dfs2(maxc[u], tp);
    for (int& v : G[u]) {
        if (v != pa[u] && v != maxc[u]) {
            dfs2(v, v);
        }
    }
}

void init() {
    dep[1] = 1;
    dfs1(1);
    dfs2(1, 1);
}

ll go(int u, int v) {
    int uu = top[u], vv = top[v];
    ll res = 0;
    while (uu != vv) {
        if (dep[uu] < dep[vv]) {
            swap(u, v);
            swap(uu, vv);
        }
        res += segt.query(dfn[uu], dfn[u]);
        u = pa[uu];
        uu = top[u];
    }
    if (dep[u] > dep[v]) swap(u, v);
    res += segt.query(dfn[u], dfn[v]);
    return res;
}
```

### Tarjan

+ 割点

```cpp
int dfn[MAXN], low[MAXN], clk;

void init() { clk = 0; memset(dfn, 0, sizeof(dfn)); }

void tarjan(int u, int pa) {
    low[u] = dfn[u] = ++clk;
    int cc = (pa != 0);
    for (int v : G[u]) {
        if (v == pa) continue;
        if (!dfn[v]) {
            tarjan(v, u);
            low[u] = min(low[u], low[v]);
            cc += low[v] >= dfn[u];
        } else low[u] = min(low[u], dfn[v]);
    }
    if (cc > 1) // ...
}
```

+ 桥

```cpp
int dfn[MAXN], low[MAXN], clk;

void init() { clk = 0; memset(dfn, 0, sizeof(dfn)); }

void tarjan(int u, int pa) {
    low[u] = dfn[u] = ++clk;
    int f = 0;
    for (int v : G[u]) {
        if (v == pa && ++f == 1) continue;
        if (!dfn[v]) {
            tarjan(v, u);
            if (low[v] > dfn[u]) // ...
            low[u] = min(low[u], low[v]);
        } else low[u] = min(low[u], dfn[v]);
    }
}
```

+ 强连通分量缩点

```cpp
int dfn[MAXN], low[MAXN], clk, tot, color[MAXN];
vector<int> scc[MAXN];

void init() { tot = clk = 0; memset(dfn, 0, sizeof dfn); }

void tarjan(int u) {
    static int st[MAXN], p;
    static bool in[MAXN];
    dfn[u] = low[u] = ++clk;
    st[p++] = u;
    in[u] = true;
    for (int v : G[u]) {
        if (!dfn[v]) {
            tarjan(v);
            low[u] = min(low[u], low[v]);
        } else if (in[v]) {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (dfn[u] == low[u]) {
        ++tot;
        for (;;) {
            int x = st[--p];
            in[x] = false;
            color[x] = tot;
            scc[tot].push_back(x);
            if (x == u) break;
        }
    }
}
```

### 支配树

+ 有向无环图

```cpp
// rt是G中入度为0的点（可能需要建超级源点）
int n, deg[MAXN], dep[MAXN], up[MAXN][22];
vector<int> G[MAXN], rG[MAXN], dt[MAXN];

bool topo(vector<int>& ans, int rt) {
    queue<int> q;
    q.push(rt);
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        ans.push_back(u);
        for (int v : G[u]) {
            deg[v]--;
            if (deg[v] == 0) q.push(v);
        }
    }
    return ans.size() == n;
}

int lca(int u, int v) {
    if (dep[u] > dep[v]) swap(u, v);
    int t = dep[v] - dep[u];
    for (int i = 0; i < 22; i++) {
        if ((t >> i) & 1) v = up[v][i];
    }
    if (u == v) return u;
    for (int i = 21; i >= 0; i--) {
        if (up[u][i] != up[v][i]) {
            u = up[u][i];
            v = up[v][i];
        }
    }
    return up[u][0];
}

void go(int rt) {
    vector<int> a;
    topo(a, rt);
    dep[rt] = 1;
    for (int i = 1; i < a.size(); i++) {
        int u = a[i], pa = -1;
        for (int v : rG[u]) {
            pa = (pa == -1) ? v : lca(pa, v);
        }
        dt[pa].push_back(u);
        dep[u] = dep[pa] + 1;
        up[u][0] = pa;
        for (int i = 1; i < 22; i++) {
            up[u][i] = up[up[u][i - 1]][i - 1];
        }
    }
}
```

+ 一般有向图

```cpp
vector<int> G[MAXN], rG[MAXN];
vector<int> dt[MAXN];

namespace tl {
    int pa[MAXN], dfn[MAXN], clk, rdfn[MAXN];
    int c[MAXN], best[MAXN], sdom[MAXN], idom[MAXN];

    void init(int n) {
        clk = 0;
        fill(c, c + n + 1, -1);
        fill(dfn, dfn + n + 1, 0);
        for (int i = 1; i <= n; i++) {
            dt[i].clear();
            sdom[i] = best[i] = i;
        }
    }

    void dfs(int u) {
        dfn[u] = ++clk;
        rdfn[clk] = u;
        for (int& v: G[u]) {
            if (!dfn[v]) {
                pa[v] = u;
                dfs(v);
            }
        }
    }

    int fix(int x) {
        if (c[x] == -1) return x;
        int& f = c[x], rt = fix(f);
        if (dfn[sdom[best[x]]] > dfn[sdom[best[f]]]) best[x] = best[f];
        return f = rt;
    }

    void go(int rt) {
        dfs(rt);
        for (int i = clk; i > 1; i--) {
            int x = rdfn[i], mn = clk + 1;
            for (int& u: rG[x]) {
                if (!dfn[u]) continue; // 可能不能到达所有点
                fix(u);
                mn = min(mn, dfn[sdom[best[u]]]);
            }
            c[x] = pa[x];
            dt[sdom[x] = rdfn[mn]].push_back(x);
            x = rdfn[i - 1];
            for (int& u: dt[x]) {
                fix(u);
                idom[u] = (sdom[best[u]] == x) ? x : best[u];
            }
            dt[x].clear();
        }
        for (int i = 2; i <= clk; i++) {
            int u = rdfn[i];
            if (idom[u] != sdom[u]) idom[u] = idom[idom[u]];
            dt[idom[u]].push_back(u);
        }
    }
}
```
