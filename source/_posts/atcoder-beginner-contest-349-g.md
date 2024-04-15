---
title: AtCoder Beginner Contest 349 G. Palindrome Construction 题解
date: 2024-04-15 11:09:24
tags:
  - 题解
  - AtCoder
categories: OI
description: AtCoder Beginner Contest 349 G. Palindrome Construction 的题解~
---

努力更新！

类似的题目：

- [[SCOI2016] 萌萌哒](https://www.luogu.com.cn/problem/P3295)；
- [2021-2022 ACM-ICPC Brazil Subregional Programming Contest L. Listing Passwords](https://codeforces.com/gym/103388/problem/L)

# 题意

定义一个序列 $T = (T_1,T_2,\cdots,T_M)$ 是回文的，当且仅当 $\forall i = 1,2,\cdots,M : T_i = T_{M - i + 1}$，即翻转过来之后一样。

给定一个长度为 $N$ 的序列 $A$，问是否存在一个长度为 $N$ 的序列 $S$，使得：

- 对于所有 $i = 1,2,\cdots,N$，$(S_{i - A_i},S_{i - A_i + 1},\cdots,S_{i + A_i - 1},S_{i + A_i})$ 是回文的，且 $(S_{i - A_i - 1},\cdots,S_{i + A_i + 1})$ 不是回文的。

如果存在，输出字典序最小的一个。

$1 \le N \le 2 \times 10 ^ 5$，$0 \le A_i \le \min\{i - 1,N - i\}$。

# 解法

看见这个限制，容易想到并查集。发现每个限制是两端长度相等的区间合并，考虑用倍增优化。

类似 [[SCOI2016] 萌萌哒](https://www.luogu.com.cn/problem/P3295)，可以建 $O(\log n)$ 层并查集，把两段区间类似 ST 表合并起来，最后一层一层下传，就把每个点和其他点的关系求出来了。

具体地，第 $i$ 层并查集中，$j$ 这个点代表 $[j,j + 2^i - 1]$ 这个区间。由于合并这个操作是可重复贡献的，合并两段区间 $[L_1,R_1]$ 和 $[L_2,R_2]$ 相当于在第 $k$ 层分别合并 $[L_1,L_1 + 2^k - 1]$ 和 $[L_2,L_2 + 2 ^ k - 1]$ 以及合并 $[R_1 - 2 ^ k + 1,R_1]$ 和 $[R_2 - 2 ^ k + 1,R_2]$，这里的 $k=\lfloor \log _{2} (R_1 - L_1 + 1) \rfloor$。对于下传操作，如果第 $i$ 层中，$[L_1,R_1]$ 和 $[L_2,R_2]$ 在一个并查集里，则在第 $(i - 1)$ 层中，它们的前一半和后一半分别在两个个并查集里。所以，此时要分别合并它们的前一半和后一半。每次合并的时间复杂度是 $O(1)$，下传只需要对每一层，枚举每个区间，时间复杂度是 $O(n\log n)$。

代码：

```cpp
int fnd(int x,int k) {return fa[k][x] = (fa[k][x] == x ? x : fnd(fa[k][x],k));}

void Merge(int l,int r,int k) // 在第 k 层上合并 [l, l + 2^k - 1] 和 [r, r + 2^k - 1]
{
	// printf("*Merge %d %d %d\n",l,r,k);
	l = fnd(l,k),r = fnd(r,k);
	if (l == r) return ;
	if (l < r) swap(l,r);
	fa[k][l] = r;
}

void merge(int l,int ll,int len) // 合并 [l, l + len - 1] 和 [ll, ll + len - 1]
{
	// printf("merge %d %d %d\n",l,ll,len);
	if (l == ll) return ;
	int r = l + len - 1,rr = ll + len - 1;
	int L = (len == 1 ? 0 : log2(len));
	Merge(l,ll,L);
	Merge(r - (1 << L) + 1,rr - (1 << L) + 1,L);
}
```

由于合并的两端区间一个是从前往后的，一个是从后往前的，可以把整个序列反过来拼在后面，把其中一个区间翻到后一半，顺序就一样了。这样就可以直接用上面的东西做了。

在算出点与点之间的关系后，只需要对于每个 $i$，判断 $(i - A_i - 1)$ 和 $(i + A_i + 1)$ 在不在一个并查集里面就行了。最后要输出字典序最小的解，只需要贪心地从前往后放当前能放的、不矛盾的最小数就行了。判断矛不矛盾可以直接暴力做，因为总共只有 $O(n)$ 条这样的限制。

合并的总时间复杂度是 $O(n)$，下传是 $O(n \log n)$，求解是 $O(n)$，总时间复杂度为 $O(n \log n)$。

[AC 提交记录](https://atcoder.jp/contests/abc349/submissions/52398116)。