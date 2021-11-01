---
title: YbtOJ 生日欢唱
date: 2021-08-22 16:36:57
tags:
 - DP
 - 区间DP
category:
 - Solutions
top: 2
mathjax: true
---


## Description

生日演唱会采用一男一女合唱的形式，每个男生和女生都有一个演唱水平值，两个水平值分别为 $a$ 和 $b$ 的同学演唱将会给大家带来 $a\times b$ 的愉悦度。演唱会开始时，$a$ 名男生和 $b$​ 名女生各排成一列。

我将会从两列的第一位同学开始，分别挑选一名男生和一名女生。如果觉得某位同学不适合登台，我就会请他回到座位，直到挑选到一位满意的同学为止。若最后有一位男生或女生找不到搭档，我只好请剩下的所有人(包括那名同学)回到座位。如果连续的一些女生或者男生没有登台演唱会损失她(他)们水平值之和的平方的愉悦度。请求出大家能获得最大的愉悦值。（回去了就不能再上来）。

$n\leq 300$

<!--more-->

## Solution

很显然是个 DP，看到数据范围大概需要 $O(n^3)$ 的算法，于是想到区间DP~~（其实这就是区间DP板块里的题）~~ 

然后我不会设计状态了，本来考虑的是 $f_{i,j}$ 表示考虑在区间 $[i,j]$ 里的男女的最大值，后来发现不太对，因为男女未必全都能匹配，如果有剩余的人，状态就出现了后效性。接着就瞎加维，然后也没啥用。

我又想男女分开考虑，也无果。

然后我就又被 DP 蹂躏了......

后来看了看题解知道了状态设计：$f_{i,j}$ 表示考虑前 $i$ 个男生，前 $j$ 个女生，况且这两个人都要选的最大价值。

那转移方程就很好想了：

对当前状态，可以由不选一部分男生、不选一部分女生的两种状态转移而来，分别枚举断点 $k$​，转移即可。下面的两个方程分别是不选一部分男生和不选一部分女生的方程。
$$
f_{i,j}=\max\limits_{1\leq k\leq i-1}\{f_{k,j-1}+a_i\times b_j+(\sum\limits_{s=k+1}^{i-1}a_s)^2\}
$$

$$
f_{i,j}=\max\limits_{1\leq k\leq j-1}\{f_{i-1,k}+a_i\times b_j+(\sum\limits_{s=k+1}^{j-1}b_s)^2\}
$$

还有个有趣的问题：怎么快速统计答案？

有个很妙的思路，在男女最后各加入一个愉悦值为 $0$ 的人，这样答案就是 $f_{n+1,n+1}$。

初始状态是男/女有一个是第一个人，然后可以推全状态。

## Code

```cpp
#include <algorithm>
#include <cstdio>
#include <iostream>
using std::max;

const int maxn = 310;

int a[maxn], b[maxn], n;
int sa[maxn], sb[maxn];
long long f[maxn][maxn]; // 前 i 个男生和前 j 个女生的最大价值
//i j 必须同时选

inline long long p_2(int x) { return (long long)x * x; }

int main()
{
    scanf("%d", &n);
    for (int i = 1; i <= n; i++)
        scanf("%d", &a[i]), sa[i] = a[i] + sa[i - 1];
    sa[n + 1] = sa[n];
    for (int i = 1; i <= n; i++)
        scanf("%d", &b[i]), sb[i] = b[i] + sb[i - 1];
    sb[n + 1] = sb[n];

    for (int i = 1; i <= n + 1; i++)
    {
        f[1][i] = a[1] * b[i] - p_2(sb[i - 1] - sb[0]);
        f[i][1] = a[i] * b[1] - p_2(sa[i - 1] - sa[0]);
    }
    for (int i = 2; i <= n + 1; i++)
        for (int j = 2; j <= n + 1; j++)
        {
            for (int k = 1; k < i; k++)
                f[i][j] = max(f[i][j], f[k][j - 1] + a[i] * b[j] - p_2(sa[i - 1] - sa[k]));
            for (int k = 1; k < j; k++)
                f[i][j] = max(f[i][j], f[i - 1][k] + a[i] * b[j] - p_2(sb[j - 1] - sb[k]));
        }
    std::cout << f[n + 1][n + 1] << std::endl;
    return 0;
}
```

## 总结

打开思路，考虑不同方向去设计状态，避免后效性。
