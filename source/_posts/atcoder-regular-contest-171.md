---
title: AtCoder Regular Contest 171 全场题解
date: 2024-04-05 15:26:16
categories: OI
tags:
	- 题解
	- AtCoder
description: AT ARC171 的翻译和题解 qwq
---

某天 vp 了一下这场比赛，题解拖到了现在（）

赛时做出了前三题，第四题写挂了 ovo

# A

## 题意

有一个 $N \times N$ 的国际象棋棋盘，需要在上面放上 $A$ 个车和 $B$ 个兵，车可以吃同行同列的棋子，兵可以吃在同列且在它上一行的棋子，问是否存在放法，使得这 $A + B$ 个棋子互相不能吃到，有多测。

$1 \le T \le 10^5$，$1 \le N \le 10^4$，$0 \le A,B$，$1 \le A + B \le N ^ 2$。

## 解法

一行一列中最多放一个车，所以车的上界是 $N$。

同时，放了车的行和列不能有兵，所以放了兵的行和列上界都为 $N - A$。

由于兵的限制只和上一行有关，可以贪心地对于每一行，把能放兵的位置放满。此时下一行就放不了了，所以放兵的行还有一个上界 $\lfloor \frac{N + 1}{2} \rfloor$。

综合起来，兵的上界就为 $\min(N - A,\lfloor \frac {N + 1}{2} \rfloor) \times (N - A)$。

考虑构造。发现任意两列是可以任意交换的，只考虑行。放兵的行的上一列放不了棋子，考虑尽量让放兵的行和放车的行间隔起来。具体地，先把车放在奇数行，放完再从上到下放偶数行，再把兵放在空出来的行。

考虑证明。若车的数量不足以占领所有奇数行，那么放兵相当于没有限制地放，可以达到上界 $\lfloor \frac{N + 1}{2} \rfloor$。若车的数量够占领奇数行，那么剩下的行都可以放车，可以达到上界 $N - A$。

# B

## 题意

对于一个 $[1,N]$ 的排列 $P$，定义 $F(P)$ 为这个排列经过以下操作后生成的排列 $B$：

- 排列 $B$ 初始为 $(1,2,\cdots,N)$。
如果有一个最小的数 $i$ 满足 $B_i < P_{B_i}$，则 $B_i \leftarrow P_{B_i}$。

给定一个长度为 $N$ 的 $A$，问有多少个排列 $P$ 满足 $F(P) = A$。

$1 \le N \le 2 \times 10 ^ 5$。

## 解法

先看看操作是什么东西。看到排列，想到建图（？）。建一个有 $N$ 个点的有向图，$i$ 向 $P_i$ 连边。则这个图被分为若干个环。

初始时，每个点上有一个东西。题面中所说的操作，相当于找一个最小编号的点，使得它的编号小于它指向的点，把在它上的东西沿边移动。发现这个“最小编号”其实没什么用，每个东西最终停住的地方，就是它到达的第一个指向的点编号大于自己的点。它经过的点编号单调递减。

将给定的 $A$ 中值一样的位置拿出来，除最后一个点外，它们原本连的边一定指向下一个点。由于图是若干个环，每一种值的编号最大的点指向的一定为一个小于自己、且为某种值的编号最小的点。同时，编号最大的点必然为这个值。这样，原问题转化为有多少种将各种值的编号最小（黑）、最大（白）点两两连边的方式。

由于每个白点只能连向比自己小的点，考虑从小到大枚举白点，第 $i$ 个点有 $(比自己小的黑点数 - i + 1)$ 种选法。乘法原理算出来就是答案。

代码：

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
using namespace std;

const int mod = 998244353;

int n;
vector <int> v[200010];
int a[200010],use[200010],pd[200010];
int ans,res = 1;

