---
title: DP 套 DP 学习笔记
date: 2024-04-16 16:04:23
tags:
  - 笔记
  - DP
categories: OI
description: DP 套 DP 的学习笔记~
---

努力更新 x 2（

# 简介

DP 套 DP 顾名思义，就是在一层 DP（或者自动机）外面再套一层，把里面的 DP（或自动机）的**结果**作为外层 DP 的**状态**，从而更方便地转移。

这么说好像很抽象，我也不知道自己在说什么，还是看题目吧 qwq

# 例题：[TJOI2018] 游园会

题目链接：<https://www.luogu.com.cn/problem/P4590>

题目要算的字符串总共有三种限制：长度为 $N$，不出现子串 `NOI`，以及最难搞的和奖章串的最长公共子序列长为 $i$。而且对于每一个 $i$ 都要算一次，如果直接算复杂度就爆炸了。

考虑 DP，状态 $f_{i,j,k}$ 表示现在构造到第 $i$ 位，与兑奖串的最长公共子序列长为 $j$，当前后缀出现了 `NOI` 的前 $k$ 位时的兑奖串个数，是一个计数 DP。这样 DP 看起来很对，但是有一个地方有问题：维护 $j$ 的时候，没有维护当前扫到了奖章串的哪一位，导致我们不知道现在构造的第 $i$ 位应该和奖章串的哪一个字符比较。于是状态应该是 $f_{i,(g_1,g_2,\cdots,g_K),k}$，其中 $i$ 和 $k$ 的含义同上，$K$ 是奖章串长度，中间的 $(g_1,g_2,\cdots,g_K)$ 代表当前扫到奖章串的第 $1,2,\cdots,K$ 位时，最长公共子序列长是多少。在转移的时候，枚举第 $i$ 位填什么，$g$ 数组就是一个单独的 DP，也就是**内层 DP**。而 $f$ 数组的转移就是**外层 DP**。

但是这样的状态根本存不下。观察一下 $g$ 数组有什么性质，发现它的差分数组全是 $0$ 和 $1$，且 $K$ 很小，可以状压。这样状态数就变成了 $O(NK2^K)$。转移的时候直接枚举 $i$、状压后的差分数组、 $k$ 和接下来填的字母，时间复杂度 $O(NK2^K)$。

代码：

```cpp
#include <iostream>
#include <cstdio>
using namespace std;

const int mod = 1e9 + 7;

int n,K;
char s[1010];
int a[1010];
int g[20],gg[20];
int f[2][33000][3],ans[30];

int main()
{
	scanf("%d%d",&n,&K);
	scanf("%s",s + 1);
	for (int i = 1;i <= K;i ++)
		a[i] = (s[i] == 'N' ? 0 : s[i] == 'O' ? 1 : 2);
	f[0][0][0] = 1;
	int p = 1;
	for (int i = 1;i <= n;i ++,p ^= 1)
	{
		for (int j = 0;j < (1 << K);j ++)
			for (int k = 0;k <= 2;k ++)
				f[p][j][k] = 0;
		for (int j = 0;j < (1 << K);j ++)
		{
			for (int k = 1;k <= K;k ++)
				g[k] = g[k - 1] + (j >> (k - 1) & 1);
			for (int k = 0;k <= 2;k ++)
			{
				for (int w = 1;w <= K;w ++)
					gg[w] = max(max(gg[w - 1],g[w]),g[w - 1] + (k == a[w]));
				int tmp = 0;
				for (int w = K;w >= 1;w --) tmp = (tmp << 1 | (gg[w] - gg[w - 1]));
				// 以上是内层 DP 以及把算出来的新 g 数组状压
				for (int nw = 0;nw <= 2;nw ++)
				{
					if (nw == 2 && k == 2) break;
					(f[p][tmp][(k == nw ? nw + 1 : (!k ? 1 : 0))] += f[p ^ 1][j][nw]) %= mod;
					// 外层 DP 的转移
				}
			}
		}
	}
	p ^= 1;
	for (int i = 0;i < (1 << K);i ++)
	{
		for (int j = 1;j <= K;j ++)
			g[j] = g[j - 1] + (i >> (j - 1) & 1);
		(ans[g[K]] += ((f[p][i][0] + f[p][i][1]) % mod + f[p][i][2]) % mod) %= mod;
	}
	for (int i = 0;i <= K;i ++) printf("%d\n",ans[i]);
}
```

