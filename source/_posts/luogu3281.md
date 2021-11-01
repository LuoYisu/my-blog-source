---
title: luogu3281 [SCOI2013]数数
tags:
  - DP
  - 数位DP
  - 数学
category:
  - Solutions
mathjax: true
top: 2
date: 2021-10-28 10:31:41
---
## Description

有两个 $B$ 进制数 $L,R$，求区间 $[L,R]$ 中，将所有 $B$ 进制数看成一个字符串，所有字符串的所有连续子串对应 $B$ 进制数的和（十进制） $\bmod 20130427$​。

<!--more-->

## Solution

这个数位DP实在是太毒瘤了。。。

看了看题解才搞懂，于是我来记录一下这道题。

为方便，下面记 一个数所有连续子串的和 为其权值，记为 $w(n)$​​；一个数所有后缀的对应数字的和为 $s(n)$​；一个数字位数为 $len(n)$​​。​

首先考虑有一个数 $p$，在其后面填一个数 $q$，得到新数 $\overline{pq}$​ 的权值
$$
w(\overline{pq})=w(p)+B\times s(p)+len(\overline{pq})\times q
$$
这时我们发现，后半部分实际就是 $s(\overline{pq})$，那么 $w(\overline{pq})=w(p)+s(\overline{pq})$，其中  $s(\overline{pq})=B\times s(p)+len(\overline{pq})\times q$，且 $len(\overline{pq})=len(p)+1$​。

上述过程都还算显然。于是我们找到了一个递推的思路。

因为我们要对 $w$ 求和，如果枚举 $0\sim B-1$，直接用以上的方法递推，复杂度会达到 $O(B(n+m))$。考虑优化：

既然我们要求和，那么就直接记录答案，然后大力推式子。

按照数位DP的传统思路，设 $f(i,0/1)$ 表示从高到低 $i$ 位，否/是 紧贴上界的 $\sum{w}$。

你就会发现式子开始变得不那么显然了起来。

记录 $sum(n,0/1)=\sum s(n,0/1)$；$sl(n,0/1)=\sum len(n,0/1)$；$a(i,0/1)$​ 为 否/是 紧贴上界时数的个数。

注意，当以上的第二维取 $1$ 时，要注意实际上在该位的取值只有一种（虽然有 $\sum$​）。

下面记 $p(i)$ 为当前的位数上的数字。我把低的位存在数组前面，所以第 $n$ 位由 $n+1$ 为转移来。

那么 $f(n,1)=f(n+1,1)+sum(n,1)$，

而 $sum(n,1)=sum(n+1,1)\times B+p(n)\times sl(n,1)$，

其中 $sl(n,1)=sl(n+1,1)+a(n,1)$，$a(n,1)=a(n+1,1)$。

上面这些式子都还算显然，接下来的转移就有点怪了。
$$
f(n,0)=f(n+1,0)\times B+sum(n,0)+f(n+1,1)\times p(n)
$$
前半部分就是我们之前推的式子，解释一下 $f(n+1,1)\times p(n)$：$sum(n,0)$ 已经包括了前 $n$ 位数中所有不紧贴上界的后缀和的和，这也包括了在 $n+1$ 位之前紧贴上界，在第 $n$ 位不紧贴的数。而 $f(n+1,0)$ 是上一位不紧贴的答案。我们发现，上一位紧贴的全部答案都不在现在的后缀和中，也就是没有被计算进去，而这种没有被计算的数恰好有 $p(n)$ 个（$0\sim p-1$）。所以有上式。

接下来要求 $sum(n,0)$，这个式子是真的恶心。首先记 $pre(n)=\sum_0^{n-1}$。

当 $n$ 是最高位的时候 $t=0$，否则 $t=B$。
$$
\begin{align*}
sum(n,0)=&sum(n+1,1)\times B\times p(n)+pre(p(n))\times sl(n,1)+\\
&sum(n+1,0)\times B\times B+pre(B)\times(sl(n+1,0)+a(n+1,0))+\\
&pre(t)
\end{align*}
$$
$$
sl(n,0)=t-1+sl(n+1,1)\times p(n)+(sl(n+1,1)+a(n+1,0))\times B
$$

$$
a(n,0)=t-1+a(n+1,0)\times B+a(n+1,1)\times p(n)
$$

式子的解释暂时咕了（）

于是把以上式子写成代码，这题就做完了。

## Code

```cpp
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <iostream>
using std::cout;
using std::endl;

const int maxn = 1e5 + 10;
const int mod = 20130427;
typedef long long ll;

ll f[maxn][2], sum[maxn][2], sl[maxn][2], a[maxn][2];
ll B, n, m;
ll S[maxn], l[maxn], r[maxn];

ll solve(ll *p, ll l)
{
    memset(f, 0, sizeof(f)), memset(sum, 0, sizeof(sum));
    memset(sl, 0, sizeof(sl)), memset(a, 0, sizeof(a));
    a[l][1] = 1;
    for (int i = l - 1; i >= 0; i--)
    {
        int t = i == l - 1 ? 0 : B;
        a[i][1] = a[i + 1][1];
        a[i][0] = (t - 1 + a[i + 1][0] * B % mod + a[i + 1][1] * p[i] % mod) % mod;
        sl[i][1] = sl[i + 1][1] + a[i + 1][1];
        sl[i][0] = (t - 1 + sl[i][1] * p[i] + (sl[i + 1][0] + a[i + 1][0]) * B % mod) % mod;
        sum[i][1] = (sum[i + 1][1] * B % mod + p[i] * sl[i][1]) % mod;
        sum[i][0] = (S[t] + sum[i + 1][1] * B * p[i] + S[p[i]] * sl[i][1] +
                     sum[i + 1][0] * B % mod * B % mod +
                     S[B] * (sl[i + 1][0] + a[i + 1][0])) %
                    mod;
        f[i][1] = (f[i + 1][1] + sum[i][1]) % mod;
        f[i][0] = (f[i + 1][0] * B + sum[i][0] + f[i + 1][1] * p[i]) % mod;
        //cout << a[i][1] << " " << a[i][0] << " " << sl[i][1] << " " << sl[i][0] << " " << sum[i][1] << " " << sum[i][0] << " " << f[i][1] << " " << f[i][0] << endl;
    }
    return (f[0][1] + f[0][0]) % mod;
}

int main()
{
    scanf("%lld", &B);
    for (int i = 0; i < B; i++)
        S[i + 1] = S[i] + i;
    scanf("%lld", &n);
    for (int i = 0; i < n; i++)
        scanf("%lld", &l[n - i - 1]);

    for (int i = 0; i < n; i++)
    {
        if (l[i] > 0)
        {
            l[i]--;
            break;
        }
        l[i] = B - 1;
    }
    if (l[n - 1] == 0)
        n--;
    scanf("%lld", &m);
    for (int i = 0; i < m; i++)
        scanf("%lld", &r[m - i - 1]);
    printf("%lld\n", (solve(r, m) - solve(l, n) + mod) % mod);
    return 0;
}
```