int main()
{
	scanf("%d",&n);
	for (int i = 1;i <= n;i ++) scanf("%d",a + i),v[a[i]].push_back(i);
	for (int i = 1;i <= n;i ++)
	{
		if (a[i] != i && v[i].size())
		{
			printf("0\n");
			return 0;
		}
		if (!v[i].size()) continue;
		for (int j : v[i])
			if (j > i)
			{
				printf("0\n");
				return 0;
			}
		pd[i] = 1;
		use[v[i][0]] = 1;
	}
	for (int i = 1;i <= n;i ++)
	{
		if (use[i]) ans ++;
		if (pd[i]) res = 1ll * res * ans % mod,ans --;
	}
	printf("%d\n",res);
}
```

# C

## 题意

给定一棵有 $N$ 个节点的树，编号为 $1$ 到 $N$。初始时，每一个点 $i$ 上有一个编号为 $i$ 的棋子。

你可以进行任意多次（包括 $0$ 次）如下操作：

- 选择一条边，交换边的两端节点上的棋子，然后把这条边删掉。

问最后树上的棋子有几种放法。

$2 \le N \le 3000$。

## 解法

发现子树内的棋子的方案数比较独立，只有子树的根和父亲交换可能影响到这棵子树。考虑树上 dp。

手动模拟一下，会发现一棵树最终的形态由操作的相对顺序（指祖先和儿子的关系（？））决定。因此，在 dp 的时候要记录操作数，便于算操作的相对顺序个数。

dp 的转移就是将两棵树的方案合并起来的过程。状态设为 $f[i][j]$，表示以 $i$ 为根的子树中发生了 $j$ 次交换操作的放法数。如果两棵树的根不交换，方案数就是直接乘起来；否则枚举这次交换操作要插进两个操作序列的哪里，再乘起来更新答案。将根为 $v$ （$v \in son_u$）的子树与根为 $u$ 的子树合并的转移式：

$$
f'[u][k] = \sum \limits_{i + j = k} f[u][i] \times f[v][j] + \sum \limits _{i + j = k - 1} f[u][i] \times f[v][j] \times (i + 1) \times (j + 1)
$$

直接 dp，时间复杂度 $O(n ^ 2)$。

# D

## 题意

给定非负整数 $P$ 和 $B$，对于一个序列 $X$，定义 $\operatorname{hash}(X) = (\sum \limits _{i=1} ^{|X|} X_i B^{|X| - i}) \mod p$。给出 $M$ 个区间 $[L_i,R_i]$，问是否可能构造序列 $A=(A_1,A_2,\cdots,A_N)$，使得对于所有 $1 \le i \le M$，$\operatorname{hash}(A_{L_i},A_{L_i + 1},\cdots,A_{R_i}) \ne 0$。

$2 \le P \le 10^9$，$1 \le B < P$，$1 \le N \le 16$，$1 \le M \le \frac{N(N + 1)}{2}$，区间互不相同。

## 解法

定义 $F(i)=\operatorname{hash}(A_i,A_{i + 1},\cdots,A_{|X|})$，根据哈希函数定义可得 $\operatorname{hash}({A_x | x \in [L_i,R_i]}) = \frac{F(L_i) - F(R_i + 1)}{B^{|X| - R_i}} \mod P$。由于 $B^{|X| - R_i} \ne 0$，要使这个值不为 $0$，则 $F(L_i) \ne F(R_i + 1)$。题目给的 $M$ 条限制，相当于在图上连了 $M$ 条边，边的两端填的数不一样。而 $0 \le F(i) < P$，所以原题有解等价于这个图上没有 $(P + 1)$ 阶完全图。

发现 $N \le 16$，考虑用状压 dp 解决。定义 $f[i]$ 为二进制数 $i$ 代表的子图中最大的完全图有几个点，转移方程为

$$
f[i] = \min \limits _{j \oplus k = i} (f[j] + f[k])
$$

注意特判 $i$ 代表的子图中没有边的情况。时间复杂度 $O(2 ^ N N ^ 2)$。

代码：

```cpp
#include <iostream>
#include <cstdio>
using namespace std;

int P,B,n,m;
int pd[20][20],f[200010];

