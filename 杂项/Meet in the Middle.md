- Meet in the Middle（中间相遇）：此类问题一遍的解决思路是，将输入数据拆分成两半，每一部分单独处理，最后再合并两部分的结果即可。与分治法的区别在于，分治法通常是递归进行的（多层），而中间相遇法通常只进行一次（或有限次）拆分，目标通常是把时间复杂度从 $O(2^n)$ 优化到 $O(2^{\frac{n}{2}})$ 或从 $O(n^k)$ 优化到 $O(n^{\lceil\frac{k}{2}\rceil})$，即在问题规模减半后可以直接直到如何解决，重点在于状态合并的技巧。

- [Meet in the Middle](https://cses.fi/problemset/task/1628/)

**题目大意**：给定一个长度为 $n(n\le40)$ 的数组，找出是否存在一个子集使其和为目标值 $s$。

**题目思路**：
- 如果 $n$ 减半，则直接遍历即可解决，复杂度为 $O(2^{\frac{n}{2}})$，那么解决方法考虑为将数组分成两半后，分别计算所有子集和，然后合并即可。
- 考虑如何合并，目前问题转化为，在左子集和集合和右自子集和集合中各找一个数 $x,y$，使得 $x+y=s$ 成立，那么对于每一个 $x$，都需要在右侧统计有多少个 $y=s-x$ 与之配对，使用二分法统计即可，时间复杂度为 $(2^\frac{n}{2}\cdot\log{n})$，其中 $\log{n}$ 为足够小可忽略的常数。

```cpp
#include<bits/stdc++.h>
#define endl '\n'
#define pb push_back
#define all(x) x.begin(), x.end()

using namespace std;
using ll = long long;
using vll = vector<ll>;

void search(vll &res, vector<ll> nums) {
	ll n = nums.size(), lim = 1 << n;
	for(int i = 0; i < lim; i++) {
		ll s = 0;
		for(int j = 0; j < n; j++) {
			if(i >> j & 1) s += nums[j];
		}
		res.pb(s);
	}
}

void solve() {
	ll n, s; cin >> n >> s;
	ll n1 = n / 2, n2 = n - n1;
	vll res1, res2, a(n1), b(n2);
	for(ll &x : a) cin >> x;
	for(ll &x : b) cin >> x;
	search(res1, a); search(res2, b);
	sort(all(res2));
	ll ans = 0;
	for(ll x : res1) {
		if(x > s) continue;
		auto l = lower_bound(all(res2), s - x);
		auto r = upper_bound(all(res2), s - x);
		ans += r - l;
	}
	cout << ans << endl;
}

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	solve();
	return 0;
}

```

**题目大意**：给定一个数组 $a_1,a_2,\dots,a_n(n\le 2000)$，统计递增三元组（即对于 $i\lt j\lt k$，满足 $a_i\lt a_j\lt a_k$）的数量。

**题目思路**：
- 在该问题中。将 $n$ 减半思考对于问题解决没有实质性帮助，，而如果将三元组视作一个长度为 $3$ 的子序列，那么考虑将该子序列减半思考。
- 遍历每一个位置 $j\ (2\le j\le n-1)$ 作为三元组的中间项，它是合并左边长度为 $2$ 的子序列以及右边长度为 $2$ 的子序列的关键。对于固定的 $j$，需要统计左边有多少个元素小于 $a_j$，以及右边有多少个元素大于 $a_j$，再根据乘法原理相乘即可，这样暴力枚举就从 $O(n^3)$ 优化到了 $O(n^{\lceil\frac{3}{2}\rceil})=O(n^2)$。

- [Sum of Four Values](https://cses.fi/problemset/task/1642)

**题目大意**：给定 $n\ (n\le1000)$ 个数字的数组，找到数组中 $4$ 个不同位置，使得它们的数字之和为 $x$。

**题目思路**：
- 倘若直接暴力，那么时间复杂度为 $O(n^4)$，考虑 Meet in the Middle 思想，遍历数组的前两个元素，计算它们的和，然后查找剩余元素中是否存在一对数的和等于目标值减去前两个数的和，最后合并，时间复杂度可以降为可行的 $O(n^2)$。

```cpp
#include<bits/stdc++.h>
#define endl '\n'
#define x first
#define y second

using namespace std;
using ll = long long;
using vll = vector<ll>;

void solve() {
	ll n, x; cin >> n >> x;
	unordered_map<ll, pair<int, int>> pos;
	vll a(n + 1);
	for(int i = 1; i <= n; i++) cin >> a[i];
	for(int i = 2; i <= n; i++) {
		for(int j = 1; j < i; j++) {
			ll s = a[i] + a[j];
			pos[s] = {i, j};
		}
		for(int j = i + 2; j <= n; j++) {
			ll s = a[i + 1] + a[j];
			if(pos.find(x - s) != pos.end()) {
				cout << i + 1 << ' ' << j << ' ' <<  pos[x - s].x << ' ' << pos[x - s].y << endl;
				return;
			}
		}
	}
	cout << "IMPOSSIBLE\n";
}

int main() {
	ios::sync_with_stdio(0), cin.tie(0), cout.tie(0);
	solve();
	return 0;
}
```