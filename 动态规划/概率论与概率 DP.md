# 马尔可夫链与贝尔曼方程

## 马尔可夫链

马尔可夫链的核心性质——**状态转移的无后效性**：
>若一个随机过程满足 **未来的状态只依赖于当前状态，与过去的状态无关**，则该过程为马尔可夫链。
>数学表达：当状态为 $i$，下一个状态为 $j$ 的概率为 $p_{ij}$，对任意时刻 $t$ 和状态 $s_t$，以及任意之前可能的状态序列 $s_0,\dots,s_{t-1}$，有 $P(s_{t+1}=j \mid s_t=i, s_{t-1}=i_{t-1}, ..., s_0=i_0) = P(s_{t+1}=j \mid s_t=i)=p_{ij}$。

**关键概念**
- **状态分类**：
  - 常返状态：从该状态出发，经过无限次时间、回合或者轮次后总能回访到该状态的概率为 $1$，在问题中通常表现为非法态和终态。
  - 非常返状态：从该状态出发，经过有限次时间、回合、或者轮次后永远离开的概率为 $1$，在问题中通常表现为初态和状态转移当中的过渡态。

![[状态转移示意图.svg]]

- **转移概率矩阵**：用 $P_{ij}$ 表示从状态 $i$ 一步转移到状态 $j$ 的概率，所有转移概率构成二维状态转移矩阵 $P$，每一列的后续状态转移的系数本质为 $P_{ij}$，每一列的概率和为 $1$。
- **吸收态马尔可夫链**：存在至少一个常返状态（吸收态），且所有非常返状态都会转移到吸收态，这是概率 DP 中常见场景。

## 贝尔曼方程

贝尔曼方程是 **将状态价值与后续状态价值关联** （今天的最优选择，取决于明天可能的最好结果）的数学表达式，分为 **概率型** 和 **期望型**，其本质是 **全概率公式和期望的线性性质在马尔可夫链上的运用**。

- **状态价值**：在当前状态 $s$ 下，从这个状态出发，未来能获得的期望总收益，该收益要在某个终态（常返态）才能结算。
- **动作价值**：在当前状态 $s$ 下，选择动作 $a$ 后，能够得到的即时收益，在一些题目里为 $0$。

