---
title: 一些后缀数组（SA）的题目
date: 2024-04-19 14:47:45
tags:
  - 题解
  - 后缀数组（SA）
  - 字符串
categories: OI
description: 后缀数组的题目们
---

~~不知道 SA 是什么？[SA 的基础知识](https://rabbiew.github.io/2022/08/06/SA/)~~

以下所有题目都可以用 SAM 解决，这里记录 SA 做法。

# SPOJ LCS2 - Longest Common Substring II

题目链接：<https://www.spoj.com/problems/LCS2/>

## 题意

给定不超过 $10$ 个长度不超过 $10^5$ 的字符串，求它们的最长公共子串长度。

## 做法

子串就是某个后缀的前缀。把所有字符串中间加上分隔符连成一个求 SA，最长公共子串的长度就是从原先的每个字符串挑一个后缀出来求 LCP，也就是 $height$ 数组的区间求 $\min$。

考虑排名数组上的每一个区间，这个区间满足条件，当且仅当原先的每个字符串都有一个后缀在这里面。把原先的字符串当成颜色，对应后缀染色，区间满足条件即为这里面每种颜色都出现了。可以滑动窗口求所有满足条件的区间，单调队列维护区间 $\min$。时间复杂度 $O(n\log n)$。

在实现的时候，字符串连在一起时加上分隔符可以规避一些奇怪的特殊情况。

代码：
```cpp
// code by RabbieWjy
#include <iostream>
#include <cstdio>
#include <queue>
#include <cstring>
using namespace std;

char s[1000010],S[1000100];
int n,K;
int rk[2001000],ork[2001000],sa[2001000],cnt[2001000],rn[2001000],num[2001000];
int hei[1000100];
int col[1000100],Cnt[20],ans;
deque <int> q;

void SA()
{
	int m = max(n,300);
	for (int i = 1;i <= n;i ++) rk[i] = S[i],cnt[rk[i]] ++;
	for (int i = 1;i <= m;i ++) cnt[i] += cnt[i - 1];
	for (int i = n;i >= 1;i --) sa[cnt[rk[i]] --] = i;
	for (int len = 1;len < n;len <<= 1)
	{
		int no = 0;
		for (int i = n - len + 1;i <= n;i ++) num[++ no] = i;
		for (int i = 1;i <= n;i ++)
			if (sa[i] > len) num[++ no] = sa[i] - len;
		for (int i = 1;i <= m;i ++) cnt[i] = 0;
		for (int i = 1;i <= n;i ++) rn[i] = rk[num[i]],cnt[rn[i]] ++;
		for (int i = 1;i <= m;i ++) cnt[i] += cnt[i - 1];
		for (int i = n;i >= 1;i --) sa[cnt[rn[i]] --] = num[i];
		no = 0;
		for (int i = 1;i <= n;i ++) ork[i] = rk[i];
		for (int i = 1;i <= n;i ++)
			if (ork[sa[i]] == ork[sa[i - 1]] && ork[sa[i] + len] == ork[sa[i - 1] + len])
				rk[sa[i]] = no;
			else rk[sa[i]] = ++ no;
		if (no == n) break;
	}
	for (int i = 1;i <= n;i ++)
	{
		int tmp = max(hei[rk[i - 1]] - 1,0);
		while (i + tmp <= n && sa[rk[i] - 1] + tmp <= n && S[i + tmp] == S[sa[rk[i] - 1] + tmp]) tmp ++;
		hei[rk[i]] = tmp;
	}
}

int main()
{
	while (~scanf("%s",s + 1))
	{
		int m = strlen(s + 1);
		K ++;
		for (int i = 1;i <= m;i ++) S[n + i] = s[i],col[n + i] = K;
		S[n + m + 1] = K; // 分隔符
		n += m + 1;
	}
	if (K == 1)
	{
		printf("%d\n",n - 1);
		return 0;
	}
	SA();
	int i = 1,j = 1,ncnt = 0;
	Cnt[col[sa[1]]] ++;
	while (j <= n)
	{
		while (i <= j && ncnt == K && (!col[sa[i]] || Cnt[col[sa[i]]] > 1))
		{
			Cnt[col[sa[i]]] --;
			if (q.size() && q.front() == i + 1) q.pop_front();
			i ++;
		}
		if (ncnt == K) ans = max(ans,hei[q.front()]);
		Cnt[col[sa[j + 1]]] ++;
		if (col[sa[j + 1]] && Cnt[col[sa[j + 1]]] == 1) ncnt ++;
		while (q.size() && hei[q.back()] >= hei[j + 1]) q.pop_back();
		q.push_back(j + 1);
		j ++;
	}
	printf("%d\n",ans);
}
```
# Luogu P2408 不同子串个数

题目链接：<https://www.luogu.com.cn/problem/P2408>

## 题意

给你一个长为 $n$ 的字符串，求本质不同的子串的个数。

$1 \le n \le 10^5$。

## 做法

先求出 SA 和 $height$ 数组。总共的 $\frac{n(n + 1)}{2}$ 个子串中，对于每个后缀，它和前一名的 LCP 都是多算了的，要减去，也就是总共减去 $\sum \limits _{i=2} ^n height[i]$。

也许可以直接记结论？

代码简单，就不放了。

# [NOI2018] 你的名字

题目链接：<https://www.luogu.com.cn/problem/P4770> / <https://loj.ac/p/2720>

## 题意

给定字符串 $S$，$Q$ 次询问，每次询问给出字符串 $T$ 和区间 $[L,R]$，求有多少个 $T$ 的本质不同子串不是 $S_LS_{L + 1}\cdots S_R$ 的子串。

$|S| \le 5 \times 10^5$，$Q \le 10^5$，$\sum |T| \le 10^6$。

## 做法

符号：$S(L,R)$ 表示字符串 $S_LS_{L+1}\cdots S_R$，字符串 $T$ 的后缀 $i$ 表示字符串 $T_iT_{i + 1}\cdots T_{|T|}$。

和上面一道题类似，考虑从所有子串中减去算多的。

先把所有字符串加上分隔符连起来。所有本质不同子串个数就是上一题，现在需要把这些子串中是 $S(L,R)$ 的子串的减掉。

对于每个字符串 $T$ 的后缀 $i$，存在一个最长的前缀，使得它是 $S(L,R)$ 的子串，且比这个前缀长的前缀都不是 $S(L,R)$ 的子串。记这个前缀长度为 $len_i$，则有 $len_{i + 1} \ge \max(0,len_i - 1)$。证明类似 $height$ 数组的结论证明，具体来说就是 $T(i + 1,i + len_i - 1)$ 就是 $T(i,i + len_i - 1)$ 去掉第一个字符，显然也是 $S(L,R)$ 的子串。

根据这个结论，我们可以类似求 $height$ 数组的过程求出 $len$ 数组。但是，这里面需要快速判断 $T(i,i + X - 1)$ 是不是 $S(L,R)$ 的子串。这个判断相当于判断是否存在 $j \in [L,R - X + 1]$，使得 $\operatorname{LCP}(S(j,j + X - 1),T(i,i + X - 1)) = X$，也就是 $\operatorname{LCP}(S(j,|S|),T(i,|T|) \ge X$。对于一个固定的 $i$，$S(j,|S|)$ 的排名在一个包含 $rank[T(i,|T|)]$ 且区间 $height_{\min} \ge X$ 的区间里，可以二分+ST 表找出这个区间。判断就转化为了区间中是否存在 $j\in [L,R - X + 1]$ 的问题，可以通过主席树解决。

求出来 $len$ 以后，就可以把答案算出来了。注意减掉是 $S(L,R)$ 的子串个数的时候，有一些已经在求本质不同子串的时候减掉了的子串要加回来。

总时间复杂度是 $O(n\log n)$，$n=|S| + \sum |T|$。我的代码常数较大。

代码：

```cpp
// code by RabbieWjy
#include <iostream>
#include <cstdio>
#include <cmath>
#include <cstring>
#define mid ((l + r) >> 1)
using namespace std;

typedef long long ll;

int n,N,q;
char s[2000010],t[1000010];
int col[2000010],L[100010],R[100010];
int rk[4000010],cnt[4000010],sa[4000010],rn[4000010],num[4000010],ork[4000010];
int hei[2000010];
int st[30][2000010];
int ed[100010],res[2000010];
ll ans[100010];
struct node
{
	int ls,rs,sum;
} tree[500010 * 20];
int rt[2000010],tcnt;
int del[2000010],las[100010];

void SA()
{
	int m = max(n,300);
	for (int i = 1;i <= n;i ++) rk[i] = s[i],cnt[rk[i]] ++;
	for (int i = 1;i <= m;i ++) cnt[i] += cnt[i - 1];
	for (int i = n;i >= 1;i --) sa[cnt[rk[i]] --] = i;
	for (int len = 1;len < n;len <<= 1)
	{
		int no = 0;
		for (int i = n - len + 1;i <= n;i ++) num[++ no] = i;
		for (int i = 1;i <= n;i ++)
			if (sa[i] > len) num[++ no] = sa[i] - len;
		for (int i = 1;i <= m;i ++) cnt[i] = 0;
		for (int i = 1;i <= n;i ++) rn[i] = rk[num[i]],cnt[rn[i]] ++;
		for (int i = 1;i <= m;i ++) cnt[i] += cnt[i - 1];
		for (int i = n;i >= 1;i --) sa[cnt[rn[i]] --] = num[i];
		for (int i = 1;i <= n;i ++) ork[i] = rk[i];
		no = 0;
		for (int i = 1;i <= n;i ++)
			if (ork[sa[i]] == ork[sa[i - 1]] && ork[sa[i] + len] == ork[sa[i - 1] + len])
				rk[sa[i]] = no;
			else rk[sa[i]] = ++ no;
		if (no == n) break;
	}
	for (int i = 1;i <= n;i ++)
	{
		int tmp = max(0,hei[rk[i - 1]] - 1);
		while (i + tmp <= n && sa[rk[i] - 1] + tmp <= n && s[i + tmp] == s[sa[rk[i] - 1] + tmp])
			tmp ++;
		hei[rk[i]] = tmp;
	}
}

void ST()
{
	for (int i = 1;i <= n;i ++) st[0][i] = hei[i];
	for (int i = 1;i <= 22;i ++)
		for (int j = 1;j + (1 << i) - 1 <= n;j ++)
			st[i][j] = min(st[i - 1][j],st[i - 1][j + (1 << (i - 1))]);
}

int qmin(int l,int r)
{
	if (!l || !r) return 0;
	if (l > r) swap(l,r);
	l ++;
	if (l > r) return 2e9;
	int L = log2(r - l + 1);
	return min(st[L][l],st[L][r - (1 << L) + 1]);
}

int upd(int x,int l,int r,int qx)
{
	if (qx < l || qx > r) return x;
	int nw = ++ tcnt;
	tree[nw] = tree[x];
	tree[nw].sum ++;
	if (l == r) return nw;
	if (qx <= mid) tree[nw].ls = upd(tree[nw].ls,l,mid,qx);
	else tree[nw].rs = upd(tree[nw].rs,mid + 1,r,qx);
	return nw;
}

int query(int x,int y,int l,int r,int ql,int qr)
{
	if (x == y) return 0;
	if (l >= ql && r <= qr) return tree[y].sum - tree[x].sum;
	int res = 0;
	if (ql <= mid) res = query(tree[x].ls,tree[y].ls,l,mid,ql,qr);
	if (mid < qr) res += query(tree[x].rs,tree[y].rs,mid + 1,r,ql,qr);
	return res;
}

bool fnd(int x,int len,int ql,int qr)
{
	int l = 1,r = rk[x],resl = 0,resr = 0;
	while (l <= r)
	{
		if (qmin(mid,rk[x]) >= len) resl = mid,r = mid - 1;
		else l = mid + 1;
	}
	l = rk[x],r = n;
	while (l <= r)
	{
		if (qmin(rk[x],mid) >= len) resr = mid,l = mid + 1;
		else r = mid - 1;
	}
	if (query(rt[resl - 1],rt[resr],1,N,ql,qr)) return true;
	return false;
}

int main()
{
	scanf("%s",s + 1);
	scanf("%d",&q);
	n = strlen(s + 1) + 1;
	s[n] = 1;
	N = n - 1;
	for (int i = 1;i <= q;i ++)
	{
		scanf("%s",t + 1);
		scanf("%d%d",L + i,R + i);
		int m = strlen(t + 1);
		for (int j = 1;j <= m;j ++) s[n + j] = t[j],col[n + j] = i;
		n += m + 1;
		ed[i] = n - 1;
		s[n] = 1;
	}
	SA();
	ST();
	rt[0] = ++ tcnt;
	for (int i = 1;i <= n;i ++) rt[i] = upd(rt[i - 1],1,N,sa[i]);
	for (int i = 1;i <= n;i ++)
		if (col[sa[i]])
		{
			del[sa[i]] = min(ed[col[sa[i]]] - sa[i] + 1,min(ed[col[sa[i]]] - sa[las[col[sa[i]]]] + 1,qmin(i,las[col[sa[i]]])));
			ans[col[sa[i]]] += ed[col[sa[i]]] - sa[i] + 1 - del[sa[i]];
			las[col[sa[i]]] = i;
		}
	for (int i = 1;i <= n;i ++)
	{
		if (!col[i]) continue;
		int tmp = max(0,res[i - 1] - 1);
		while (i + tmp <= ed[col[i]] && tmp <= R[col[i]] - L[col[i]] && fnd(i,tmp + 1,L[col[i]],R[col[i]] - tmp)) tmp ++;
		res[i] = tmp;
		ans[col[i]] -= max(0,tmp - del[i]);
	}
	for (int i = 1;i <= q;i ++) printf("%lld\n",ans[i]);
}
```

# [美团杯2020] 半前缀计数

题目链接：<https://uoj.ac/problem/523>

## 题意

定义一个字符串的半前缀为一个前缀（可以为空）删去它的一个子串（可以为空）的结果。求字符串 $S$ 有多少个本质不同的半前缀。

$1 \le |S| \le 10^6$。

## 做法

半前缀的定义相当于一个前缀加上后面的某个子串。对于每一种半前缀，存在一个最大的 $i$，使得它由 $S(1,i)$ 加上后面的某个子串构成，且无法被 $S(1,j)$（$j > i$）和后面的某个子串构成。考虑对于每一个前缀 $S(1,i)$，计算它作为最大满足上述条件的 $i$ 的半前缀数量。

把后面加的子串记为 $T$。由于 $i$ 是最大的，$T_1 \ne S_{i + 1}$，否则可以选 $S(1,i + 1)$ 和 $T$ 去掉第一个字符拼起来。发现如果 $T_1 \ne S_{i + 1}$，则 $i$ 一定是最大的。因此只需要计算 $T_1 \ne S_{i + 1}$ 的本质不同串数量。

求出 $S$ 的 SA，由于 $T$ 要是 $S(1,i)$ 后面的某个子串，考虑从后往前求答案。可以用上两道题的结论，求出 $S(i + 1,|S|)$ 的本质不同子串数和分别以每个字母开头的本质不同子串数，然后直接求答案。中间需要维护后缀 $i + 1$ 到 $|S|$ 的按排名排序后两两相邻的 $\operatorname{LCP}$，可以用数据结构（如树状数组）快速求出当前新加入排名的前驱和后继，加上 ST 表更新答案。总时间复杂度 $O(n\log n)$，$n = |S|$。

代码：

```cpp
// code by RabbieWjy
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <cmath>
using namespace std;

typedef long long ll;

int n;
char s[1000010];
int rk[1000010],cnt[1000010],sa[1000010],num[1000010],rn[1000010],ork[2000010];
int hei[1000010];
ll nsum,sum[30],ans;
int st[30][1000010];

void SA()
{
	int m = max(n,150);
	for (int i = 1;i <= n;i ++) rk[i] = s[i],cnt[rk[i]] ++;
	for (int i = 1;i <= m;i ++) cnt[i] += cnt[i - 1];
	for (int i = n;i >= 1;i --) sa[cnt[rk[i]] --] = i;
	for (int len = 1;len < n;len <<= 1)
	{
		int no = 0;
		for (int i = n - len + 1;i <= n;i ++) num[++ no] = i;
		for (int i = 1;i <= n;i ++)
			if (sa[i] > len) num[++ no] = sa[i] - len;
		for (int i = 1;i <= m;i ++) cnt[i] = 0;
		for (int i = 1;i <= n;i ++) rn[i] = rk[num[i]],cnt[rn[i]] ++;
		for (int i = 1;i <= m;i ++) cnt[i] += cnt[i - 1];
		for (int i = n;i >= 1;i --) sa[cnt[rn[i]] --] = num[i];
		no = 0;
		for (int i = 1;i <= n;i ++) ork[i] = rk[i];
		for (int i = 1;i <= n;i ++)
			if (ork[sa[i]] == ork[sa[i - 1]] && ork[sa[i] + len] == ork[sa[i - 1] + len])
				rk[sa[i]] = no;
			else rk[sa[i]] = ++ no;
		if (no == n) break;
	}
	for (int i = 1;i <= n;i ++)
	{
		int tmp = max(0,hei[rk[i - 1]] - 1);
		while (i + tmp <= n && sa[rk[i] - 1] + tmp <= n && s[i + tmp] == s[sa[rk[i] - 1] + tmp])
			tmp ++;
		hei[rk[i]] = tmp;
	}
}

struct BIT
{
	int tree[2][1000010];

	int lowbit(int x) {return x & (-x);}

	void ini()
	{
		for (int i = 1;i <= n;i ++) tree[1][i] = n + 1;
	}

	void upd(int x)
	{
		for (int i = x;i <= n;i += lowbit(i)) tree[0][i] = max(tree[0][i],x);
		for (int i = x;i;i -= lowbit(i)) tree[1][i] = min(tree[1][i],x);
	}

	int query(int opt,int x)
	{
		int res = (opt ? n + 1 : 0);
		if (opt)
			for (int i = x;i <= n;i += lowbit(i)) res = min(res,tree[1][i]);
		else
			for (int i = x;i;i -= lowbit(i)) res = max(res,tree[0][i]);
		return res;
	}
} T[30];

void ST()
{
	for (int i = 1;i <= n;i ++) st[0][i] = hei[i];
	for (int i = 1;i <= 20;i ++)
		for (int j = 1;j + (1 << i) - 1 <= n;j ++)
			st[i][j] = min(st[i - 1][j],st[i - 1][j + (1 << (i - 1))]);
}

int qmin(int l,int r)
{
	if (!l || r == n + 1) return 0;
	l ++;
	if (l > r)
	{
		printf("ERROR\n");
		return 0;
	}
	int L = log2(r - l + 1);
	return min(st[L][l],st[L][r - (1 << L) + 1]);
}

int main()
{
	scanf("%s",s + 1);
	n = strlen(s + 1);
	SA();
	ST();
	for (int i = 0;i <= 26;i ++) T[i].ini();
	for (int i = n;i >= 0;i --)
	{
		ans += nsum - sum[s[i + 1] - 'a'];
		if (!i) break;
		nsum += n - i + 1;
		sum[s[i] - 'a'] += n - i + 1;
		int pre = T[0].query(0,rk[i]),nxt = T[0].query(1,rk[i]);
		nsum += qmin(pre,nxt);
		nsum -= qmin(pre,rk[i]) + qmin(rk[i],nxt);
		pre = T[s[i] - 'a' + 1].query(0,rk[i]),nxt = T[s[i] - 'a' + 1].query(1,rk[i]);
		sum[s[i] - 'a'] += qmin(pre,nxt);
		sum[s[i] - 'a'] -= qmin(pre,rk[i]) + qmin(rk[i],nxt);
		T[0].upd(rk[i]),T[s[i] - 'a' + 1].upd(rk[i]);
	}
	printf("%lld\n",ans + n + 1);
}
```

更新中……