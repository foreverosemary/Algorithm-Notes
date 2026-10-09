# 树形 DP 基础

## 树的平衡点

- **树的平衡点**：定义平衡点为树中的一个点，删掉该点后，剩下的若干个连通块中，最大的连通块的块大小最少。

设 $g(u)$ 为删除节点 $u$ 后最大连通块的块大小，取 $f(u)$ 为以 $u$ 为根节点的子树大小，那么显然有
$$g(u)=\max(n-f(u),\max\{f(v)\mid u\to v\})$$

## 树的最大独立集

- **树的最大权独立集**：在树上挑选出一个点集，该点集中，不存在两点之间有边，且这个点集中所有点的权值和最大。

# 子树信息/选择节点类

在树上进行动态规划时，常以节点从深到浅（子树从小到大）的顺序作为 DP 的阶段。DP 的状态表示中，第一维通常是节点编号（代表以该节点为根的子树），大多数时候采用递归的方式实现树形 DP。对于每个节点 $u$，先递归在它的每个子节点上进行 DP，回溯时，从子节点向节点 $u$ 进行状态转移。

## 练习题

- [Select Edges](https://atcoder.jp/contests/abc259/tasks/abc259_f?lang=en)

**题目大意**：给你一棵有 $n$ 个顶点的树，其中每一条边都带有一个权重。考虑从 $n-1$ 条边中选择一些边（可能一条都没有，也可能全部都是），并且对于顶点 $i$，最多可以选择 $d_i$ 条与顶点 $i$ 连接的边。求所选边的最大总权重。

**题目思路**：
- 取 $\text{dp}[u][0/1]$ 表示以 $u$ 为根节点的子树上 没有选择/选择 与父节点相连的边后的最大总权重，考虑某个子节点 $v$，那么该子节点的贡献为 $\max(\text{dp}[v][1]+w_{u,v},\text{dp}[v][0])$。
-  由于对取 $\text{dp}[v][1]+w_{u,v}$ 有次数限制，而 $\text{dp}[v][0]$ 没有，故有 $$\sum\limits_{u\to v\in S}(\text{dp}[v][1]+w_{u,v})+\sum\limits_{u\to v\notin S}\text{dp}[v][0]$$其中 $S$ 代表在子树中与 $u$ 相连被选中的边，对于 $\text{dp}[u][0]$，有 $|S|\le d_u$；而由于 $\text{dp}[u][1]$ 已经有一条边与父节点相连，故 $|S|\le d_u-1$。考虑最大化上式，可将式子变换为 $$\sum\limits_{u\to v\in S}(\text{dp}[v][1]+w_{u,v}-\text{dp}[v][0])+\sum\limits_{u\to v}\text{dp}[v][0]$$故问题转化为最大化 $\sum\limits_{u\to v\in S}(\text{dp}[v][1]+w_{u,v}-\text{dp}[v][0])$，贪心即可。

```cpp
#include<bits/stdc++.h>
#define endl '\n'
#define eb emplace_back
#define pb push_back
#define rall(x) x.rbegin(), x.rend()

using namespace std;
using ll = long long;
using vvi = vector<vector<int>>;
using vll = vector<ll>;
using pii = pair<int, int>;

const ll LLINF = 0x3f3f3f3f3f3f3f3fll;
const int N = 3e5 + 10;
ll d[N], dp[N][2];
vector<vector<pii>> g(N);

void dfs(int u, int pre) {
	vll res;
	for(auto [v, w] : g[u]) {
		if(pre == v) continue;
		dfs(v, u);
		dp[u][1] += dp[v][0];
		res.pb(dp[v][1] + w - dp[v][0]);
	}
	sort(rall(res));
	for(int i = 0; i < d[u] - 1; i++) {
		if(res[i] <= 0) break;
		dp[u][1] += res[i];
	}
	dp[u][0] = dp[u][1];
	if(d[u] - 1 < res.size()) dp[u][0] += max(0ll, res[d[u] - 1]);
	if(!d[u]) dp[u][1] = -LLINF;
}

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	int n; cin >> n;
	for(int i = 1; i <= n; i++) cin >> d[i];
	for(int i = 1; i < n; i++) {
		int u, v, w; cin >> u >> v >> w;
		g[u].eb(v, w); g[v].eb(u, w);
	}
	dfs(1, -1);
	cout << dp[1][0] << endl;
	return 0;
}

```

---
# 树上背包

- [选课](https://www.luogu.com.cn/problem/P2014)

**题目大意**：大学里有 $N$ 门课程，每门课有一定的学分 $s_i$。课程之间存在先修关系：若想修课程 $i$，必须先修课程 $k_i$（$k_i=0$ 则无先修课）。现在的目标是从中选出 $M$ 门课程，使得获得的总学分最大。

**题目思路**：
这是一个典型的 **有依赖的背包问题**:
1. **构建树形结构**：课程的依赖关系构成了一个森林。为了方便处理，我们可以引入一个虚拟根节点 $0$（学分为 $0$），将所有无先修课的节点作为 $0$ 号节点的子节点。这样森林就变成了一棵以 $0$ 为根的树。
2. **转换限制**：因为引入了虚拟根节点，且必须选根节点才能选子节点，所以题目转化为在 $N+1$ 个点中选 $M+1$ 个点（包含根节点）。

**解法一：卷积合并**

定义 $\text{dp}[u][j]$ 为在以 $u$ 为根的子树中，选择 $j$ 门课程（包含 $u$ 自己）所能获得的最大学分。
对于节点 $u$ 的每一个子节点 $v$，我们将 $v$ 的子树看作一个泛化物品。我们在计算 $\text{dp}[u][\dots]$ 时，实际上就是将 $u$ 作为一个背包，把它的所有子节点 $v$ 对应的泛化物品一个个合并进去。状态转移实际上就是分组背包的变种（或者说是泛化物品的卷积）：
$$dp[u][j] = \max(dp[u][j], dp[u][j-k] + dp[v][k])$$
其中 $k$ 是分配给子树 $v$ 的课程数量。

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N = 305;
vector<int> g[N];
int dp[N][N]; // dp[i][j] 表示以 i 为根的子树选 j 门课的最大价值
int n, m;

void dfs(int u) {
    // 初始化：选自己获得 value[u] 的价值，实际题目中通常在输入时直接存入 dp[i][1]
    // 这里的 dp[u][1] 已经在输入时被赋值为该课程的学分
    for(int v : g[u]) {
        dfs(v); // 递归处理子树，先把子树这个“泛化物品”算出来
        // 泛化物品合并（树上背包的卷积）
        // 倒序枚举容量，避免重复选取
        for(int j = m + 1; j >= 1; j--) {
            // 枚举分给子树 v 的体积 k
            // 注意：这里 k < j 是因为 u 本身至少占 1 个体积
            for(int k = 0; k < j; k++) {
                dp[u][j] = max(dp[u][j], dp[u][j - k] + dp[v][k]);
            }
        }
    }
}

int main() {
    cin >> n >> m;
    for(int i = 1; i <= n; i++) {
        int pre, score;
        cin >> pre >> score;
        g[pre].push_back(i);
        dp[i][1] = score; // 初始化：选1门课（就是选自己）的价值
    }
    
    dfs(0); // 从虚拟根节点开始
    cout << dp[0][m + 1] << endl; // 选 m+1 门（含虚拟根）
    return 0;
}
```

**解法二：DFS 序 DP**

利用 DFS 序将树形依赖转化为线性序列问题。定义 $\text{dp}[i][j]$ 表示在 DFS 序中，考虑从第 $i$ 个节点及其之后的所有节点中，选出 $j$ 个节点能获得的最大价值。对于当前节点 $u$（DFS 序中位置为 $i$），有两种决策：
1.  选择 $u$：花费一个体积，获得 $val[u]$ 价值。接下来的子问题是处理 DFS 序中的 $i+1$ 号节点（即 $u$ 的第一个子节点），剩余容量 $j-1$。
2.  不选 $u$：根据依赖关系，不选 $u$ 则其整棵子树都不能选。接下来的子问题是跳过 $u$ 的整棵子树，直接处理 DFS 序中的 $i + size[u]$ 号节点，剩余容量 $j$。
转移方程：
$$\text{dp}[i][j] = \max(\text{dp}[i+1][j-1] + val[u],\text{dp}[i+size[u]][j])$$

```cpp
#include<bits/stdc++.h>
using namespace std;

const int N = 305;
vector<int> g[N];
int sz[N], dfn[N], val[N], dp[N][N];
int n, m, tot;

// 预处理 DFS 序和子树大小
void DFS(int u) {
    sz[u] = 1; 
    dfn[++tot] = u; // 记录 DFS 序第 tot 个节点是 u
    for(int v : g[u]) {
        DFS(v);
        sz[u] += sz[v];
    }
}

int main() {
    cin >> n >> m;
    m++; // 包含虚拟根节点
    for(int i = 1; i <= n; i++) {
        int pre;
        cin >> pre >> val[i];
        g[pre].push_back(i);
    }
    
    DFS(0);
    
    // 逆序遍历 DFS 序进行 DP
    for(int i = tot; i >= 1; i--) {
        int u = dfn[i]; // 获取当前 DFS 序对应的真实节点编号
        for(int j = 0; j <= m; j++) {
            // 决策 1：选择当前节点 u
            // 必须满足 j >= 1，且转移到 DFS 序的下一个节点 i+1
            int pick = -1e9;
            if(j >= 1) pick = dp[i + 1][j - 1] + val[u];
            
            // 决策 2：不选当前节点 u
            // 跳过整个子树，转移到 DFS 序中 i + sz[u] 处
            int skip = dp[i + sz[u]][j];
            
            dp[i][j] = max(pick, skip);
        }
    }
    
    cout << dp[1][m] << endl; // DFS 序第 1 个节点（根）开始，容量 m
    return 0;
}
```

## 练习题

- [二叉苹果树](https://www.luogu.com.cn/problem/P2015)

**题目大意**：有一颗苹果树，每个节点要么有两个子节点，要么为叶节点，指定根节点为 $1$ 号节点。每个树枝（即连接节点的边）上都有一些苹果，给定要保留的与根节点相连的树枝个数 $Q$，求出最多能留住多少苹果。

**题目思路**：
- 该问题本质上与上一题是同一道，考虑如何将边约束转化为点约束。如果选择了节点 $u$，那么还需要选择子节点 $v$，才有 $w\langle u,v\rangle$（$w\langle u,v\rangle$ 表示有向边 $\langle u,v\rangle$ 上的苹果数，因为如果选择了 $v$ 而不选择 $u$，那么 $v$ 必定不与根节点相连，此时考虑该节点贡献无意义，所以只有在选择了 $u$ 节点后才考虑选择其子节点 $v$）的贡献，因此将 $w\langle u,v\rangle$ 视作 $v$ 的点权而非边权。即使根节点点权为 $0$，这对于根节点也是良定义的（单独选择根节点没有任何收益，必须还要选择其对应的子节点才会产生相应收益）。
- 由于原问题是保留 $Q$ 条边，因此现在问题可以转变为保留 $Q+1$ 个节点，且只有保留父节点时，才能考虑保留其子节点。经过这种转化后这道题目就完全等价于上面的题目，采用任意一种解法即可。

---
# 换根 DP

换根 DP 又被称为二次扫描。这类题目的特点是通常不会指定根节点， 而是需要对以每个节点为根进行一系列统计，一般通过二次扫描来求解：
- 第一次扫描：任选一个节点为根，在有根树上执行一次树形 DP，通常是预处理一些信息供第二次扫描使用。
- 第二次扫描：从刚才选出的根出发，对整棵树执行 DFS，在每次递归前进行自顶向下的推导，计算出换根后的解。

## 练习题

- [Accumulation Degree](https://www.luogu.com.cn/problem/P10974)

**题目大意**：给定一颗无根树，每一条边都有一定的容量，每条边的流量不得超过其容量。将度数为 $1$ 的节点定义为终端节点，定义 $h(x)$ 为指定节点 $x$ 为源点后可以流向其它终端节点的最大流量总和，求出最大的 $h(x)$。

**题目思路**：
- 指定任意一个节点为根节点，定义 $f(x)$ 表示从 $x$ 流向其子树的终端节点流量和，用 $pre_x$ 表示 $x$ 的父节点，那么对于第一次 DFS 有 $$f(u)=\sum\nolimits_{u\to v,v\neq pre_u}\min(\text{cap}(u,v),f(v))$$其中 $\text{cap}(u,v)$ 代表无向边 $(u,v)$ 的容量。
- 若指定的根节点为 $1$，那么有 $h(1)=f(1)$，根据递推关系可知，$h(u)$ 的计算是建立在 $h(pre_u)$ 已经被计算出来的基础上，即计算 $h(u)$ 时 $h(pre_u)$ 为已知量，根据定义可得 $$h(u)=f(u)+\min(\text{cap}(pre_u, u),h(pre_u)-\min(\text{cap}(pre_u,u),f(u)))$$其中 $\min(f(u),\text{cap}(pre_u,u))$ 为以 $pre_u$ 根节点时，从 $pre_u$ 流向 $u$ 的流量大小。那么 $$h(pre_u)-\min(\text{cap}(pre_u,u),f(u))=\sum\limits_{pre_u\to v,v\neq u}\min(\text{cap}(pre_u,v),f(v))$$故 $\min(\text{cap}(pre_u, u),h(pre_u)-\min(\text{cap}(pre_u,u),f(u)))$ 就是以 $u$ 为根节点时，从 $u$ 流向 $pre_u$ 的流量。这个过程可以通过第二次 DFS 实现，

```cpp
#include<bits/stdc++.h>
#define endl '\n'
#define eb emplace_back

using namespace std;
using ll = long long;
using pii = pair<int, int>;

const int N = 2e5 + 10;
vector<pii> g[N];
ll f[N], h[N], ans;

void dfs(int u, int pre) {
	for(auto [v, c] : g[u]) {
		if(v == pre) continue;
		dfs(v, u);
		if(g[v].size() == 1) f[u] += c;
		else f[u] += min(f[v], (ll)c);
	}
}

void dp(int u, int pre) {
	for(auto [v, c] : g[u]) {
		if(v == pre) continue;
		if(g[u].size() > 1) h[v] = f[v] + min(h[u] - min(f[v], (ll)c), (ll)c);
		else h[v] = f[v] + c;
		ans = max(ans, h[v]);
		dp(v, u);
 	}
}

void solve() {
	int n; cin >> n;
	for(int i = 1; i <= n; i++) g[i].clear();
	fill(f + 1, f + 1 + n, 0);
	fill(h + 1, h + 1 + n, 0);
	for(int i = 1; i < n; i++) {
		int u, v, c; cin >> u >> v >> c;
		g[u].eb(v, c); g[v].eb(u, c);
	}
	dfs(1, 0);
	ans = h[1] = f[1];
	dp(1, 0);
	cout << ans << endl;
}

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	int t = 1; cin >> t;
	while(t--) {
		solve();
	}
	return 0;
}
```