贝尔曼方程的本质是 **递归**—— **当前状态的价值 = 当前动作价值的即时收益 + 未来的期望价值** 。
1. **贝尔曼概率方程**：概率通常描述为 **从初始状态出发，最终到达某一结果的可能性**。对于状态 $i$，根据无后效性和全概率公式，其转移方程较为直接$$\text{dp}[s]=\sum_{s'\in \text{pre}(s)}\text{dp}[s']\cdot P_{s'\rightarrow s}$$这是因为概率具有可加性，多个路径到达同一状态的概率可以直接相加，故我们需要从前往后递推，而吸收概率方程组是贝尔曼概率方程的特殊场景。
2. **贝尔曼期望方程：无最优决策，仅描述期望**
$$V(s)=\mathbb{E}[R_{t+1}+\gamma\cdot V(S_{t+1})\mid S_t=s]$$其中：
- $V(s)$：当前状态 $s$ 的价值；
- $\mathbb{E}[\cdot]$：期望，下一步的状态是随机且未知的；
- $R_{t+1}$：从 $s$ 转移到下一个状态的动作价值带来的即时收益，可能为 $0$；
- $\gamma$：折扣因子，表示未来收益的重要性，大多数题目都为短期收益，$\gamma$ 为 $1$；
-  $S_{t+1}$：下一步的状态。

3. **贝尔曼最优方程：有最优决策，求最大价值**
$$V^*(s,a)=\max\nolimits_a\mathbb{E}[R_{t+1}+\gamma\cdot V(S_{t+1})\mid S_t=s,A_t=a]$$其中：
- $A_t$：$s$ 状态下做出的动作 $a$；
- $\max\nolimits_a$：在当前状态下，必须选能让未来收益最大的动作。

---
# 概率计算

**吸收概率**：固定一个吸收态 $s$（一个常返态 $s$ 是吸收的，也就是 $p_{ss}=1$ 且对于所有的 $k\neq s,p_{sk}=0$，定义 $a_i$ 表示链从状态 $i$ 开始，最终达到 $s$ 的概率$$a_i=P(X_n=s\mid X_0=i)$$这个概率被称为吸收概率，通过求解线性方程组得到吸收概率方程组$$a_i=\begin{cases} 1 & i=s\\a_i=0 & \text{对于所有的吸收状态 $i\neq s$}\\ \sum\limits_{ j=1}^mp_{ij}a_j &\text{对于所有的非常返状态 $i$}\end{cases}$$

### 练习题

- [Scout YYF I ](https://vjudge.net/problem/POJ-3744)

**题目大意**：在一条数轴上，侦察兵位于初始位置 $x=1$，给定 $n$ 个 $x$ 轴上的坐标 $x_i(1\le x_i\le 100000000)$ 代表地雷所处的位置，给定概率 $p$，每次行动侦察兵有 $p$ 概率跳一步，有 $1-p$ 概率跳两步，求侦察兵安全通过地雷阵的概率。

**题目思路**：
- 考虑吸收概率模型，显然到达地雷所处坐标对应到达吸收态，但这并不是题目所要求的吸收态的概率，题目要求侦察兵安全通过地雷阵，即到达 $x\gt x_n$ 处的概率，故将 $x=x_1,x_2,\dots,x_n$ 视作吸收态 $X_1,X_2,\dots,X_n$，将 $x\gt x_n$ 视作吸收态 $X_{n+1}$。根据吸收态马尔可夫链的性质可知，所有状态最后都会转移到一个吸收态，故求解转移到吸收态 $X_{n+1}$ 的过程可以视为不被 $X_1,X_2,\dots,X_n$ 中任何一个吸收态吸收的过程，取 $x_0=0$，很自然的考虑对 $[x_0+1,x_1],[x_1+1,x_2],\dots,[x_{n-1}+1,x_n]$ 进行分段处理。取 $\text{dp}[i]$ 表示到达点 $i$ 的概率， 在 $[x_{i-1}+1,x_i]$ 这一段上不被 $X_i$ 吸收的概率为 $\text{dp}[x_i-1]\cdot(1-p)$，即从 $x_i-1$ 处到达 $x_i+1$ 处的概率 $\text{dp}[x_i+1]$。
- 现考虑如何处理得到 $\text{dp}[x_i-1]$，由于 $x$ 有 $10^8$ 的规模，且有递推式 $\text{dp}[i]=\text{dp}[i-1]\cdot p+\text{dp}[i-2]\cdot (1-p)$，故构造状态转移矩阵 $P=\begin{bmatrix} p & 1-p \\ 1 & 0 \end{bmatrix}$，以及状态矩阵 $\begin{bmatrix} \text{dp}[i]\\ \text{dp}[i-1]\end{bmatrix}$，那么根据矩阵快速幂可得 $$\begin{bmatrix} \text{dp}[x_i-1]\\ \text{dp}[x_i-2]\end{bmatrix}=P^{x_i-x_{i-1}-1}\cdot \begin{bmatrix} \text{dp}[x_{i-1}+1]\\ \text{dp}[x_{i-1}]\end{bmatrix}$$所以转移到 $x_i+1$ 不被 $X_i$ 吸收的概率 $\text{dp}[x_i+1]=\text{dp}[x_i-1]\cdot(1-p)$。
- 最后考虑乘法原理可得所求答案为可以表示为$$P^{\sum_{i=1}^n(x_i-x_{i-1}-1)}\cdot \begin{bmatrix} \text{dp}[1]=1\\ \text{dp}[0]=0\end{bmatrix}\cdot(1-p)^n$$该矩阵中的第一行的那一项。
- 或者换种思路，对于 $[x_{i-1},x_i]$ 这个区间上的状态，其要么被 $X_i$ 吸收，则设从 $x_{i-1}$ 出发被 $X_i$  吸收的概率为 $p_i=P^{x_i-x_{i-1}-1}[1][1]$，要么不被 $X_i$ 吸收， 那么概率为 $(1-p_i)$，那么同样根据乘法原理可得转移到 $X_{n+1}$ 的概率表示为$$\text{dp}[1]\cdot\prod\limits_{i=1}^n(1-p_i)=\prod\limits_{i=1}^n(1-p_i)$$

```cpp
#include<stdio.h>
#include<string.h>
#include<algorithm>
#include<iostream>
#include<math.h>
using namespace std;

struct matrix {
	double p[3][3];
	int n, m;
	matrix() {memset(p, 0, sizeof p);}
};

matrix operator*(const matrix &x, const matrix &y) {
	matrix t;
	for(int i = 1; i <= x.n; i++) {
		for(int j = 1; j <= y.m; j++) {
			for(int k = 1; k <= x.m; k++) {
				t.p[i][j] += x.p[i][k] * y.p[k][j];
			}
		}
	}
	t.n = x.n, t.m = x.m;
	return t;
}

matrix qpow(matrix mat, int k) {
	matrix t;
	t.p[1][1] = t.p[2][2] = 1;
	t.n = t.m = 2;
	while(k) {
		if(k & 1) t = t * mat;
		mat = mat * mat;
		k >>= 1;
	}
	return t;
}

int n, pos[15]; 
double p;

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	while(cin >> n >> p) {
		matrix res, P;
		P.p[1][1] = p, P.p[1][2] = 1 - p, P.p[2][1] = 1, P.p[2][2] = 0, P.n = P.m = 2; // 转移矩阵
		res.p[1][1] = 1, res.p[2][1] = 0, res.n = 2, res.m = 1;	// res 储存 dp[i] dp[i - 1]
		for(int i = 1; i <= n; i++) cin >> pos[i];
		sort(pos + 1, pos + n + 1);
		for(int i = 1; i <= n; i++) {
			if(pos[i] == pos[i - 1]) continue;
			matrix fail = qpow(P, pos[i] - pos[i - 1] - 1) * res; // 跳到地雷前一格
			res.p[1][1] = fail.p[2][1] * (1 - p), res.p[2][1] = 0; // 跳到地雷后一格
		}
		printf("%.7f\n", res.p[1][1]);
	}
	return 0;
}
```


- [Bag of mice](https://vjudge.net/problem/CodeForces-148D#author=DeepSeek_zh)

**题目大意**：已知有 $w$ 只白老鼠和 $b$ 只黑老鼠 $(1\le w, b\le1000)$，公主和龙轮流进行游戏。每次公主抓到白老鼠则获胜，抓到的老鼠不会再放回；龙抓老鼠时，也是抓到白老鼠获胜，但它每抓出一只老鼠后（同样不放回），都会有另一只老鼠逃出，若最后公主没有抓到过白老鼠，也是龙获胜。每只老鼠被抓的概率和逃出的概率都是等概率的，求解公主先手获胜的概率。

**题目思路**：
- 由于两者的操作是轮流进行的，所以设 $\text{dp}[i][j][1]$ 表示在剩下 $i$ 只白老鼠和 $j$ 只黑老鼠，且当前为公主操作获胜的概率，$\text{dp}[i][j][0]$ 为龙操作获胜的概率。显然有 $$\text{dp}[0][j][0]=\text{dp}[i][0][0]=\text{dp}[i][0][1]=1(1\le i\le w,0\le j\le b)$$
- 考虑贝尔曼概率方程，以 $\text{dp}[i][j][1]$ 为例，当前有 $\frac{i}{i+j}$ 的概率直接抓到白老鼠赢，若以 $\frac{j}{i+j}$ 的概率抓到黑老鼠，其在后续局面赢得概率为 $1-\text{dp}[i][j-1][0]$，状态转移方程为 $$\begin{cases}\text{dp}[i][j][1]=\frac{i}{i+j}+\frac{j}{i+j}\cdot(1-\text{dp}[i][j-1][0])\\\text{dp}[i][j][0]=\frac{i}{i+j}+\frac{j}{i+j}\cdot(1-\frac{j-1}{i+j-1}\cdot \text{dp}[i][j-2][1]+\frac{i}{i+j-1}\cdot \text{dp}[i-1][j-1][1])\end{cases}$$
```cpp
#include<bits/stdc++.h>
#define endl '\n'
using namespace std;

const int N = 1e3 + 10;
double dp[N][N][2];

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	int w, b; cin >> w >> b;
	for(int i = 1; i <= b; i++) dp[0][i][0] = 1;
	for(int i = 1; i <= w; i++) dp[i][0][1] = dp[i][0][0] = 1;
	for(int i = 1; i <= w; i++) {
		for(int j = 1; j <= b; j++) {
			dp[i][j][1] = (1 - dp[i][j - 1][0]) * j / (i + j) + 1.0 * i / (i + j);
			dp[i][j][0] = (1 - (j - 1) * dp[i][j - 2][1] / (i + j - 1) - i * dp[i - 1][j - 1][1] / (i + j - 1)) * j / (i + j) + 1.0 * i / (i + j);
		}
	}
	cout << fixed << setprecision(9) << dp[w][b][1] << endl;
	return 0;
}
```

---
# 期望计算

根据贝尔曼方程，期望描述为 **未来收益的加权平均**。而未来收益依赖于最终状态的价值，如要计算当前状态下最大的期望收益，需要先知道所有后续状态的期望，再根据当前决策的概率分布加权合并。这是因为期望具有递推性，以最大收益期望为例，在当前局面 $S$ 下，所有可能发生的事件为 $A=\{A_1,A_2,\dots,A_n\}$，对于事件 $i$ 来说，在该事件下有决策集合 $e_i$，在该集合中选择决策转移到对应新局面 $S'_i$，取最大收益 $\max\{E(S')\mid S'\in{S'_i}\}$，对于每个事件加权合并得到简化版的贝尔曼最优方程：$$E(S)=\frac{\sum_{i=1}^n\max\nolimits_{e\in{e_i}}\{E(S')\mid S'\in{S'_i}\}}{|A|}$$用伪代码表示为

```clike
sum = 0
// 遍历所有事件
for event in A:
	max_exp = 0
	// 遍历该事件下所有可能决策
	for decision in e_i:
		// 找到最优决策
		max_exp = max(max_exp, E(next_S))
	// 合并各个事件收益
	sum += max_exp
E(S) = sum / |A|
```

## 非决策性期望

1. **平均吸收时间：关注从一个特定的非常返状态出发（初态），直到到达任意吸收态（该过程称为吸收）的平均步数**
对于任何一个状态 $i$，定义 $A$ 为吸收态集合，有 $$\begin{aligned}\mu_i=&\mathbb{E}[\text{从状态 $i$ 开始到达吸收态所需步数}]\\=&\mathbb{E}[\min\{n\ge0\mid X_n\in{A}\}\mid X_0=i]\end{aligned}$$根据全期望定理得到具有关于 $\mu_i$ 的平均吸收时间方程组 $$\mu_i=\begin{cases} 0 & \text{对于所有的常返状态 $i$}\\1+\sum\limits_{j=1}^mp_{ij}\mu_j & \text{对于所有的非常返状态 $i$}\end{cases}$$
2. **平均首访时间：关注从一个特定的非常返状态出发（初态），直到到达一个特定的常返状态（该过程称为吸收）的平均步数**
用于计算平均吸收时间的思想也可以用于计算开始于任何其它状态，到达某特定常返状态的平均时间，令 $t_i$ 表示从状态 $i$ 到特定的常返态 $s$ 的平均首访时间，定义$$\begin{aligned}t_i=&\mathbb{E}[\text{从状态 $i$ 开始首次到达状态 $s$ 的转移步数}]\\=&\mathbb{E}[\min\{n\ge0\mid X_n=s\}\mid X_0=i]\end{aligned}$$同样有平均首访时间方程组$$t_i=\begin{cases} 0 & i=s\\1+\sum\limits_{j=1}^mp_{ij}t_j & i\neq s\end{cases}$$
3. **平均回访时间：关注从一个特定的常返状态出发（初态），又重新回到自身的平均步数**
对于特定状态 $s$，定义$$\begin{aligned}t_s^*=&\mathbb{E}[\text{从状态 $s$ 开始首次回到状态 $s$ 的转移步数}]\\=&\mathbb{E}[\min\{n\ge0\mid X_n=s\}\mid X_0=s]\end{aligned}$$如果知道首次访问时间 $t_i$，就有以下平均回访方程$$t_s^*=1+\sum_{j=1}^mp_{sj}t_j$$
4. **平均访问时间：关注从一个特定的非常访状态出发（初态），直到被任意一个吸收态吸收前，对一个特定状态的平均访问次数**
对于任意状态 $i$，定义 $N_{s,i}$ 表示从特定状态 $s$ 出发，直到首次进入吸收集合 $A$ 前，访问状态 $i$ 的平均总次数，取 $\tau_A$ 为首次到达 $A$ 中任意一个状态的时刻，$I\{X_n=i\}$ 为指示函数（如果第 $n$ 步系统状态为 $i$  取  $1$，反之取 $0$）$$N_{s,i}=\mathbb{E}[\sum\limits_{n=0}^{\tau_A-1}I\{X_n=i\}\mid X_0=s]$$同样有平均访问时间方程组$$N_{s,i}=\begin{cases}0 &i\in{A}(\text{吸收态不被访问})\\1+\sum\limits_{j=1}^mp_{ji}N_{s,j}&i=s,s\notin{A}(\text{初态自身，首次访问记 1 次})\\\sum\limits_{j=1}^mp_{ji}N_{s,j}&i\notin{A},i\neq s(\text{非初态暂态，仅依赖转移})\end{cases}$$

### 练习题

- [T627601 Collecting Bugs (POJ - 2096 )](https://www.luogu.com.cn/problem/T627601)

**题目大意**：一个软件有 $s$ 个子系统，会产生 $n$ 种 bug，保证每天只能发现一个 bug，问每种 bug 都被发现过至少一次且每个子系统里都找到过 bug 所需天数期望。

**题目思路**：
考虑平均首访时间模型，取 $\text{dp}[i][j]$ 表示已经在 $j$ 个子系统里找到 $i$ 种 bug 的的状态下（非常返态作为过渡态）到达吸收态期望天数，有平均首访时间方程 $$\text{dp}[i][j]=1+\text{dp}[i+1][j]\cdot p_1+\text{dp}[i][j+1]\cdot p+\text{dp}[i+1][j+1]\cdot p_3+\text{dp}[i][j]\cdot p_4$$其中：
- $p_1=(1-\frac{i}{n})\cdot\frac{j}{s}$，表示找到已有 bug 种类之外的 bug 概率乘以在已经找到过 bug 的子系统里又找到 bug 的概率。
- $p_2=\frac{i}{n}\cdot(1-\frac{j}{s})$，表示又找到已找到过的 bug 种类的概率乘以在新的子系统里找到 bug 的概率。
- $p_3=(1-\frac{i}{n})\cdot(1-\frac{j}{s})$，参考前文解释。
- $p_4=\frac{i}{n}\cdot\frac{j}{s}$，参考前文解释。
方程里的加 $1$ 代表从该状态向任意状态转移（哪怕是向自身转移）都需要花费一天的时间。将该方程移项得到$$(1-p_4)\cdot\text{dp}[i][j]=1+\text{dp}[i+1][j]\cdot p_1+\text{dp}[i][j+1]\cdot p_2+\text{dp}[i+1][j+1]\cdot p_3$$代入化简得到$$\begin{aligned}\text{dp}[i][j]=&\frac{1}{n\cdot s-i\cdot j}(\text{dp}[i+1][j]\cdot(n-i)j+\text{dp}[i][j+1]\cdot(s-j)i\\&+\text{dp}[i+1][j+1]\cdot(n-i)(s-j)+n\cdot s)\end{aligned}$$

```cpp
#include<bits/stdc++.h>
#define endl '\n'
using namespace std;
using ll = long long;

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	ll n, s; cin >> n >> s;
	vector<vector<double>> dp(n + 2, vector<double>(s + 2));
	for(int i = n; i >= 0; i--) {
		for(int j = (i == n ? s - 1 : s); j >= 0; j--) {
			double p1 = (n - i) * j, p2 = (s - j) * i, p3 = (n - i) * (s - j);
			dp[i][j] = (dp[i + 1][j] * p1 + dp[i][j + 1] * p2 + dp[i + 1][j + 1] * p3 + n * s) / (n * s - i * j);
		}
	}
	cout << fixed << setprecision(4) << dp[0][0] << endl;
	return 0;
}

```

---
## 决策型期望

### 练习题

- [P2473 [SCOI2008] 奖励关](https://www.luogu.com.cn/problem/P2473)

**题目大意**：给定 $k$ 次抛出宝物的次数和宝物的种类数 $n$，获得第 $i$ 种会取得收益 $p_i$，但该宝物有一个获取前提集合 $s_i$，只有当该前提集合内所有宝物都至少被获取过一次，才能获得这个宝物，询问最优决策下的期望分值。

**题目思路**：
考虑贝尔曼方程，取当前局面 $E(s,\text{round})$，已有宝物的种类集合 $s$ 及当前轮数 $\text{round}$，发生事件 $A_i$ 为抛出了宝物 $i$：
- 若已有宝物的种类集合满足宝物 $i$ 的前置集合 $need[i]$，即 $s\ \&\ need[i]=need[i]$，那么可做出两种决策，获取该宝物或者不获取该宝物，分别对应的下一个局面为 $S'_1(s\mid(1 << i), \text{round}+1)$ 和 $S'_2(s,\text{round}+1)$，那么该事件的最大期望收益为 $\max(E(S'_1),E(S'_2))$。
- 若已有宝物的种类集合不满足宝物 $i$ 的前置集合 $need[i]$，即 $s\ \&\ need[i]\neq need[i]$，那么只能做出不获取该宝物这一种决策。
将所有事件的最大期望加权合并 $E(S)=\frac{1}{n}\sum_{i=1}^nE(A_i)$ 即可得到答案。

```cpp
#include<bits/stdc++.h>
#define endl '\n'
using namespace std;
using ll = long long;

const int INF = 0x3f3f3f3f;
const int N = 16, M = 1 << N;

ll k, n;
ll p[N], need[N]; // p[i]/need[i] 表示第 i 种宝物的收益/前置宝物集合
double dp[105][M]; // dp[round][s] 表示在第 round 轮拥有的宝物种类集合为 s 的最大期望

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	cin >> k >> n;
	for(int i = 0; i < n; i++) {
		cin >> p[i];
		int pre;
		while(cin >> pre && pre) {
			need[i] |= 1 << (pre - 1);
		}
	}
	for(int i = 0; i < 105; i++) {
		for(int j = 0; j < M; j++) dp[i][j] = -INF;
	}
	
	function<double(ll, ll)> dfs = [&](ll s, ll round) -> double {
		if(k == round) return 0; // 游戏结束
		if(dp[round][s] > 1 - INF) return dp[round][s];
		double exp = 0; // 当前局面最大期望
		for(int i = 0; i < n; i++) {
			double gain = 0.0; // 记录最优决策收益
			if((s & need[i]) == need[i]) {
				gain = dfs(s | (1 << i), round + 1) + p[i];
			}
			gain = max(gain, dfs(s, round + 1));
			exp += gain / n; // 加权合并不同事件的最优决策收益
		}
		return dp[round][s] = exp;
	};

	cout << fixed << setprecision(6) << dfs(0, 0) << endl;
	return 0;
}
```

---
## 哈希重载

我们经常用单个数字表示某个状态，当状态稍复杂时，我们还可以将状态压缩在一个数字的每一位上，可如果状压也难以记录状态的时候呢，考虑到我们的目的是存取一个状态和其对应的值，于是自然地想到哈希表可以将状态映射成健并存取其对应的值构成键值对，但这需要我们进行哈希重载，以下面这道题目演示如何进行哈希重载。

- [Yacht](https://vjudge.net/problem/AtCoder-abc421_e#author=GPT_zh)

**题目大意**：给你五颗六面骰子，每个骰子六个面上写着 $A_1,A_2,\dots,A_6$，每个面出现的概率都是 $\frac{1}{6}$，进行一次游戏规则如下：
- 第一轮：掷出所有五颗骰子，观察结果，然后保留任意数量（可以是零）的骰子的结果不变。
- 第二轮：重新掷出未保留结果的骰子，观察结果，再次保留任意数量（可以是零）的骰子的结果不变。之前留下的骰子依然保留。
- 第三轮：再一次掷出所有未保留结果的骰子，选择一个数字 $X$。设 $n$ 是五颗骰子中显示数字 $X$ 的骰子数量，游戏得分为 $n\cdot X$ 分。
求解最优策略下的最大期望得分。

**题目思路**：
- 该题目的一大难点在于如何表示状态，我们既要存储当前游戏的轮数，又要存储当前的局面（包括保留的骰子数目以及每个骰子的结果），寻常的数组存储和状压存储难以实现，这时候就需要哈希重载：
```cpp
// 状态重载
struct State {
	int round; // 当前游戏轮数
	vector<int> keep; // 保留的骰子信息
	// 判断两状态是否相等
	bool operator==(const State &s) const {
		return round == s.round && keep == s.keep;
	}
};
// 哈希函数重载
struct Hash {
	size_t operator()(const State &s) const {
		size_t hash = s.round;
		// 尽量减少哈希碰撞
		for(int i = 0; i < s.keep.size(); i++) {
			hash = hash * 7 + s.keep[i];
		}
		return hash;
	}
};
```
- 现在我们就可以用 `unordered<State, double, Hash> dp` 来存储状态对应的键值对，对于这道题目，考虑贝尔曼最优方程，对于当前局面 $S$，剩余 $rem$ 个骰子可以投掷，那么就有 $6^{rem}$ 个可能的骰子结果（即事件数 $A=6^{rem}$，第 $i$ 个事件为 $A_i$），对于一个确定的骰子结果，除了第三轮，都有 $2^{rem}$ 种选择方式（即每个事件都有 $2^{rem}$ 种决策，取第 $i$ 个事件的决策集为 $e_i$，该事件能够转移到的状态集合为 $S'_i$），套用贝尔曼最优方程可知$$E(S)=\frac{\sum_{i=1}^{6^{rem}}\max_{e\in e_i}\{E(S')\mid S'\in{S'_i}\}}{6^{rem}}$$
```cpp
#include<bits/stdc++.h>
#define endl '\n'
#define pb push_back
using namespace std;

using ll = long long;
using vi = vector<int>;

struct State {
	int round;
	vi keep;
	bool operator==(const State &s) const {
		return round == s.round && keep == s.keep;
	}
};

struct Hash {
	size_t operator()(const State &s) const {
		size_t hash = s.round;
		for(size_t i = 0; i < s.keep.size(); i++) {
			hash = hash * 7 + s.keep[i];
		}
		return hash;
	}
};

int A[7];

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	for(int i = 0; i < 6; i++) cin >> A[i];
	unordered_map<State, double, Hash> dp;
	
	function<double(const State &)> dfs = [&](const State &s) -> double {
		// 游戏结束返回结果
		if(s.round == 3) {
			unordered_map<int, int> cnt;
			for(int i = 0; i < 5; i++) {
				cnt[A[s.keep[i]]]++;
			}
			int mx = 0;
			for(auto [x, ct] : cnt) {
				mx = max(mx, x * ct);
			}
			return mx;
		}
		if(dp.count(s)) return dp[s];
		double exp = 0; // 记录所有局面的最优期望的加权平均
		int rem = 5 - s.keep.size(), t = pow(6, rem); // 局面总数
		if(s.round == 2) {
			// 最后一局无决策空间
			for(int i = 0; i < t; i++) {
				State nxt = s; nxt.round++;
				for(int j = 0, x = i; j < rem; j++, x /= 6) {
					nxt.keep.pb(x % 6);
				}
				exp += (double)dfs(nxt);
			}
		} else {
			for(int i = 0; i < t; i++) {
				vi res(rem); // 记录当前局面掷出的骰子
				for(int j = 0, x = i; j < rem; j++, x /= 6) res[j] = x % 6;
				double mx = 0; // 记录当前局面所有决策最值
				for(int j = 0; j < (1 << rem); j++) {
					State nxt = s; nxt.round++;
					for(int k = 0; k < rem; k++) {
						if(j >> k & 1) nxt.keep.pb(res[k]);
					}
					mx = max(mx, dfs(nxt));
				}
				exp += mx;
			}
		}
		return dp[s] = exp / t;
	};
	
	State st; st.round = 0; // 初状态
	cout << fixed << setprecision(10) << dfs(st) << endl;
	return 0;
}
```

---
## 期望的线性性质

当问题的状态空间及其庞大（例如涉及到集合、排列组合时）以至于无法进行动态规划时，就要回归期望的性质求解：
- 期望的线性可加性：对于任意随机变量 $X$ 和 $Y$，无论是否独立，都有 $$E(aX+bY)=aE(X)+bE(Y)$$这代表可以将一个复杂随机变量的期望，分解为多个简单随机变量的期望之和。
- 全期望公式：假设 $\{A_i\mid i=1,2,3,\dots\}$ 是一个概率空间的有限或者可数无限的分割，则对任意事件 $X$ 有 $$E(X)=\sum P(A_i)E(X\mid A_i)$$
通过以下例题展示求解过程：
- [Play It by Ear](https://qoj.ac/contest/2606/problem/14944)

**题目大意**：有标号 $1\sim 2n$ 的 $2n$ 张不同卡牌，被均匀随机地洗牌并叠成一副牌堆。抽取顶部 $n$ 张牌作为起始手牌，而牌堆剩余牌的顺序是不可见的。每一轮，从手牌中任选一张牌打出，将其置于牌堆底部然后抽取当前牌堆顶部的牌，手牌数量从而保持为 $n$。给出 $m$ 个任务按顺序完成（即只有在第 $i-1$ 个任务被完成后，才能进行第 $i$ 个任务），第 $i$ 个任务要求打出标号为 $a_i$ 的牌，且事先知道完整的 $a_1,a_2,\dots,a_m$（值可能重复）。计算在最优纯策略的情况下完成全部 $m$ 个任务所需轮数的最小期望值。

---
## 状态分布演化

初始状态概率分布定义为 $$P^{(0)}=\begin{bmatrix}I_1 & I_2 &\dots &I_n\end{bmatrix}$$其中 $I_i$ 表示初始态为第 $i$ 个状态的概率，再给定状态转移矩阵 $$P=\begin{bmatrix}p_{11}&p_{12}&\dots&p_{1n}\\p_{21}&p_{22}&\cdots&p_{2n}\\\vdots&\vdots&\vdots&\vdots\\p_{n1}&p_{n2}&\cdots&p_{nn}\end{bmatrix}$$求马尔可夫链在 $m$ 步后的状态概率分布有$$\begin{aligned}P^{(1)}=&P^{(0)}P\\P^{(2)}=&P^{(1)}P\\\vdots\\P^{(m)}=&P^{(m-1)}P\end{aligned}$$定义稳态分布 $$\pi=\begin{bmatrix}\pi_1 & \pi_2 &\dots &\pi_n\end{bmatrix}$$即 $\pi=\lim\limits_{m\rightarrow\infty}P^{(m)}$，根据 $\pi$ 的性质有 $$\begin{cases}\pi\cdot P=\pi\\\pi_1+\pi_2+\cdots\pi_n=1\end{cases}$$由于马尔可夫链转移矩阵的特殊性质（每行和为 $1$），将 $\pi\cdot P=\pi$ 展开后，得到的 $n$ 个方程是线性相关的，秩为 $n-1$ ，因此需要补充归一化条件 $\pi_1+\pi_2\dots+\pi_n=1$，形成 $n$ 个独立方程的方程组 $$\begin{cases}\sum_{j=1}^n\pi_j\cdot p_{ji}=\pi_i&(i=1,2,\dots,n-1)\\\sum_{j=1}^n\pi_j=1\end{cases}$$再使用高斯消元求解即可。

---
# 有后效性概率计算

在求解概率 DP 时，常常会遇到有后效性概率的计算，但这里的有后效性不代表马尔可夫链的无后效性不成立，而是指由于状态的相互依赖（在状态转移图中表示成环，而非自环），当前根据马尔可夫链的性质推导出的递推式中，含有还未计算过的状态，这就是有后效性概率 DP 计算，这个递推式的得出是建立在马尔可夫链的无后效性上的。

## 单后效性

单后效性是指所有状态的后效性都集中在同一个状态上（例如游戏中的某种惩罚机制只会让你回到起点，其余情况正常推进游戏）。此时，我们将这个状态称为 **回归锚点**，它是所有回溯路径（表现为环）的唯一终点，也是整个 DP 系统的根未知量，以下面这道例题讲解如何处理。

- [One Person Game](https://vjudge.net/problem/ZOJ-3329)

**题目大意**：给定三个骰子，分别有 $K_1,K_2,K_3$ 个面，每一个骰子都是公平的（即每一个骰子中从 $1$ 到 $K_1,K_2,K_3$ 的每一个面出现的概率分别为 $\frac{1}{K_1},\frac{1}{K_2},\frac{1}{K_3}$）。游戏初始得分从 $0$ 分开始，同时掷出三个骰子，倘若 $1$ 号骰子掷出 $a$，且 $2$ 号骰子掷出 $b$，且 $3$ 号骰子掷出 $c$，那么得分归零，否则得分加上这三个骰子上的数字之和，当得分大于等于 $n$ 时游戏结束，求游戏结束的期望轮数。

**题目思路**：
- 将得分大于等于 $n$ 的状态视为吸收态 $X_n$，起始态为 $X_0$，取 $\text{dp}[i]$ 表示得分 $i$ 时所需期望轮数，$p_k$ 表示单轮得到 $k$ 分的概率，$p_0=\frac{1}{K_1}\cdot\frac{1}{K_2}\cdot\frac{1}{K_1}$ 表示得分归零的概率。依据平均首访时间方程组得到状态转移方程 $$\text{dp}[i]=\sum\nolimits_{k=3,k\neq a+b+c}^{K_1+K_2+K_3} (\text{dp}[i+k]\cdot p_k)+\text{dp}[0]\cdot p_0+1$$边界为 $\text{dp}[n]=0$。
- $\text{dp}[0]$ 为我们要求解的答案，但是出现在了状态转移方程里，这就是有后效性的体现。也就是说，起始态 $X_0$ 对应前文提及的回归锚点，所有状态的重置操作都指向它，所有状态的未知量最终都可以只通过它来表达（整个 DP 系统的根未知量），故所有非吸收态都能表达成关于这单个回归锚点 $X_0$ 的线性方程 $\text{dp}[i]=A[i]\cdot \text{dp}[0]+B[i]$。
- 将线性方程形式代入状态转移方程得到$$\begin{aligned}\text{dp}[i]=&\sum(A[i+k]\cdot \text{dp}[0]\cdot p_k+B[i+k]\cdot p_k)+\text{dp}[0]\cdot p_0+1\\=&(\sum (p_k\cdot A[i+k])+p_0)\cdot \text{dp}[0]+\sum (p_k\cdot B[i+k])+1\\=&A[i]\cdot \text{dp}[0]+B[i]\end{aligned}$$比较系数可知$$\begin{aligned}A[i]=&\sum (p_k\cdot A[i+k])+p_0\\B[i]=&\sum (p_k\cdot B[i+k])+1\end{aligned}$$而对于 $\text{dp}[0]$ 来说有 $\text{dp}[0]=A[0]\cdot \text{dp}[0]+B[0]$，即$$\text{dp}[0]=\frac{B[0]}{1-A[0]}$$而 $A[0],B[0]$ 又可以通过上述的递推式求得。

```cpp
#include<stdio.h>
#include<string.h>
#include<iostream>
#include<algorithm>
#include<vector>
using namespace std;
const int N = 1e4 + 10;

void solve() {
	int n, k1, k2, k3, a, b, c; cin >> n >> k1 >> k2 >> k3 >> a >> b >> c;
	vector<double> A(N), B(N), pk(N);
	double p = 1.0 / k1 / k2 / k3; // 得分归零概率
	// 初始化 pk 数组
	for(int i = 1; i <= k1; i++) {
		for(int j = 1; j <= k2; j++) {
			for(int k = 1; k <= k3; k++) {
				if(i == a && j == b && k == c) continue;
				pk[i + j + k] += p; // pk[k] 表示得到 k 分的概率
			}
		}
	}
	for(int i = n; i >= 0; i--) {
		A[i] = p, B[i] = 1;
		for(int k = 3; k <= k1 + k2 + k3; k++) {
			// 单轮总得分至少为 3 分
			A[i] += pk[k] * A[i + k];
			B[i] += pk[k] * B[i + k];
		}
	}
	printf("%.15f\n", B[0] / (1 - A[0]));
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

倘若将题目修改一下，当触发得分归零条件时，得分并非归零，而是若当前得分比给定的 $K(0\lt K\lt K_1+K_2+K_3)$ 值大时得分变为 $K$，否则仍然加上三个骰子上的数字之和。此时回归锚点变为 $X_K$，状态转移方程变为$$\text{dp}[i]=\begin{cases} 0 & i=n\\\sum(\text{dp}[i+k]\cdot p_k)+\text{dp}[K]\cdot p_K+1 &K\le i\lt n\\\sum(\text{dp}[i+k]\cdot p_k &0\le i\lt K\end{cases}$$那么具有单后效性的状态只会聚焦于 $X_K,X_{K+1},\dots,X_{n-1}$ 这些状态上，只需要对这些状态递推多项式求出 $\text{dp}[K]$，再从 $X_K$ 往前普通递推即可求出 $\text{dp}[0]$（相当于无后效性）。

### 练习题

- [Activation](https://vjudge.net/problem/HDU-4089#author=GPT_zh)

**题目大意**：给定一个有 $n$ 个人的队列在排队申请进入游戏，一个粉丝排在第 $m$ 个，每次服务器处理队头第一个人的请求：
- $p_1$ 概率激活失败，队列不变，下一次处理器重复处理该人请求。
- $p_2$ 概率连接失败，队头的元素被调到队尾等待处理。
- $p_3$ 概率激活成功，队头元素成功进入游戏，被永久从队列中删除。
- $p_4$ 概率服务器崩溃，所有在队列中的人都再也不会得到进入游戏的机会。
倘若这个粉丝排在前 $k$ 个位置时服务器崩溃，他就会觉得很可惜，求这种情况发生的概率。

**题目思路**：
- 取 $\text{dp}[i][j]$ 表示队列中一共 $i$ 个人而该粉丝排在第 $j$ 位时该情况发生的概率，根据吸收概率方程组得到状态转移方程$$\text{dp}[i][j]=\begin{cases}p_1\cdot \text{dp}[i][1]+p_2\cdot \text{dp}[i][i]+p_4 &j=1\\ p_1\cdot \text{dp}[i][j]+p_2\cdot \text{dp}[i][j-1]+p_3\cdot \text{dp}[i-1][j-1]+p_4 &1\lt j\le k\\p_1\cdot \text{dp}[i][j]+p_2\cdot \text{dp}[i][j-1]+p_3\cdot \text{dp}[i-1][j-1] &k\lt j\le n\end{cases}$$化简得$$\text{dp}[i][j]=\begin{cases}\frac{p_2}{1-p_1}\cdot \text{dp}[i][i]+\frac{p_4}{1-p_1} &j=1\\ \frac{p_2}{1-p_1}\cdot \text{dp}[i][j-1]+\frac{p_3}{1-p_1}\cdot \text{dp}[i-1][j-1]+\frac{p_4}{1-p_1} &1\lt j\le k\\\frac{p_2}{1-p_1}\cdot \text{dp}[i][j-1]+\frac{p_3}{1-p_1}\cdot \text{dp}[i-1][j-1] &k\lt j\le n\end{cases}$$
- 可以看出回归锚点为处于队尾的状态，故 $\text{dp}[i][i]$ 作为整个 DP 的根未知量，可以将其它所有状态表示为 $\text{dp}[i][j]=A[j]\cdot dp[i][i]+B[j]$，由 $\text{dp}[i][1]=\frac{p_2}{1-p_1}\cdot \text{dp}[i][i]+\frac{p_4}{1-p_1}$ 可知 $A[1]=\frac{p_2}{1-p_1},B[1]=\frac{p_4}{1-p_1}$，将 $\text{dp}[i-1][j-1]$ 视作前一行推出的已知量 $c$，可以得到 $$\text{dp}[i][j]=\begin{cases}\frac{p_2}{1-p_1}\cdot A[j-1]\cdot \text{dp}[i][i]+(\frac{p_2}{1-p_1}\cdot B[j-1]+\frac{p_3}{1-p_1}\cdot c+\frac{p_4}{1-p_1}) &1\lt j\le k\\\frac{p_2}{1-p_1}\cdot A[j-1]\cdot \text{dp}[i][i]+(\frac{p_2}{1-p_1}\cdot B[j-1]+\frac{p_3}{1-p_1}\cdot c)  &k\lt j\le n\end{cases}$$比较系数可知$$\begin{aligned}A[j]=&\frac{p_2}{1-p_1}\cdot A[j-1]\\B[j]=&\frac{p_2}{1-p_1}\cdot B[j-1]+\frac{p_3}{1-p_1}\cdot c+\frac{p_4}{1-p_1}\cdot[1\lt j\le k]\end{aligned}$$而 $\text{dp}[i][i]=\frac{B[i]}{1-A[i]}$。

```cpp
#include<stdio.h>
#include<iostream>
#include<math.h>
#include<algorithm>
#include<cstring>
#include<vector>
using namespace std;
using ll = long long;

const int N = 2e3 + 10;
ll n, m, k;
double p1, p2, p3, p4;
vector<vector<double>> dp(2, vector<double>(N));

void solve() {
	for(int i = 0; i < 2; i++) {
		for(int j = 0; j < N; j++) dp[i][j] = 0.0;
	}
	for(int i = 1; i <= n; i++) {
		// 初始化递推系数
		vector<double> A(i + 1), B(i + 1);
		A[1] = p2 / (1 - p1), B[1] = p4 / (1 - p1);
		for(int j = 2; j <= i; j++) {
			A[j] = p2 / (1 - p1) * A[j - 1];
			B[j] = p2 / (1 - p1) * B[j - 1] + p3 / (1 - p1) * dp[0][j - 1] + p4 / (1 - p1) * (j <= k);
		}
		// 计算根未知量
		dp[1][i] = B[i] / (1 - A[i]);
		// 递推得到每个状态
		for(int j = i - 1; j >= 1; j--) {
			dp[1][j] = A[j] * dp[1][i] + B[j];
		}
		swap(dp[0], dp[1]);
	}
	printf("%.5lf\n", dp[0][m]);
}

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	while(cin >> n >> m >> k >> p1 >> p2 >> p3 >> p4) {
		if(p4 < 1e-4) {
			cout << "0.00000\n";
			continue;
		}
		solve();
	}
	return 0;
}
```

- [Maze](https://vjudge.net/problem/HDU-4035#author=DeepSeek_zh)

**题目大意**：给定一颗 $n$ 个节点的树，玩家初始在 $1$ 号节点，当玩家移动到第 $i$ 号节点时，有 $e_i$ 的概率逃脱，有 $k_i$ 的概率回到 $1$ 号节点（此次移动不会被计入移动步数），有 $1-e_i-k_i$ 的概率随机向第 $i$ 个节点的所有相邻节点移动，求解玩家逃脱的期望步数。

**题目思路**：
- 取 $X_1,X_2,\dots,X_n$ 为玩家在节点 $1,2,\dots,n$ 的状态，容易得出该题目中 $X_1$ 为回归锚点，取 $\text{dp}[i]$ 表示位于节点 $i$ 时逃脱的期望步数，有状态转移方程$$\text{dp}[i]=e_i\cdot0+k_i\cdot \text{dp}[1]+\frac{1-e_i-k_i}{\deg(i)}\cdot((\text{dp}[pre]+1)+\sum(\text{dp}[son]+1))$$
- 但该题目与前面的题目不一样的是状态不只是单一方向往后转移，由于树形结构，其还会向父节点回溯转移，因此不能再像之前的题目一样简单地取 $\text{dp}[i]=A[i]\cdot \text{dp}[1]+B[i]$，而是设 $\text{dp}[i]=A[i]\cdot \text{dp}[1]+B[i]\cdot \text{dp}[pre]+C[i]$，这样才能将状态转移方程中的单后效性消除（同时有 $\text{dp}[1]$ 和 $\text{dp}[pre]$ 看似是多后效性，但事实上 $\text{dp}[pre]$ 也被 $\text{dp}[1]$ 隐式表达了，其仅仅作为中间状态起传递作用，在真正的多后效性中，各个状态是相互独立的，这会在后文中体现）。
- 将线性方程代入状态转移方程，取 $p_i=\frac{1-e_i-k_i}{\deg[i]}$ 可得$$\begin{aligned}(1-p_i\sum B[son])\cdot \text{dp}[i]=&(k_i+p_i\sum A[son])\cdot \text{dp}[1]\\&+p_i\cdot \text{dp}[pre]+p_i\cdot(\sum C[son]+\deg[i])\end{aligned}$$比较系数可得 $$\begin{aligned}A[i]&=\frac{k_i+p_i\sum A[son]}{1-p_i\sum B[son]}\\B[i]&=\frac{p_i}{1-p_i\sum B[son]}\\C[i]&=\frac{p_i\cdot(\sum C[son]+\deg[i])}{1-p_i\sum B[son]}\end{aligned}$$
- 边界条件为当 $i$ 为子节点时，$A[i]=k_i,B[i]=p_i,C[i]=p_i\cdot\deg[i]=1-e[i]-k[i]$，再由 $\text{dp}[1]=A[1]\cdot \text{dp}[1]+C[1]$ 可解得 $\text{dp}[1]=\frac{C[1]}{1-A[1]}$，由此也可见真正的答案只与 $A[i],C[i]$ 有关，而与 $B[i]$ 无关。

```cpp
#include<stdio.h>
#include<string.h>
#include<algorithm>
#include<iostream>
#include<math.h>
#include<vector>
using namespace std;

const int N = 1e4 + 10;
const double eps = 1e-9;
vector<vector<int>> g(N);
int k[N], e[N];
double A[N], B[N], C[N];

void dfs(int u, int pre) {
	int m = g[u].size();
	double ki = k[u] / 100.0, ei = e[u] / 100.0;
	double p = (1 - ki - ei) / m; 
	double den = 1;
	A[u] = ki, B[u] = p, C[u] = 1 - ki - ei;
	for(int v : g[u]) {
		if(v == pre) continue;
		dfs(v, u);
		den -= p * B[v];
		A[u] += p * A[v];
		C[u] += p * C[v];
	}
	A[u] /= den, B[u] /= den, C[u] /= den;
}

void solve() {
	int n; cin >> n;
	for(int i = 1; i < n; i++) {
		int x, y; cin >> x >> y;
		g[x].push_back(y), g[y].push_back(x);
	}
	for(int i = 1; i <= n; i++) cin >> k[i] >> e[i];
	dfs(1, -1); // 初始化系数矩阵
	if(fabs(1 - A[1]) <= eps) printf("impossible\n");
	else printf("%.6lf\n", C[1] / (1 - A[1]));
	for(int i = 1; i <= n; i++) g[i].clear();
}

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	int t = 1; cin >> t;
	for(int i = 1; i <= t; i++) {
		printf("Case %d: ", i);
		solve();
	}
	return 0;
}
```

---
## 多后效性

多后效性是后效性概率 DP 的另一类核心场景，其核心特征是状态间形成多个独立的依赖环，不存在唯一的回归锚点，多个未知状态相互嵌套出现在递推式中，无法通过单变量线性表达简化，必须通过构建方程组再用高斯消元求解。

相交于单后效性 **所有依赖最终汇聚于一个根未知量** 的特点，多后效性的关键区别在于：
- 依赖结构不同：但后效性是 **多对一** 的依赖，多后效性是 **多对多** 的依赖。
- 求解逻辑不同：单后效性可以通过 **线性表达式代入递推** 逐步消去后效性，多后效性需将每个未知状态视为方程变量，直接构建线性方程组，通过矩阵运算或者高斯消元求解。
当遇到以下特征的概率 DP 问题时，通常属于多后效性范畴：
- 状态转移图中存在 **至少两个不共享回归锚点的环**（例如两个独立的循环组，状态 $1$ 依赖状态 $2$，状态 $2$ 依赖状态 $1$；同时状态 $3$ 依赖状态 $4$，状态 $4$ 依赖状态 $3$）。
- 无法将任意一个状态表示为 **单一未知量的线性函数**，递推式中会同时出现两个及以上未求解的独立状态变量。
- 常见于 **状态间的双向转移** 的场景，如网格中相邻单元格的概率依赖、多角色交互的状态循环等。
可以通过下面这一道例题感受多后效性的不同之处。

- [Where is the canteen](https://vjudge.net/problem/HDU-2262#author=DeepSeek_zh)
- 另一道相似题目为 [First Knight](https://vjudge.net/problem/UVA-12177)

**题目大意**：从仅有的一个起点开始，每要移动一步，会检查上下左右的相邻区域是否可通行，并等概率随机选择一个可行方向前进，直到到达一个食堂为止（食堂可能会有多个，网格图中还包括障碍），求解到达食堂的期望步数。

**题目思路**：
- 先通过一次 bfs 遍历到所有能够从起点遍历到的点，如果没有餐馆被访问过，那么直接输出 $-1$。同时，对于无法到达的点，无需再对其做任何考虑。
- 将每个点映射到状态 $X_1,X_2,\dots,X_n$ 上，用 $s$ 表示起点，$e$ 表示终点，取 $\text{dp}[i]$ 表示从状态 $i$ 开始到达 $e$ 的期望步数，考虑平均首访时间方程组可得状态转移方程 $$\text{dp}[i]=\begin{cases}0 &X_i=e\\1+\frac{1}{\deg(i)}\sum_{i\rightarrow j}\text{dp}[j] &X_i\neq e\end{cases}$$这个状态转移方程看似很简单的，但却体现了一种无向性，$X_i$ 与 $X_j$ 之间相互依赖，而 $X_j$ 又可以与其它的未知状态相互依赖，这就构成了多后效性。这种情况下通常需要根据状态转移方程构建方程组进行高斯消元，对状态转移方程变形有 $$\frac{1}{\deg(i)}\text{dp}[i]-\sum_{i\rightarrow j}\text{dp}[j]=\deg(i)$$将每个 $\text{dp}[i]$ 视作方程组里的未知元，再进行高斯消元求解即可。

```cpp
#include <iostream>
#include <queue>
#include <algorithm>
#include <cmath>
#include <iomanip>
#include <cstring> 
#define ep emplace

using namespace std;
using pii = pair<int, int>;

const int N = 17;
const double eps = 1e-7;
char g[N][N];
int n, m, sx, sy;
bool vis[N][N];
double mat[230][230];
int dx[4] = {1, -1, 0, 0}, dy[4] = {0, 0, 1, -1};

// 映射
int ver(int x, int y) {
	return y - 1 + (x - 1) * m;
}

bool bfs(int x, int y) {
	queue<pii> q;
	q.ep(x, y);
	bool reach = false;
	vis[x][y] = 1;
	while(q.size()) {
		int x = q.front().first, y = q.front().second; q.pop();
		if(g[x][y] == '$') reach = true;
		for(int i = 0; i < 4; i++) {
			int nx = x + dx[i], ny = y + dy[i];
			if(nx < 1 || ny < 1 || nx > n || ny > m || g[nx][ny] == '#' || vis[nx][ny]) continue;
			vis[nx][ny] = 1;
			q.ep(nx, ny);
		}
	}
	return reach;
}

void Guass(int n) {
    for(int i = 0; i < n; i++) {
        int lim = i;
        for(int j = 0; j < n; j++) {
            if(j < i && fabs(mat[j][j]) >= eps) continue;
            if(fabs(mat[j][i]) > fabs(mat[lim][i])) lim = j;
        }
        swap(mat[i], mat[lim]);
        if(fabs(mat[i][i]) < eps) continue;
        double temp = mat[i][i];
        for(int j = i; j <= n; j++) {
            mat[i][j] /= temp;
        }
        for(int j = 0; j < n; j++) {
            if(i == j) continue;
            double rate = mat[j][i];
            for(int k = i; k <= n; k++) {
                mat[j][k] -= mat[i][k] * rate;
            }
        }
    }
}

void solve() {
	// bfs 去除无关点
	if(!bfs(sx, sy)) {
		cout << -1 << endl;
		return;
	}
	for(int i = 1; i <= n; i++) {
		for(int j = 1; j <= m; j++) {
			if(!vis[i][j]) continue;
			int cur = ver(i, j), cnt = 0;
			if(g[i][j] == '$') {
				mat[cur][cur] = 1;
				mat[cur][n * m] = 0;
				continue;
			}
			// 由该点向周围能够到达的点建立期望方程
			for(int k = 0; k < 4; k++) {
				int nx = i + dx[k], ny = j + dy[k];
				if(nx < 1 || ny < 1 || nx > n || ny > m || !vis[nx][ny]) continue;
				cnt++;
				mat[cur][ver(nx, ny)] = -1;
			}
			mat[cur][cur] = mat[cur][n * m] = cnt;
		}
	}
	// 高斯消元
	Guass(n * m);
	cout << fixed << setprecision(6) << mat[ver(sx, sy)][n * m] << endl;
}

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	while(cin >> n >> m) {
		memset(vis, 0, sizeof vis);
		memset(mat, 0, sizeof mat);
		for(int i = 1; i <= n; i++) {
			for(int j = 1; j <= m; j++) {
				cin >> g[i][j];
				if(g[i][j] == '@') sx = i, sy = j;
			}
		}
		solve();
	}
	return 0;
}
```

---
### 练习题

- [Circles of Waiting ](https://vjudge.net/problem/CodeForces-963E#author=DeepSeek_zh)

**题目大意**：在二维平面上，起点为 $(0,0)$ 坐标，每次移动都有 $p_1$ 概率移动到 $(x-1,y)$，$p_2$ 概率移动到 $(x,y-1)$，$p_3$ 概率移动到 $(x+1,y)$，$p_4$ 概率移动到 $(x,y+1)$，求移动到与原点距离首次超过 $R(0\le R\le50)$ 的期望时间（即满足 $x^2+y^2\gt R^2$）。

**题目思路**：
- 从左到右，从上到下依次将圆内的点映射到点集里，取从第 $i$ 个点开始运动到首次与原点的距离超过 $R$  的期望时间为 $\text{dp}[i]$，定义 $idx(x,y)$ 为坐标 $(x,y)$ 在点集内的序号，考虑平均首访时间方程组有 $$\begin{aligned}\text{dp}[i]&=p_1\cdot \text{dp}[idx(x-1,y)]+p_2\cdot \text{dp}[idx(x,y-1)]\\&+p_3\cdot \text{dp}[idx(x+1,y)]+p_4\cdot \text{dp}[(x,y+1)]+1\end{aligned}$$
- 依据该方程建立方程组再进行高斯消元，复杂度为 $O(R^6)$，考虑方程组特点，由于对于每个点，只向周围四个点建立联系，这五个点的索引差最大为 $2R$。即对于第 $i$ 行和第 $i$ 列，只会在 $[i-r,i+r]$ 上有信息，对于这种带状方程组，要采用带状高斯消元法解决，复杂度为 $O(R^5)$。

```cpp
#include<bits/stdc++.h>
#define endl '\n'
#define x first
#define y second

using namespace std;
using ll = long long;
using pii = pair<int, int>;

const int mod = 1e9 + 7;

int r, a1, a2, a3, a4, n, tot, s;
int flag[120][120], mat_mod[8010][8010];
pii idx[8010];

ll qpow(ll x, ll n) {
	ll res = 1;
	while(n) {
		if(n & 1) res = res * x % mod;
		x = x * x % mod;
		n >>= 1;
	}
	return res;
}

void Guass_mod(int n, ll p1, ll p2, ll p3, ll p4) {
	int len = 2 * r;
    for(int i = 1; i <= n; i++) {
    	// 找到主元行
        for(int j = i; j <= n; j++) {
            if(mat_mod[j][i]) {
                swap(mat_mod[i], mat_mod[j]);
                break;
            }
        }
        ll inv = qpow(mat_mod[i][i], mod - 2);
        int end_row = min(n, i + len);
        // 带状消元
        for(int j = i + 1; j <= end_row; j++) {
        	if(!mat_mod[j][i]) continue;
        	int factor = 1ll * mat_mod[j][i] * inv % mod;
        	int end_col = min(n, j + len);
        	for(int k = max(1, j - len); k <= end_col; k++) {
        		mat_mod[j][k] = (mat_mod[j][k] - 1ll * factor * mat_mod[i][k] % mod + mod) % mod;
        	}
        	// 常数项单独消元
        	mat_mod[j][n + 1] = (mat_mod[j][n + 1] - 1ll * factor * mat_mod[i][n + 1] % mod + mod) % mod;
        }
    }
    // 对上三角形矩阵回代处理
    for(int i = n; i >= 1; i--) {
    	int end_col = min(n, i + len);
    	for(int j = i + 1; j <= end_col; j++) {
    		mat_mod[i][n + 1] = (mat_mod[i][n + 1] - 1ll * mat_mod[i][j] * mat_mod[j][n + 1] % mod + mod) % mod;
    	}
    	ll inv = qpow(mat_mod[i][i], mod - 2);
    	mat_mod[i][n + 1] = 1ll * mat_mod[i][n + 1] * inv % mod;
    }
}

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	cin >> r >> a1 >> a2 >> a3 >> a4;
	if(r == 0) {
		cout << 1 << endl;
		return 0;
	}
	s = a1 + a2 + a3 + a4;
	ll den = qpow(s, mod - 2);
	ll p1 = den * a1 % mod, p2 = den * a2 % mod, p3 = den * a3 % mod, p4 = den * a4 % mod;
	for(int i = -r; i <= r; i++) {
		for(int j = -r; j <= r; j++) {
			if(i * i + j * j <= r * r) {
				flag[i + r][j + r] = ++tot;
				idx[tot].x = i, idx[tot].y = j;
			}
		}
	}
	for(int i = 1; i <= tot; i++) {
		mat_mod[i][i] = mat_mod[i][tot + 1] = 1;
    	auto [x, y] = idx[i];
    	x += r, y += r;
    	if(flag[x - 1][y]) mat_mod[i][flag[x - 1][y]] = mod - p1;
    	if(flag[x][y - 1]) mat_mod[i][flag[x][y - 1]] = mod - p2;
    	if(flag[x + 1][y]) mat_mod[i][flag[x + 1][y]] = mod - p3;
    	if(flag[x][y + 1]) mat_mod[i][flag[x][y + 1]] = mod - p4;
	}
	Guass_mod(tot, p1, p2, p3, p4);
	cout << mat_mod[flag[r][r]][tot + 1] << endl;
	return 0;
}
```


- [Broken robot](https://vjudge.net/problem/CodeForces-24D#author=DeepSeek_zh)

**题目大意**：给定 $n$ 行 $m$ 列的网格图 $(1\le n,m\le1000)$，机器人初始坐标为 $(sx,sy)$，目标是要到达最底层的第 $n$ 行。每次行动，机器人可以选择停留在原地，向左、向右、或者向下移动（位于最左侧不能左移，位于最右侧不能右移），每一个选择的概率都是相等的，求解到达最底层的期望步数。

**题目思路**：
- 取 $dp[i][j]$ 表示从 $(i,j)$ 开始移动到达最底层的期望步数，由平均首访时间方程组有 $$\text{dp}[i][j]=\begin{cases}\frac{1}{4}\cdot \text{dp}[i][j+1]+\frac{1}{4}\cdot \text{dp}[i][j-1]+\frac{1}{4}\cdot \text{dp}[i][j]+\frac{1}{4}\cdot \text{dp}[i+1][j]+1&1\lt j\lt m\\\frac{1}{3}\cdot \text{dp}[i][j+1]+\frac{1}{3}\cdot \text{dp}[i][j]+\frac{1}{3}\cdot \text{dp}[i+1][j]+1 &j=1\\\frac{1}{3}\cdot \text{dp}[i][j-1]+\frac{1}{3}\cdot \text{dp}[i][j]+\frac{1}{3}\cdot \text{dp}[i+1][j]+1&j=m\end{cases}$$变形得 $$\begin{cases}3\cdot \text{dp}[i][j]-\text{dp}[i][j+1]-\text{dp}[i][j-1]=\text{dp}[i+1][j]+4&1\lt j\lt m\\2\cdot \text{dp}[i][j]-\text{dp}[i][j+1]=\text{dp}[i+1][j]+3 &j=1\\2\cdot \text{dp}[i][j]-\text{dp}[i][j-1]=\text{dp}[i+1][j]+3 &j=m\end{cases}$$
- 很明显是多后效性的方程组，但如果直接套用高斯消元，时间复杂度为 $O(m^3)$。观察方程组特点，将 $\text{dp}[i+1][j]$ 视作已经计算好的常数，那么其只与左右两项相关，这就是带状方程组的特殊情况，三对角矩阵，采用追赶法（Thomas 算法）可在 $O(m)$ 内求解，总时间复杂度为 $O(n\cdot m)$。

```cpp
#include<bits/stdc++.h>
#define endl '\n'
using namespace std;

const int N = 1e3 + 10;
double dp[N][N], mat[N][N];

int n, m, sx, sy;

void solve() {
	cin >> n >> m >> sx >> sy;
	// 特判
	if(m == 1) {
		cout << fixed << setprecision(10) << (n - sx) * 2 << endl;
		return;
	}
	for(int i = n - 1; i >= sx; i--) {
		memset(mat, 0, sizeof mat);
		// 构造方程组
		mat[1][1] = 2, mat[1][2] = -1, mat[1][m + 1] = 3 + dp[i + 1][1];
		mat[m][m] = 2, mat[m][m - 1] = -1, mat[m][m + 1] = 3 + dp[i + 1][m];
		for(int j = 2; j < m; j++) {
			mat[j][j] = 3, mat[j][j + 1] = mat[j][j - 1] = -1, mat[j][m + 1] = 4 + dp[i + 1][j];
		}
		// 三对角矩阵高斯消元
		for(int j = 2; j <= m; j++) {
			double factor = mat[j][j - 1] / mat[j - 1][j - 1];
			mat[j][j - 1] = 0;
			mat[j][j] -= factor * mat[j - 1][j], mat[j][m + 1] -= factor * mat[j - 1][m + 1];
		}
		// 回代
		mat[m][m + 1] /= mat[m][m], mat[m][m] = 1, dp[i][m] = mat[m][m + 1];
		for(int j = m - 1; j >= 1; j--) {
			double factor = mat[j][j + 1] / mat[j + 1][j + 1];
			mat[j][j + 1] = 0;
			mat[j][m + 1] = (mat[j][m + 1] - factor * mat[j + 1][m + 1]) / mat[j][j];
			dp[i][j] = mat[j][m + 1], mat[j][j] = 1;
		}
	}
	cout << fixed << setprecision(10) << dp[sx][sy] << endl;
}

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	solve();
	return 0;
}
```


- [游走](https://www.luogu.com.cn/problem/P3232)

**题目大意**：给定 $n$ 个点 $m$ 条边的无向图，固定起点为 $1$，终点为 $n$（到达终点后停止移动），每次移动时都会随机等概率选择当前所在顶点的某条边，沿着这条边走向下一个顶点。现给每一条边标上唯一的从 $1$ 到 $m$ 的标号，求解所有边的期望访问次数乘以其标号的最小和是多少。

**题目思路**：
- 当给期望访问次数越多的分配越小的标号时，总和显然最少。现考虑如何计算每条边的期望访问次数，对于边的两端点 $u,v$，倘若其期望访问次数为 $E(u),E(v)$，由于对于 $u,v$ 分别有 $\frac{1}{\deg(u)},\frac{1}{\deg(v)}$ 的概率通过该边，故该边的期望访问次数为 $\frac{E(u)}{\deg(u)}+\frac{E(v)}{\deg(v)}$。
- 现考虑如何计算每个点的期望访问次数，考虑平均访问时间方程组有 $$E(u)=\begin{cases}0&u=n\\1+\sum\limits_{v\rightarrow u}\frac{E(v)}{\deg(v)}&u=1\\\sum\limits_{v\rightarrow u}\frac{E(v)}{\deg(v)}&u\neq1,n\end{cases}$$建立好具有后效性的方程组后采用高斯消元即可。

```cpp
#include<bits/stdc++.h>
#define endl '\n'
#define eb emplace_back
#define pb push_back
#define rall(x) x.rbegin(), x.rend()