int main()
{
	scanf("%d%d%d%d",&P,&B,&n,&m);
	if (P > n)
	{
		printf("Yes\n");
		return 0;
	}
	for (int i = 1;i <= m;i ++)
	{
		int x,y;
		scanf("%d%d",&x,&y);
		y ++;
		pd[x][y] = pd[y][x] = 1;
	}
	n ++;
	for (int i = 1;i < (1 << n);i ++)
	{
		bool fl = true;
		for (int j = 1;j <= n;j ++)
		{
			if (!(i >> (j - 1) & 1)) continue;
			for (int k = j + 1;k <= n;k ++)
			{
				if (!(i >> (k - 1) & 1)) continue;
				fl &= (!pd[j][k]);
			}
		}
		if (fl)
		{
			f[i] = 1;
			continue;
		}
		f[i] = n;
		for (int j = (i - 1) & i;j;j = (j - 1) & i)
			f[i] = min(f[i],f[j] + f[i ^ j]);
		if (f[i] >= P + 1)
		{
			printf("No\n");
			return 0;
		}
	}
	printf("Yes\n");
}
```

# E

## 题意

给定一个 $N \times N$ 的网格，$(i,j)$ 表示第 $i$ 行第 $j$ 列的格子。现在需要在网格上摆放 $1$ 个黑色石头和 $M$ 个白色石头，且每行每列最多只有一个白色石头，黑色石头在 $(A,B)$ 上。接下来，要进行以下操作，直到不可进行为止：

 - 设黑色石头位于 $(i,j)$。若上下左右四个方向上有至少一个白色石头，选择其中一个，将黑色石头放到白色石头后面（白色石头所对方向）一格，并拿走这个白色石头。

如果操作结束后，黑色石头重新位于 $(A,B)$，且白色石头全部被拿走，则操作成功。问有多少种白色石头的摆放方式，使得操作成功。

$2 \le M \le N \le 2 \times 10^5$。

## 解法

黑色石头走的路线和白色石头的摆放方式是一一对应的，证明略，可以手动模拟感性理解一下（）

由于黑色石头的路线是横一下竖一下的，总步数应为偶数。每一次黑色石头进入一行或者一列，相当于把两行或两列标记上不能再次进入，因为这两行或两列的白色石头被拿走了。考虑确定白色石头的位置，每个石头占领两行或两列，且互不相同。行和列可以分开考虑，再乘起来。

观察行的情况，总共 $K = \lfloor \frac {M}{2} \rfloor$ 个石头占领两行，其中一定有一个石头，占领的两行包括第 $A$ 行。分两种情况考虑，一种是第 $A - 1$ 和 $A$ 行，一种是 $A$ 和 $A + 1$ 行。先确定占领第 $A$ 行的石头是哪个，再算剩下的占领哪些行，两种情况的方案数分别如下：

$$
\sum \limits _i i{ {A - 2 - i} \choose {i} } { {N - A - (K - i - 1)} \choose {K - i - 1} }
$$

$$
\sum \limits _i (K - i - 1){ {A - 1 - i} \choose i} { {N - A - 1 - (K - i - 1)} \choose {K - i - 1} }
$$

列的情况同理。最后再乘上行选择列的 $(M - 2)!$。时间复杂度线性。

# F

## 题意

定义一个字符串 $T$ 是**好的**，当且仅当它满足下列条件：

- 存在非空字符串 $A$ 和 $B$，使得 $A + B = T$，且 $A + \operatorname{rev}(B)$ 和 $\operatorname{rev}(A) + B$ 均为回文串。

$A + B$ 表示将 $A$ 和 $B$ 拼接得到的字符串，$\operatorname{rev}(A)$ 表示将 $A$ 首尾反转得到的字符串。

给定一个长为 $N$，由小写英文字母和 `?` 组成的字符串 $S$，问有多少种将 `?` 改成小写字母的方法，使得 $S$ 是**好的**字符串。

$2 \le N \le 5 \times 10^4$。

## 解法

（这里空着一段官方题解的说明，从最后一段的 `This part is also somewhat complex, but we will omit the details.` 开始 qwq）

官方题解讲得十分详细，但是实现没有讲。

题解中得到了一个结论，就是把 $S$ 划分成 $(A,B)$ 分为两种情况：

1. $S = T^n$，其中 $T$ 为 primitive root。此时有 $n - 1$ 种分法，把 $S$ 分为两个回文串。

2. $S$ 不为回文串，那么 $A$ 和 $B$ 形如一堆 $T$ 和 $T$ 的一半拼在一起。此时分法是唯一的。

考虑怎么用这个结论和容斥求出原问题的解。

对于第一种情况，直接枚举 $T$ 的长度，把相对应位置并起来求方案数就行了。注意 $T$ 要求是 primitive root，要把不是的情况容斥掉，即去掉 $|T| \mod |T'| = 0$ 的情况。

对于第二种情况，也是枚举 $T$ 的长度，同时枚举 $A$ 和 $B$ 的断点。把 $T$ 看成 $X + \operatorname{rev} (X)$，相当于 $X$ 正着接一个，反着接一个，但在断点左右都是正着的。可以先正着和反着预处理正反正反地接到当前节点的方案数，枚举断点的时候左右一拼就行了。

想一想这么做会多算哪些东西。一种是 $X$ 也为回文串的情况，需要减掉。另一种是 $X$ 不为回文串，但 $T$ 为回文串的情况，如 `abbaabbaabba`。此时，通过观察可以发现，在同一断点上， $T$ 比较长的情况包含了比较短的情况。所以，在实际处理中，只需要在同一断点上保留 $|T|$ 最大时算出来的答案就行了。

代码：

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
using namespace std;

const int mod = 998244353;

int n;
char s[50010],t[50010];
int P[50010];
int pre[50010],suf[50010];
vector <char> prestr[50010],sufstr[50010];
int tot,Ans[50010];

int main()
{
	scanf("%d",&n);
	scanf("%s",s + 1);
	for (int i = 1;i <= n;i ++)
	{
		if (n % i != 0) continue;
		// is a palindrome
		bool fl = true;
		for (int j = 1;j <= i;j ++) t[j] = s[j];
		for (int j = 2;j <= n / i;j ++)
		{
			int l = (j - 1) * i + 1,r = j * i;
			for (int k = l;k <= r;k ++)
				if (t[k - l + 1] == '?' && s[k] != '?')
					t[k - l + 1] = s[k];
				else if (t[k - l + 1] != '?' && s[k] != '?' && t[k - l + 1] != s[k])
					fl = false;
		}
		for (int j = 1;j < i - j + 1;j ++)
			if (t[j] != t[i - j + 1] && t[j] != '?' && t[i - j + 1] != '?')
				fl = false;
		if (fl && i < n)
		{
			int ans = 1;
			for (int j = 1;j <= i - j + 1;j ++)
				if (t[j] == '?' && t[i - j + 1] == '?') ans = 26ll * ans % mod;
			P[i] = ans;
		}
		// is not a palindrome
		if (i * 2 > n || n / i % 2 != 0 || i == 1) continue;
		int tmp = n / i;
		for (int j = 1;j <= i;j ++) t[j] = '?';
		pre[0] = 1;
		for (int j = 1;j <= tmp;j ++)
		{
			int l = (j - 1) * i + 1,r = j * i;
			prestr[j].clear();
			if ((j + 1) & 1)
			{
				pre[j] = 1;
				if (!pre[j - 1]) pre[j] = 0;
				for (int k = 1;k <= i;k ++)
					if (t[k] == '?' && s[r - k + 1] != '?')
						t[k] = s[r - k + 1];
					else if (t[k] != '?' && s[r - k + 1] != '?' && t[k] != s[r - k + 1])
						pre[j] = 0;
			}
			else
			{
				pre[j] = 1;
				if (!pre[j - 1]) pre[j] = 0;
				for (int k = 1;k <= i;k ++)
					if (t[k] == '?' && s[l + k - 1] != '?')
						t[k] = s[l + k - 1];
					else if (t[k] != '?'&& s[l + k - 1] != '?' && t[k] != s[l + k - 1])
						pre[j] = 0;
			}
			for (int k = 1;k <= i;k ++) prestr[j].push_back(t[k]);
		}
		for (int j = 1;j <= i;j ++) t[j] = '?';
		suf[tmp + 1] = 1;
		for (int j = tmp;j >= 1;j --)
		{
			int l = (j - 1) * i + 1,r = j * i;
			sufstr[j].clear();
			if ((tmp - j) & 1)
			{
				suf[j] = 1;
				if (!suf[j + 1]) suf[j] = 0;
				for (int k = 1;k <= i;k ++)
					if (t[k] == '?' && s[r - k + 1] != '?')
						t[k] = s[r - k + 1];
					else if (t[k] != '?' && s[r - k + 1] != '?' && t[k] != s[r - k + 1])
						suf[j] = 0;
			}
			else
			{
				suf[j] = 1;
				if (!suf[j + 1]) suf[j] = 0;
				for (int k = 1;k <= i;k ++)
					if (t[k] == '?' && s[l + k - 1] != '?')
						t[k] = s[l + k - 1];
					else if (t[k] != '?'&& s[l + k - 1] != '?' && t[k] != s[l + k - 1])
						suf[j] = 0;
			}
			for (int k = 1;k <= i;k ++) sufstr[j].push_back(t[k]);
		}
		for (int j = 1;j < tmp;j += 2)
		{
			if (!pre[j] || !suf[j + 1]) continue;
			int ans = 1;
			for (int k = 0;k < i;k ++)
				if (prestr[j][k] == '?' && sufstr[j + 1][k] == '?') ans = 26ll * ans % mod,t[k + 1] = '?';
				else if (prestr[j][k] != '?' && sufstr[j + 1][k] != '?' && prestr[j][k] != sufstr[j + 1][k])
					ans = 0;
				else if (prestr[j][k] != '?') t[k + 1] = prestr[j][k];
				else t[k + 1] = sufstr[j + 1][k];
			if (ans)
			{
				int res = 1;
				for (int k = 1;k <= i - k + 1;k ++)
					if (t[k] == '?' && t[i - k + 1] == '?')
						res = 26ll * res % mod;
					else if (t[k] != '?' && t[i - k + 1] != '?' && t[k] != t[i - k + 1])
						res = 0;
				ans = (ans - res + mod) % mod;
			}
			Ans[j * i] = ans;
		}
	}
	for (int i = 1;i < n;i ++)
		if (n % i == 0 && P[i])
		{
			for (int j = 1;j < i;j ++)
				if (i % j == 0) P[i] = (P[i] - P[j] + mod) % mod;
			tot = (tot + P[i]) % mod;
		}
	for (int i = 1;i <= n;i ++) tot = (tot + Ans[i]) % mod;
	printf("%d\n",tot);
}
```

-The End-