---
title: CF13B
tags:
  - 计算几何
category:
  - Solutions
mathjax: true
top: 2
date: 2021-08-29 09:57:11
---
## Description

给定三条线段，判断能否构成 A，即是否满足以下条件：

1. 有两条线段有公共点（下称“第一、二条线段”，另一条线段称“第三条线段”）；
2. 第三条线段的两个端点分别在第一、二条线段上；

3. 第一、二条线段夹角大于 $0$，小于 $\dfrac{\pi}{2}$；

4. 第三条线段分别将第一、二条线段截成两段，较短的线段与较长的线段的长度比不小于 $\dfrac14$。

<!--more-->

## Solution

这是道计算几何的水题。~~想水黑题的快来~~

其实只需要暴力模拟即可，每组数据时间复杂度 $O(1)$，总时间复杂度 $O(T)$。

我用的纯计算几何的做法，用向量进行计算。因此把向量封装了，代码还算简单。

第一个要求直接暴力枚举，找有没有相同的点即可。

第二个要求也是计算几何基本操作：先用向量叉积是否为 $0$ 判断三点是否共线，再看点是否在以线段为对角线的矩形内部。

第三个要求本来想用叉积的几何意义配合反三角函数搞，但我不知道为啥一直挂，后来意识到是因为这个做法没法判断钝角。于是用余弦定理，就过了。

第四个要求很简单，就用向量做减法然后算一下验证就行。

感觉比起解析几何要省点脑子。

## Code

```cpp
#include <algorithm>
#include <cmath>
#include <cstdio>
#include <iostream>
using std::max;
using std::min;

const double pi = 3.141592653589793;
const double eps = 1e-10;

class Vector // 向量（也用来存点）
{
public:
    long long x, y;
    Vector() { x = y = 0; }
    Vector(long long a, long long b) { x = a, y = b; }
    bool operator==(const Vector &b) const { return x == b.x && y == b.y; }
    Vector operator+(const Vector &b) const { return Vector(x + b.x, y + b.y); }
    Vector operator-(const Vector &b) const { return Vector(x - b.x, y - b.y); }
    long long operator*(const Vector &b) const { return x * b.x + y * b.y; }
    long long operator&(const Vector &b) const { return x * b.y - b.x * y; } // 向量叉积
    Vector operator=(const Vector &b)
    {
        x = b.x, y = b.y;
        return *this;
    }
    long long dis_pow() { return (long long)x * x + y * y; }
    double dis() { return sqrt((double)x * x + y * y); }
} p[7];

bool in_seg(Vector a, Vector b, Vector x) // 判断点 x 是否在 线段ab 上
{
    if (((x - a) & (a - b)) != 0) // 叉乘为 0
        return false;
    return min(a.x, b.x) <= x.x && x.x <= max(a.x, b.x) && min(a.y, b.y) <= x.y && x.y <= max(a.y, b.y);
}

int main()
{
    int T;
    std::cin >> T;
    while (T--)
    {
        Vector s;
        for (int i = 1; i <= 6; i++)
            std::cin >> p[i].x >> p[i].y;
        
        int a = -1, b = -1, c = -1; // 第一、二条线段为 a、b，第三条线段为 c
        
        for (int i = 1; i <= 5; i++)
            for (int j = i + 1; j <= 6; j++)
                if (p[i] == p[j]) // 找到三条线段
                {
                    a = (i - 1) / 2 + 1;
                    b = (j - 1) / 2 + 1;
                    c = 6 - a - b;
                    s = p[i]; // s 为相同的端点
                    break;
                }
        if (a == -1) // 没有相同的点
        {
            puts("NO");
            continue;
        }
        
        Vector a1 = p[a * 2 - 1], a2 = p[a * 2], b1 = p[b * 2 - 1], b2 = p[b * 2], c1 = p[c * 2 - 1], c2 = p[c * 2]; // 6 个点分别用向量表示
        Vector _d1, _d2, _d3, _d4; // 第三条线段端点到到其所在线段两个端点的向量
        // 分两种情况：第三条线段一个端点在 a 上，另一个在 b 上，或者相反
        if (in_seg(a1, a2, c1) && in_seg(b1, b2, c2))
            _d1 = c1 - a1, _d2 = c1 - a2, _d3 = c2 - b1, _d4 = c2 - b2;
        else if (in_seg(a1, a2, c2) && in_seg(b1, b2, c1))
            _d1 = c2 - a1, _d2 = c2 - a2, _d3 = c1 - b1, _d4 = c1 - b2;
        else
        {
            puts("NO");
            continue;
        }
        
        Vector _a = a1 - s + a2 - s, _b = b1 - s + b2 - s; // 第一、二条线段的向量，这里的写法避免了分类讨论
        Vector a_b = _a - _b; // a、b线段组成三角形的另一条边
        if (_a.dis_pow() + _b.dis_pow() < a_b.dis_pow()) // 余弦定理判断夹角
        {
            puts("NO");
            continue;
        }
        
        double d1 = _d1.dis(), d2 = _d2.dis(), d3 = _d3.dis(), d4 = _d4.dis();
        if (d1 > d2)
            std::swap(d1, d2);
        if (d3 > d4)
            std::swap(d3, d4);
        // 这题似乎不卡精度，我就直接用 double 比较了，其实最好用距离的平方之比与 16 比较
        if (d2 / d1 - 4 <= eps && d4 / d3 - 4 <= eps)
            puts("YES");
        else
            puts("NO");
    }
}
```