using namespace std;
using ll = long long;
using vvi = vector<vector<int>>;
using pii = pair<int, int>;

const int p = 998244353;
const int N = 510;
const double eps = 1e-9;
int n, m;
vvi g(N);
double mat[N][N];
vector<pii> edge;

void Guass(int n) {
    for(int i = 1; i <= n; i++) {
        int lim = i;
        for(int j = i; j <= n; j++) {
            if(fabs(mat[j][i]) > fabs(mat[lim][i])) lim = j;
        }
        swap(mat[i], mat[lim]);
        if(fabs(mat[i][i]) < eps) continue;
        double temp = mat[i][i];
        for(int j = i; j <= n + 1; j++) {
            mat[i][j] /= temp;
        }
        for(int j = 1; j <= n; j++) {
            if(i == j) continue;
            double rate = mat[j][i];
            for(int k = i; k <= n + 1; k++) {
                mat[j][k] -= mat[i][k] * rate;
            }
        }
    }
}

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	cin >> n >> m;
	for(int i = 0; i < m; i++) {
		int u, v; cin >> u >> v;
		g[u].pb(v); g[v].pb(u);
		edge.eb(u, v);
	}
	// 建立方程组
	mat[1][n] = 1.0;
	for(int u = 1; u < n; u++) {
		mat[u][u] = 1.0;
		for(int v : g[u]) {
			if(v == n) continue;
			mat[u][v] = -1.0 / g[v].size();
		}
	}
	// 高斯消元
	Guass(n - 1);
	// 排序标号
	vector<double> a;
	for(auto [u, v] : edge) {
		double exp = 0;
		if(u != n) exp += mat[u][n] / g[u].size();
		if(v != n) exp += mat[v][n] / g[v].size();
		a.pb(exp);
	}
	sort(rall(a));
	double ans = 0;
	for(int i = 0; i < m; i++) ans += (i + 1) * a[i];
	cout << fixed << setprecision(3) << ans << endl;
	return 0;
}
```


- [Time travel](https://vjudge.net/problem/HDU-4418#author=GPT_zh)

**题目大意**：给定起点 $st$，终点 $ed$，以及整条路的范围 $n$，玩家会在 $0\sim n-1$ 的范围上来回移动（即 $0,1,2,\dots,n-2,n-1,n-2,\dots,1,0,1,2,\dots$）。每次移动，玩家有 $p_i$ 的概率走 $i$ 步（$1\le i\le m$）。给定初始移动方向 $D$，$D=0$ 表示沿着 $x$ 轴正方向移动，$D=1$ 表示沿着 $x$ 轴负方向移动，$D=-1$ 表示 $st$ 为 $0$ 或 $n-1$，求解玩家到达 $ed$ 的期望步数。

**题目思路**：
- 题目中玩家的移动虽然是在数轴上，但是是按照环的方式运动的，破环为链，将数组扩容成 $0,1,2,\dots,n-1,n-2,\dots,1,0$ 的形式，即长度 $len=2(n-1)$，这样 $x$ 对应从 $x$ 点正方向移动，$len-x$ 对应从 $x$ 点反方向移动，移动 $k$ 步后下标为 $(x+k)\mod{len}$。
- 由于题目中存在 $p_i$ 为 $0$ 的情况，先从 $st$ 开始进行 bfs，若 $D=1$，则从 $len-st$ 开始 bfs，将所有能够遍历到的点打上标记，若 $ed$ 和 $len-ed$ 均未被遍历到，则无法到达。
- 对于 bfs 后所有打上标记的点，取 $\text{dp}[i]$ 表示位于 $i$ 的时候到达 $ed$ 的期望步数，根据平均吸收时间方程组有状态转移方程 $$\text{dp}[i]=\begin{cases}0&i=ed,len-ed\\\sum\limits_{k=1}^mp_k\cdot (\text{dp}[(i+k)\%len]+k)&i\neq ed,len-ed\end{cases}$$对于该后效性的方程组进行高斯消元求解即可。

```cpp
#include<stdio.h>
#include<iostream>
#include<algorithm>
#include<string.h>
#include<queue>
#include<math.h>
using namespace std;

