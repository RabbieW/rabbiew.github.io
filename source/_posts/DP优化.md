---
title: DP 优化
date: 2022-02-19 13:01:12
tags:
    - DP
    - 笔记
categories: OI
mathjax: true
description: 各种 DP 优化的课件（学习笔记）
---

upd on 20220405 修了炸了的图和 LaTeX

upd on 20230903 关于数据结构优化 DP，可以看看新写的博客~

upd on 20240228 加了一道题 qwq

题单：<https://www.luogu.com.cn/training/144843>


有的时候，普通的动态规划不能通过题目，需要用到一些数据结构或思想进行优化。

# 单调队列优化

## 前置知识

- 单调队列

## 主要思想

在 dp 转移的时候，会取上一个阶段最优的答案进行转移得到现在的答案。

而一个一个枚举上一个阶段的所有情况会导致时间复杂度增加，所以我们可以使用单调队列进行优化。

由于单调队列的时间复杂度很小，所以经常被使用。

## 例题

CF372C Watching Fireworks is Fun <https://codeforces.com/problemset/problem/372/C>

### 解法

普通的 dp 解法时间复杂度为 $O(mn^2)$，显然过不去。

观察转移方程：$f_{i,j}$​ 表示第 $i$​ 个烟花，在位置 $j$​ 的最优答案。


$$
f_{i,j} = \max \limits_{k=1} ^{n} ( {f_{i-1,k} } +b_i-|{a_i}-j| )
$$

由于我们枚举了 $i$ 和 $j$，所以可以把确定的量提出来：

$$
f_{i,j} = \max \limits_{k=1} ^n ( {f_{i-1,k} } )+b_i-|{a_i} -j|
$$

时间复杂度降为了 $O(mn)$，可以过。

## 单调队列优化多重背包

观察一下朴素多重背包算法的式子：

$$
f[i][j]=\max \limits_{k=0} ^{m[i]} ( f[i-1][j-k\cdot w[i]]+k\cdot v[i] )
$$

其中 $i$ 表示现在枚举到第 $i$ 中物品，$j$ 表示已经用了的背包容量，$k$ 表示第 $i$ 种物品放几个，$w[i]$ 表示体积，$v[i]$​ 表示价值，$m[i]$ 表示物品个数。

令 $x=j-w[i]$，可以发现只有在 $j \equiv x \space (mod\space w[i])$​ 的情况下才能转移。

我们把这个式子改一下：

$$
\begin{aligned} f[i][j+y \cdot  w[i]] &=\max \limits_{k=1} ^{m[i]} ( f[i-1][j+(y-k)w[i]]+k\cdot v[i] ) \newline &= \max \limits_{k=1} ^{m[i]} ( f[i-1][j+(y-k) \cdot w[i]] -(y-k)v[i] )+y\cdot v[i] \end{aligned}
$$

令 $t=y-k$​​，则

$$
f[i][j+y\cdot w[i]]=\max \limits_{t=y-m[i]} ^{y-1}( f[i-1][j+t\cdot w[i]] - t\cdot v[i] )+y\cdot v[i]
$$

花括号内的东西可以用单调队列维护。

枚举 $j$​ 的时间复杂度为 $O(w[i])$​，枚举 $t$​ 的时间复杂度为 $O([\tfrac {W}{w[i]}])$​，

总的时间复杂度就是 $O(n)\times O(w[i]) \times O([\tfrac{W}{w[i]}])=O(nW)$​。

## 练习题

多重背包板子：Luogu P1776 宝物筛选 <https://www.luogu.com.cn/problem/P1776>（用单调队列做）

Loj #10183 股票交易 <https://loj.ac/p/10183>

# 斜率优化

## 主要思想

有一些题目，可以把转移方程看做直线的方程，把问题转化为求二维平面上选择一些点，最小化直线的截距。

## 例题

Loj 玩具装箱 <https://loj.ac/p/10188>

### 解法

状态转移方程：$f_i$​​​ 表示以 $i$​ 为右端点分一段的最小代价，$sum_i$​ 表示 $c$​​ 数组的前缀和。

$$
f_{i}=\min \limits_{j=0} ^{i-1} ( f_j + (i-(j+1) + sum_i-sum_j - L) ^2 )
$$

枚举 $i$，把确定的提出来：

$$
\begin{aligned}
f_i &= \min \limits_{j=0} ^{i-1}( f_j + (i+sum_i - 1-L) ^2 + (j + sum_j) ^2 - 2 (i+sum_i-1-L)(j+sum_j) )\newline
&= \min \limits_{j=0} ^{i-1} ( f_j+(j+sum_j) ^2 - 2(i + sum_i-1-L)(j + sum_j) ) + (i+sum_i - 1 - L) ^2
\end{aligned}
$$

