---
title: SA 的 基础 知识
date: 2022-08-06 16:28:38
tags:
    - 字符串
    - 笔记
categories: OI
mathjax: true
description: SA 学习笔记
---

# SA 的 基础 知识

~~你问我为什么是基础？因为我还没有学会更难的~~

## 后缀数组

### 后缀数组（SA）是什么？

主要指两个数组：$sa$ 和 $rk$。

$sa$ 存储的是**后缀是什么**，$rk$​ 存储的是**排名是多少**。

$sa[i]$ 代表将所有后缀排序后，排名为 $i$ 的后缀；$rk[i]$ 表示从第 $i$ 个字符开始的后缀的排名。

显然，有性质 $sa[rk[i]]=rk[sa[i]]=i$。​

知道这两个数组中的一个可以 $O(n)$​ 求出另一个。

下面用 $Sa(i)$​ 表示从第 $i$​ 个字符开始的后缀，$Str(i,j)$​ 表示从第 $i$​ 个字符开始，长度为 $j$​​ 的字符串。

### 维护方法

**算法一**：暴力求出每一个后缀，存进一个数组里，```sort``` 排序。由于比较两个字符串的大小是 $O(n)$​ 的，所以时间复杂度为 $O(n^2 \log n)$​​​​。

**算法二**：

运用倍增的思想，用 $rk_{len}[i]$ 表示的 $Str(i,2 ^{len})$​排名，

可以用 $rk _{len-1}$ 推出 $rk _{len}$。

显然，$Str(i,2 ^{len})$​​​​ 是由 $Str(i,2 ^{len-1})$​​​​ 和 $Str(i + 2 ^{len - 1},2 ^{len - 1})$​​​​​ 两个字符串组成的。

根据字符串大小比较的规则，两个长度为 $len$​ 的字符串的大小为以 $Str(i,2 ^{len - 1})$ 为第一关键字，以 $Str(j,2 ^{len - 1})$ 为第二关键字比较的结果。

因此，按照上述比较规则，将上一次求出的结果进行排序即可得出这一次的结果。

注意，排序后要进行去重，因为两个一样的字符串排名是相等的。

时间复杂度 $O(n \log^2 n)$。

倍增排序示意图（from OI-Wiki）：