const int N = 210;
const double eps = 1e-9;
double mat[N][N], p[N];
int n, m, ed, st, d, len, tot;
int flag[N];

void bfs(int st) {
	flag[st] = tot++;
	queue<int> q; q.push(st);
	while(q.size()) {
		auto pos = q.front(); q.pop();
		for(int i = 1; i <= m; i++) {
			if(fabs(p[i]) < eps) continue;
			int nxt = (pos + i) % len;
			if(flag[nxt] != -1) continue;
			flag[nxt] = tot++;
			q.push(nxt);
		}
	}
}

void Guass(int n) {
    for(int i = 0; i < n; i++) {
        int lim = i;
        for(int j = 0; j < n; j++) {
            if(j < i && fabs(mat[j][j]) >= eps) continue;
            if(fabs(mat[j][i]) > fabs(mat[lim][i])) lim = j;
        }
        swap(mat[i], mat[lim]);
        if(fabs(mat[i][i]) < eps) continue;
        double temp = mat[i][i];
        for(int j = i; j <= n; j++) {
            mat[i][j] /= temp;
        }
        for(int j = 0; j < n; j++) {
            if(i == j) continue;
            double rate = mat[j][i];
            for(int k = i; k <= n; k++) {
                mat[j][k] -= mat[i][k] * rate;
            }
        }
    }
}

