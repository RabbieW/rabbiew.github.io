---
title: 圆方树 学习笔记
date: 2023-10-25 12:55:27
categories: OI
tags:
    - 笔记
    - 图论
mathjax: true
description: 圆方树的学习笔记
---
> 前置知识：
点双联通分量，Tarjan 算法

# 圆方树是什么

（在下面的所有内容中，认为两个点由一条边连接也双联通）

一个无向图可以建出一棵树，树上有圆点和方点，圆点对应原图中的点，方点对应点双联通分量，方点在树中连上在这个双联通分量中的圆点。每个方点周围会连一圈圆点，每个圆点会连一圈方点。

# 怎么建树

可以用 Tarjan 建圆方树。

在 Tarjan 算法中，对于每个点 $x$，我们都记录了它的 dfs 序 $dfn_x$ 和经过最多一条返祖边（对于 dfs 树来说）能到达的最小 dfs 序 $low_x$。

用 Tarjan 求割点的时候，判断一个点 $x$ 是不是割点的条件是是否存在一个儿子 $u$，使得 $low_u \geq dfn_x$，即是否存在一个儿子不经过 $x$ 就不能往上走。

如果一个点 $x$ 的儿子 $u$ 满足 $low_u=dfn_x$，代表 $u$ 只经过一条返祖边最高只能走到 $x$，这就有了一个点双联通分量。把栈里从栈顶一直到 $u$ 的元素出栈，和一个新的方点连边，同时把 $x$ 也连向这个方点（不出栈）就好了。

OI-wiki 的代码比较好看。

下面是我的代码 qwq

```cpp
void tar(int x)
{
    dfn[x] = low[x] = ++ dfncnt;
    p[x] = 1;
    st.push(x);
    for (int i = head[x];i;i = nxt[i])
    {
        int u = to[i];
        if (!dfn[u])
        {
            tar(u);
            low[x] = min(low[x],low[u]);
            if (low[u] == dfn[x])
            {
                cnt2 ++;
                int nw;
                do
                {
                    nw = st.top();
                    st.pop();
                    p[nw] = 0;
                    Add(cnt2 + n,nw),Add(nw,cnt2 + n);
                } while (nw != u);
                Add(x,cnt2 + n),Add(cnt2 + n,x);
            }
        }
        else if (p[u]) low[x] = min(low[x],dfn[u]);
    }
}
```

# 圆方树的一些性质

两个点之间所有简单路径上的点的点集，就是圆方树上两个点的路径上所有方点代表的点集的并集。

（待补）

题目：
	
[[APIO2018] 铁人两项](https://www.luogu.com.cn/problem/P4630)

技巧：给圆方树的每个点赋予适当的权值。

[[SDOI2018] 战略游戏](https://www.luogu.com.cn/problem/P4606)

结合虚树

（待补）