![](https://oi-wiki.org/string/images/sa2.png)

**算法三**：可以用[基数排序](https://oi-wiki.org/basic/radix-sort/)将算法二优化至 $O(n \log n)$。

代码：

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;

char s[1000010];
int n,m;
int cnt[2000010];
int rk[2000010],sa[1000010],num[1000010],tmp[2000010];

int main()
{
    scanf("%s",s + 1);
    n = strlen(s + 1);
    m = max(n,250);
    for (int i = 1;i <= n;i ++) cnt[s[i]] ++,rk[i] = s[i];
    for (int i = 1;i <= m;i ++) cnt[i] += cnt[i - 1];
    for (int i = n;i >= 1;i --) sa[cnt[rk[i]] --] = i;// 对每个后缀的第一位计数排序
    for (int len = 1;len <= n;len <<= 1)
    {
        memset(cnt,0,sizeof(cnt));
        for (int i = 1;i <= n;i ++) num[i] = sa[i];
        for (int i = 1;i <= n;i ++) cnt[rk[num[i] + len]] ++;
        for (int i = 1;i <= m;i ++) cnt[i] += cnt[i - 1];
        for (int i = n;i >= 1;i --) sa[cnt[rk[num[i] + len]] --] = num[i];// 对第二关键字计数排序
        memset(cnt,0,sizeof(cnt));
        for (int i = 1;i <= n;i ++) num[i] = sa[i];
        for (int i = 1;i <= n;i ++) cnt[rk[num[i]]] ++;
        for (int i = 1;i <= m;i ++) cnt[i] += cnt[i - 1];
        for (int i = n;i >= 1;i --) sa[cnt[rk[num[i]]] --] = num[i];// 对第一关键字计数排序
        int no = 0;
        for (int i = 1;i <= n;i ++) tmp[i] = rk[i];
        for (int i = 1;i <= n;i ++)
        {
            if (tmp[sa[i]] == tmp[sa[i - 1]] && tmp[sa[i] + len] == tmp[sa[i - 1] + len])
                rk[sa[i]] = no;
            else no ++,rk[sa[i]] = no;
            // 计算每个字符串的排名
        }
    }
    for (int i = 1;i <= n;i ++) printf("%d ",sa[i]);
}
```

**算法四**：有时间复杂度 $O(n)$​​ 的算法​，但是一般来说算法三够用~~但是我不会~~

模板：[LOJ #111 后缀排序](https://loj.ac/p/111) （Luogu P3809）

代码：

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;

char s[1000010];
int n,m;
int cnt[2000010];
int rk[2000010],sa[1000010],num[1000010],tmp[2000010],rn[2000010];

int main()
{
    scanf("%s",s + 1);
    n = strlen(s + 1);
    m = max(n,250);
    for (int i = 1;i <= n;i ++) cnt[s[i]] ++,rk[i] = s[i];
    for (int i = 1;i <= m;i ++) cnt[i] += cnt[i - 1];
    for (int i = n;i >= 1;i --) sa[cnt[rk[i]] --] = i;// 对每个后缀的第一位计数排序
    for (int len = 1;len < n;len <<= 1)
    {
        int no = 0;
        // num[no] 表示按第二关键字排好序后，第 no 位对应的第一关键字 x 的 sa[x] 是什么
        for (int i = n;i >= n - len + 1;i --) no ++,num[no] = i;
        // 后缀长度不满 len，就直接把这个后缀扔到前面去（第二关键字为 0）
        for (int i = 1;i <= n;i ++) if (sa[i] > len) no ++,num[no] = sa[i] - len;
        // 剩下的按第二关键字的顺序把第一关键字排好
        memset(cnt,0,sizeof(cnt));
        for (int i = 1;i <= n;i ++) rn[i] = rk[num[i]],cnt[rn[i]] ++;// 记录 rk[num[i]]，减少不连续访问
        for (int i = 1;i <= m;i ++) cnt[i] += cnt[i - 1];
        for (int i = n;i >= 1;i --) sa[cnt[rn[i]] --] = num[i];// 对第一关键字计数排序
        no = 0;
        for (int i = 1;i <= n;i ++) tmp[i] = rk[i];
        for (int i = 1;i <= n;i ++)
        {
            if (tmp[sa[i]] == tmp[sa[i - 1]] && tmp[sa[i] + len] == tmp[sa[i - 1] + len])
                rk[sa[i]] = no;
            else no ++,rk[sa[i]] = no;
            // 处理 rk
        }
        if (no == n) break;// 如果每个后缀的排名都不一样了，意味着排好了（这之后每个后缀的排名都不会变）
    }
    for (int i = 1;i <= n;i ++) printf("%d ",sa[i]);
}
```

### 例题

#### 字符串匹配

即在主串 $T$ 中寻找模式串 $S$​。

**解法**：

若 $S$ 在 $T$ 中出现，那么它一定是一些后缀的前缀。

由于已经将后缀通过后缀数组有序化，那么前缀为 $S$ 的后缀一定是连续的，直接二分查找即可。时间复杂度 $O(n \log n + \log n)$，可以求出 $S$ 的每个出现位置。

-----------------------------

#### [Luogu P4051  [JSOI]字符加密](https://www.luogu.com.cn/problem/P4051)

喜欢钻研问题的JS 同学，最近又迷上了对加密方法的思考。一天，他突然想出了一种他认为是终极的加密办法：把需要加密的信息排成一圈，显然，它们有很多种不同的读法。

例如‘JSOI07’，可以读作： JSOI07 SOI07J OI07JS I07JSO 07JSOI 7JSOI0 把它们按照字符串的大小排序： 07JSOI 7JSOI0 I07JSO JSOI07 OI07JS SOI07J 读出最后一列字符：I0O7SJ，就是加密后的字符串。给定一个字符串 $S$，求加密后的字符串。

对于 $40 \%$ 的数据，$\left| s \right| \leq 10000$；

对于所有数据，$\left| s \right| \leq 10^5$。

**解法**：

将字符串复制一遍到后面，就变成了普通的后缀排序问题。

--------------------

#### [Luogu P2870 [USACO07DEC]Best Cow Line G](https://www.luogu.com.cn/problem/P2870)

给你一个字符串，每次从首或尾取一个字符组成字符串，问所有能够组成的字符串中字典序最小的一个。

$1 \leq N \leq 5\times 10^5$。

**解法**：

考虑暴力做法，每次从头选还是从尾选只需要判断原串小还是反串小，选小的那个更优，$O(n)$ 判断，总时间复杂度 $O(n^2)$。

考虑优化判断过程。

### $height$ 数组

#### $lcp$

$lcp(i,j)$ 表示字符串 $i$ 和 $j$​ 的最长公共前缀。

下文中用 $lcp(i,j)$ 表示它们的最长公共前缀长度。

#### 定义

$height_i$ 表示 $lcp(sa[i - 1],sa[i])$。

#### 求 $height$ 数组

有一个定理：$height_{rk[i]} \geq height_{rk[i - 1]} - 1$。

证明：

定义 $Suffix(x)$ 为排名为 $x$ 的后缀，$S(x)$ 表示以第 $x$ 个字符开头的后缀。

设已知 $height_{rk[i - 1]}$。

若 $height_{rk[i - 1]} \leq 1$，上式成立；

若 $height_{rk[i - 1]} > 1$：

设 $Suffix(rk[i - 1])$ 为 `aAC`，$Suffix(rk[i - 1] - 1)$ 为 `aAB`，且 $lcp(B,C) = 0$，$B < C$，则 $height_{rk[i - 1]} = 1+|A|$。

而 $S(i)$ 为 `AC`，$S(sa[rk[i - 1] - 1] + 1)$ 即 $Suffix(rk[i - 1] - 1)$ 在原串中的后一个后缀为 `AB`，

所以 $S(sa[rk[i - 1] - 1] + 1)$ 肯定在 $S(i)$ 前面。

所以 $lcp(s(i),s(sa[rk[i] - 1])) \geq |A| = height_{rk[i - 1]} - 1$，即它们的最大公共前缀有一个前缀为 `A`。

#### $height$ 数组求 $lcp$

有一个定理：$lcp(sa_i,sa_j) = \min \limits _{k=i + 1} ^j \{ height_k \}$。

证明：感性理解一下，由于后缀已经排好了序，两个后缀的排名差越大，$lcp$ 就越小，

$lcp(sa_{i},sa_{i+2}) = \min \{ height_{i + 1},height_{i + 2} \}$。

-------------------------

有了这个定理，两个**子串**求 $lcp$ 就变成了区间求最小值（RMQ）问题。通常用 ST 表解决，或者观察题目性质，或者其他数据结构。

$lcp(Str(l_1,r_1),Str(l_2,r_2)) = \min \{ lcp(S(l_1),S(l_2)), (r_1 - l_1 + 1),(r_2 - l_2 + 1) \}$。

https://oi-wiki.org/string/sa/