void solve() {
	cin >> n >> m >> ed >> st >> d;
	for(int i = 1; i <= m; i++) cin >> p[i], p[i] /= 100;
	if(st == ed) {
		printf("0.00\n");
		return;
	}
	len = 2 * (n - 1);
	if(d == 1) st = len - st;
	// bfs 判定能够到达的状态
	bfs(st);
	if(flag[ed] == -1 && flag[len - ed] == -1) {
		printf("Impossible !\n");
		return;
	}
	// 构建方程组
	for(int i = 0; i < len; i++) {
		if(flag[i] == -1) continue;
		int cur = flag[i];
		if(i == ed || i == len - ed) {
			mat[cur][tot] = 0, mat[cur][cur] = 1;
			continue;
		}
		mat[cur][cur] = 1, mat[cur][tot] = 0;
		for(int j = 1; j <= m; j++) {
			int nxt = (i + j) % len;
			if(flag[nxt] == -1 || fabs(p[j]) < eps) continue;
			mat[cur][flag[nxt]] -= p[j];
			mat[cur][tot] += j * p[j];
		}
	}
	// 高斯消元输出答案
	Guass(tot);
	if(flag[st] != -1) printf("%.2lf\n", mat[flag[st]][tot]);
	else printf("%.2lf\n", mat[flag[len - st]][tot]);
}

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	int t = 1; cin >> t;
	while(t--) {
		tot = 0;
		memset(mat, 0, sizeof mat);
		memset(flag, -1, sizeof flag);
		solve();
	}
	return 0;
}
```