令 $g_i=sum_i+i$，$L'=L+1$，则

$$
f_i-(g_i-L')^2 = \min ( f_j+g_j\ ^2 - 2(g_i-L')g_j )
$$

我们把直线的方程 $y=kx+b$​ 变一下，就可以得到 $b=y-kx$​。

令

$$
b_i=f_i-(g_i-L') ^2\newline
y_j = f_j+g_j\ ^2\newline
k_i = 2(g_i-L')\newline
x_j = g_j
$$

我们就成功地把转移方程变成了直线方程：

$$
b_i=\min \limits_{j=0} ^{i-1} ( y_j - k_i x_j )
$$

$k_i$ 表示一条经过点 $(x_j,y_j)$​ 的直线的斜率。

把问题转化为了选择合适的 $j$，使得 $b_i$，也就是直线的**截距**最小。

- 怎么求这个点呢？

我们画一下图。

![pkSrIjP.png](https://s21.ax1x.com/2024/04/20/pkSrIjP.png)

红色直线的斜率为 $k_i$。

我们把这条直线向上移，直到碰到一个点：

![pkSr7B8.png](https://s21.ax1x.com/2024/04/20/pkSr7B8.png)

碰到的第一个点就是要求的点。

证明：反证法，若继续向上移，$b_i$ 会继续变大，所以后面的都不会比这个点更优。

不难发现可能的答案在下凸壳（黑色线）上。

所以我们不需要枚举每一个点，只需要维护下凸壳上的点即可。

这个点的性质为：它和左边的点连成的直线的斜率 $\leq k_i<$ 它和右边的点连成的直线的斜率。

而本题中，$k_i=2(g_i-L')$​​​，$g_i$​​​ 随着 $i$​​​ 的增加而增加，所以 $k_i$​​​​ 是递增的。

所以可以用单调队列维护下凸壳，用一个指针 $ans$ 维护答案。

由于 $k_i$ 单调递增，所以每次 $ans$​ 的移动次数为均摊 $O(1)$。

在更新完 $f_i$ 后，我们把 $(x_i,y_i)$​ 也加进去。

这样，就成功地把时间复杂度降到了 $O(n)$。

## 总结

斜率优化的思想就是：把状态转移方程通过转化，把确定的量提到外面来，变成直线方程，使点的坐标代表待定的值，斜率和截距代表确定的值和待求的值，再转化成让截距最小的问题。

## 练习题

### [SDOI2016] 征途

<https://loj.ac/p/2035> / <https://www.luogu.com.cn/problem/P4072>

**题解**

$$
\begin {aligned}
ans &= m ^2 \frac {\sum (dis[i]-average)^2}{m}\newline
&= m\sum(dis[i]-average)^2\newline
&= m\sum(dis[i] ^2 + average ^2 - 2dis[i]\cdot average)\newline
&= m\sum(dis[i]^2)-2\cdot sumdis ^2 + m^2 \cdot \frac {sumdis^2}{m^2}\newline
&= m\sum(dis[i] ^2) - sumdis^2
\end {aligned}
$$

所以问题变为了使 $\sum (dis[i]^2)$ 最小。

状态转移：

$$
\begin{aligned} &f[i]=\min \limits_{j=0} ^{i-1} ( f[j]+(s[i]-s[j])^2 )\newline
\Rightarrow &f[i] = \min ( f[j]+s[i] ^2 + s[j] ^2 - 2 s[i]s[j] ) \newline
\Rightarrow &f[i]-s[i] ^2 = \min ( f[j]+s[j] ^2 - 2s[i]s[j] )\newline
\Rightarrow &b = \min ( y-kx ) \end{aligned}
$$

也就是：

$$
b[i]=f[i]-s[i] ^2\newline
y[j]=f[j]+s[j] ^2\newline
k[i]=2s[i]\newline
x[j]=s[j]
$$

后面就和上面那道题一样了。

# 四边形不等式优化

## 四边形不等式

> 设$w(x,y)$是定义在整数集合上的二元函数。若对于定义域上的任意整数 $a,b,c,d$​，其中 $a\leq b \leq c \leq d$​，都有 $w(a,d)+w(b,c) \geq w(a,c) + w(b,d)$​ 成立，则称函数 $w$​​ 满足**四边形不等式**。
> 
> 若等号永远成立，则称函数 满足**四边形恒等式**。

### 定理（另一种定义）

设 $w(x,y)$​​ 是定义在整数集合上的二元函数。若对于定义域上的任意整数 $a,b$​，其中 $a <b$​，

都有 $w(a,b + 1) + w(a+1,b) \geq w(a,b)+w(a+1,b+1)$​ 成立，则函数 $w$​ 满足四边形不等式。

### 定理证明

对于 $a<c$，有 $w(a,c+1)+w(a+1,c) \geq w(a,c)+w(a+1,c+1)$；

对于 $a+1 <c$，有 $w(a+1,c+1)+w(a+2,c) \geq w(a+1,c)+w(a+1,c+1)$。

两式相加，得到 $w(a,c+1)+w(a+2,c) \geq w(a+1,c)+w(a+2,c+1)$。

以此类推，对任意的 $a \leq b \leq c$，有 $w(a,c+1)+w(b,c) \geq w(a,c)+w(b,c+1)$。

同理，对任意的 $a\leq b \leq c \leq d$，有 $w(a,d)+w(b,c) \geq w(a,c)+w(b,d)$。

## 一维线性 dp 的四边形不等式优化

### 决策单调性

对于形如 $f[i]=\min \limits_{j=0} ^{i-1} ( f[j] + val(j,i) )$ 的转移方程，记 $p[i]$ 为令 $f[i]$ 取到最小值的值，即 $f[i]$ 的最优决策。若 $p$ 单调不减，则称 $f$​ 具有决策单调性。

### 定理

在状态转移方程 $f[i]=\min \limits_{j=0} ^{i-1} ( f[j] + val(j,i) )$ 中，若函数 $val$ 满足四边形不等式，则 $f$​ 具有决策单调性。

### 定理证明

$$
\begin {aligned} &f[i]=\min {f[j]+w(j,i)}\newline \Rightarrow &f[p[i]]+w(p[i],i) \leq f[j]+w(j,i) \space\space\space\space\space (1)\newline &j\leq p[i] \leq i \leq i' \newline \Rightarrow &w(j,i')+w(p[i],i)\geq w(j,i)+w(p[i],i')\newline \Rightarrow &w(j,i')-w(j,i) \geq w(p[i],i') - w(p[i],i)\space\space\space\space\space (2)\newline &(1)+(2) \Rightarrow f[j]+w(j,i') \geq f[p[i]]+w(p[i],i') \end{aligned}
$$

也就是对于 $i'$ 来说，$p[i]$ 比 $j$ 更优，即 $p[i'] \geq p[i]$。

### 优化过程

当 $f$ 具有决策单调性时，$p$ 会被分成若干段，使每一段中所有元素的值相同，如下图：

![pkSrqAg.png](https://s21.ax1x.com/2024/04/20/pkSrqAg.png)

当我们求出了一个新的 $f[i]$，就可以通过一些操作计算出有哪些 $i'$ 的最优决策是 $p[i]$。

根据决策单调性，我们一定可以找出一个位置 $pos$，使得 $pos$ 之前的所有位置目前存储的决策比 $p[i]$ 更优，而 $pos$ 及以后的所有位置目前存储的决策不比 $p[i]$​ 更优。

我们的目标就变为了找出 $pos$，并把 $pos$ 及以后的所有位置存储的决策更新为 $p[i]$。

直接暴力修改显然会超时，还不如不优化。这时，我们可以借助队列。

我们把 $p$​​​​ 的每一段用一个三元组记录下来：$(p_t,l_t,r_t)$​​​​​，其中 $p$​​​ 记录的是当前存储的决策，$l$​​​ 和 $r$​​​ 分别时这一段的左端点和右端点。设队尾为 $(p_0,l_0,r_0)$​​​。

如果对于 $l_0$​ 来说，$p[i]$​ 比 $p_0$​​​ 更优，则代表我们需要更新这一段存储的决策，直接弹出队尾；

如果对于 $r_0$​​​ 来说，$p_0$​​​ 比 $p[i]$​​​ 更优，则代表我们已经找到了最右的存储决策比 $p[i]$ 更优的点，即 $pos-1$​，把新的三元组 $(p[i],r_0+1,n)$​​ 插入队尾；

否则，进行二分查找，找到 $pos$，更新 $r_0$，把新三元组 $(p[i],pos,n)$​ 插入队尾。

这样，我们就把 $O(n^2)$ 的时间复杂度优化到了 $O(n \log n)$​。

## 二维区间dp的四边形不等式优化

朴素的转移方程：$f(i,j)=\min \limits_{i \leq k < j} ( f(i,k)+f(k+1,j)+w(i,j) )$

### 定理1

在转移方程 $f(i,j)=\min \limits_{i \leq k < j} ( f(i,k)+f(k+1,j)+w(i,j) )$（$f(i,i)=w(i,i)=0$）中，如果下面两个条件成立：

1. $w$ 满足四边形不等式；
2. 对于任意的 $a\leq b \leq c \leq d$，有 $w(a,d) \geq w(b,c)$​。

那么 $f$ 也满足四边形不等式。

### 定理2（二维决策单调性）

在转移方程 $f(i,j)=\min \limits_{i \leq k < j} ( f(i,k)+f(k+1,j)+w(i,j) )$（$f(i,i)=w(i,i)=0$）中，记 $p(i,j)$ 为 $f(i,j)$ 的最优决策。

如果 $f$ 满足四边形不等式，那么对于任意 $i<j$，有 $p(i,j-1) \leq p(i,j) \leq p(i+1,j)$。

---------------------------

有了定理2，我们就只需要在 $p(l,r-1) \leq k \leq p(l+1,r)$ 的范围内枚举 $k$，时间复杂度为 $O(n^2)$。

## 练习题

### Luogu P1880 石子合并

<https://www.luogu.com.cn/problem/P1880> $n\leq 5000$​

### Luogu P4767 [IOI2000] 邮局

<https://www.luogu.com.cn/problem/P4767>

**题解**

转移方程：

$$
f[i][j]=\min \limits_{k=0} ^{i-1} ( f[k][j-1]+w(k+1,i) )
$$

其中$f[i][j]$ 表示前 $i$ 个村庄放了 $j$ 个邮局的最小距离和，$w(i,j)$ 表示第 $i$ ~ $j$ 个村庄中放一个邮局的最小距离和。

显然在一个区间中，邮局要放在最中间的那个村庄那里。

$sumx[i]$ 表示前 $i$ 个村庄的坐标和。

对于 $w$：

$$
\begin{aligned} &w(i,j)=sumx[j]-sumx[(i+j)/2]-sumx[(i+j-1)/2] + sumx[i-1]\newline \Rightarrow &w(l,r+1)+w(l+1,r)-w(l,r) - w(l+1,r+1)=sumx[(l+r+1)/2]-sumx[(l+r-1)/2]\geq 0\newline \Rightarrow &w(l,r+1)+w(l+1,r) \geq w(l,r)+w(l+1,r+1) \end{aligned}
$$

所以 $w$​ 满足四边形不等式。

又因为显然对于任意的 $a\leq b \leq c \leq d$，有 $w(a,d) \geq w(b,c)$，

所以 $f$ 满足四边形不等式，具有二维决策单调性。

时间复杂度为 $O(nV)$。

### SPOJ LARMY

<https://www.spoj.com/problems/LARMY/>

# KMP（？）

主要是思想。

KMP算法中，通过**记录失配指针**减少枚举的量，优化了时间复杂度。AC自动机就运用了这种思路。

扩展KMP算法中，通过**记录最靠右的已经算出的区间**，让待计算的一部分直接使用之前算出的结果，减少了枚举的量，优化了时间复杂度。Manacher也运用了类似的思路。

# 数据结构优化

在一些情况下，我们可以用数据结构维护决策的候选集合，从而快速执行插入、查找、删除等操作。

主要是把转移的一部分过程用数据结构优化。

upd on 2023/9: 可以看看新写的线段树优化 DP 的博客！

## 练习题

（注意选用好写、适用的数据结构）

### Luogu P1442 铁球落地

<https://www.luogu.com.cn/problem/P1442>

**题解**

线段树优化dp。
先离散化，预处理出每一块木板左边会掉到那块木板，右边会掉到哪里，再从下往上dp。
预处理的时候用线段树优化。

### Luogu P1020 导弹拦截

<https://www.luogu.com.cn/problem/P1020>

**题解**

朴素算法的基础上，用树状数组 / 线段树维护最优决策。

### HBTSC PRE 2024 花神诞日 / sabzeruz

<https://www.luogu.com.cn/problem/P10200>

**题解**

一个性质：$a < b < c$ 时，$a \oplus b > \min(a \oplus c,c\oplus b)$。于是排序后可以 dp，记录当前两组的最后分别是什么。发现两组的最后肯定有一个是上一个数，状态数可以压掉一维。用 01-Trie 优化转移，可以做到 $O(n\log V)$。

# 其他的dp题

## CF1627E Not Escaping

<https://codeforces.com/problemset/problem/1627/E>