可以发现，DP 套 DP 这个名字确实很形象。

# 其他题目

##  [ZJOI2019] 麻将

题目链接：<https://www.luogu.com.cn/problem/P5279>

这道题的内层 DP 是个自动机 ~~，我还没有搞懂，先放在这里（~~

## [SDOI/SXOI2022] 小 N 的独立集

题目链接：<https://www.luogu.com.cn/problem/P8352>

像上面的例题一样，可以搞一个计数 DP，$f_{i,j,k}$ 代表以 $i$ 为根的子树中，选 $i$ 和不选 $i$ 的答案分别是 $j$ 和 $k$ 的数量。状态和转移的时间复杂度还是一如既往地爆炸。观察一下 $j$ 和 $k$ 之间的关系，发现如果 $j \le k$，那 $j$ 没有一点用；否则 $j - k \le K$（这里的 $K$ 就是原题面的 $k$），不然的话可以通过从选 $i$ 的最优方案中去掉 $i$ 创造出一种新的、比当前最优的不选 $i$ 的方案更优的方案，产生矛盾。这样就可以换一种状态表示，$f_{i,j,k}$ 表示以 $i$ 为根的子树中，不选 $i$ 和选不选 $i$ 都行（也就是上面 $j$ 和 $k$ 的较大值）的答案分别是 $j$ 和 $k$。由于 $k \ge j$ 且 $0\le k - j \le K$，可以把状态写成 $f_{i,j,k - j}$，状态数变成 $O(n^2K^2)$。根据[树上背包](https://oi-wiki.org/dp/tree/#%E6%A0%91%E4%B8%8A%E8%83%8C%E5%8C%85)的时间复杂度分析，总的时间复杂度是 $O(n^2K^4)$，因为还要枚举每个点填什么。

代码：

```cpp
#include <iostream>
#include <cstdio>
using namespace std;

const int mod = 1e9 + 7;

int n,K;
int to[2010],nxt[2010],head[1010],cnt;
int siz[1010],f[1010][5010][7],ff[1010][5010][7];
int ans[5010];

void add(int x,int y)
{
	to[++ cnt] = y;
	nxt[cnt] = head[x];
	head[x] = cnt;
}

void dfs(int x,int la)
{
	siz[x] = 1;
	for (int i = 1;i <= K;i ++) f[x][0][i] = 1;
	for (int i = head[x];i;i = nxt[i])
	{
		int u = to[i];
		if (u == la) continue;
		dfs(u,x);
		for (int j = 0;j <= siz[x] * K;j ++)
			for (int k = 0;k <= K;k ++)
			{
				if (!f[x][j][k]) continue;
				int valx1 = j,valx2 = j + k;
				for (int l = 0;l <= siz[u] * K;l ++)
					for (int w = 0;w <= K;w ++)
					{
						if (!f[u][l][w]) continue;
						int valu1 = l,valu2 = l + w;
						int newval1 = valx1 + valu2,newval2 = valx2 + valu1;
						newval2 = max(newval2,newval1);
						(ff[x][newval1][newval2 - newval1] += 1ll * f[x][j][k] * f[u][l][w] % mod) %= mod;
					}
			}
		siz[x] += siz[u];
		for (int j = 0;j <= siz[x] * K;j ++)
			for (int k = 0;k <= K;k ++)
				f[x][j][k] = ff[x][j][k],ff[x][j][k] = 0;
	}
}

int main()
{
	scanf("%d%d",&n,&K);
	for (int i = 1;i < n;i ++)
	{
		int x,y;
		scanf("%d%d",&x,&y);
		add(x,y),add(y,x);
	}
	dfs(1,0);
	for (int i = 0;i <= n * K;i ++)
		for (int j = 0;j <= K;j ++)
			ans[i + j] = (ans[i + j] + f[1][i][j]) % mod;
	for (int i = 1;i <= n * K;i ++) printf("%d\n",ans[i]);
}
```

（DP 套 DP 的代码好像还挺好写的？直接按照算法流程写就行了）

其他题目待补 qwq