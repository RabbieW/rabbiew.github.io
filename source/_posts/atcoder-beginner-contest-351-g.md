---
title: AtCoder Beginner Contest 351 G. Hash on Tree 题解
date: 2024-04-28 09:16:57
tags:
  - 题解
  - AtCoder
  - 树链剖分
  - 树
  - DP
categories: OI
description: AT ABC351G 的题解~
---

题目链接：[洛谷](https://www.luogu.com.cn/problem/AT_abc351_g)，[AtCoder](https://atcoder.jp/contests/abc351/tasks/abc351_g)。

赛时想出来的动态 DP 做法，但是赛时没调完。

# 题意

给定一棵 $N$ 个点的有根树，编号为 $1$ 到 $N$，根节点为 $1$，点 $i$ 的父亲是 $p_i\space (p_i < i)$。

有一个序列 $a=(a_1,a_2,\cdots,a_N)$，初始值给定。定义函数 $f(x)$ 如下：

- 如果 $x$ 是叶子，则 $f(x) = a_x$；

- 否则 $f(x) = a_x + \prod \limits_{c \in son(x)} f ( c ) $。

定义这棵树的值为 $f(1) \operatorname{mod} 998244353$。

有 $Q$ 次询问，每次将 $a_x$ 改为 $y$，询问当前树的值。

$2 \le N \le 2 \times 10 ^ 5$，$1 \le Q \le 2 \times 10^5$，$0 \le a_i,y < 998244353$。

# 解法

发现 $f(x)$ 的值由它的儿子转移得来，可以 DP 求出刚开始的所有 $f(x)$。观察到每次修改只会影响到修改的点到根路径上的 $f(x)$ 值。

对于一个点 $u$，有$f(p_u) = a_{p_u} + f(u) \prod \limits_{c\in son(p_u) \operatorname{and} c \ne u} f ( c ) $。如果 $f(u)$ 被修改了，这个式子中的其他部分并不会改变。也就是说，对于一条往上走的链，设这条链的底部是点 $v$，那么这条链的值写出来形如 $A_k(A_{k-1}(\cdots(A_1 f(v) + B_1) \cdots) + B_{k - 1}) + B_k$，也就是若干个 $Ax + B$ 嵌套起来。这种形式让我们想到可以用矩阵表示这样的转移，用线段树维护矩阵的区间积，而修改一个点的 $a_x$ 只需要修改它对应的一个矩阵就行了。

现在要把链的做法放到树上。考虑树链剖分，一个点的 $f$ 先算出轻儿子的贡献，再从重儿子直接转移过来。记这个点为 $u$，重儿子为 $v$，对应到上面的 $Ax + B$，也就是 $A = \prod \limits_{c\in son(u) \operatorname{and} c\ne v} f ( c ) $，$x = f(v)$，$B = a_u$。把转移写成矩阵形式如下：

$$
\left[ \begin{matrix} f(v) & 1 \end{matrix} \right] \times \left[ \begin{matrix} \prod \limits_{c\in son(u) \operatorname{and} c\ne v} f ( c )  & 0 \newline a_u & 1 \end{matrix} \right] = \left[ \begin{matrix} f(u) & 1 \end{matrix} \right]
$$

每次修改时，要修改被修改的点的矩阵的左下角，以及每次跳轻边的时候，要修改新算出来的链头父亲的矩阵左上角，总共 $O(\log n)$ 次修改。对于一个点 $u$，$f(u)$ 也就是把 $u$ 开始的重链拉出来，从下到上求一次矩阵乘法，左下角的值。这里可以按 dfn 建一棵维护矩阵的区间乘积的线段树，支持单点修改和区间查询，注意矩阵乘法的顺序。

跳轻边时，矩阵左上角的值可以除以之前链头贡献的值，再乘上新的链头的 $f$ 求出来。注意题目数据中存在先把一个点赋值为 $0$，再改成非 $0$ 值的情况，不能只维护当前的矩阵左上角的值，否则还原不回去。可以对每个点，记录儿子中有多少个 $0$ 以及非 $0$ 儿子的乘积，如果儿子中有 $0$，则矩阵左上角也为 $0$，否则为儿子的积。

总时间复杂度为 $O(Kn \log n + QK \log n (\log n +  \log 998244353))$。这里的 $K$ 是矩阵乘法的复杂度，$\log 998244353$ 是因为我在算新的矩阵左上角的值时用了逆元。

[AC 提交记录](https://atcoder.jp/contests/abc351/submissions/52901323)。