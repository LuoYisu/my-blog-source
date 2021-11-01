---
title: _slb's Code Style
tags:
  - OI
category:
  - OI
mathjax: true
top: 3
date: 2021-10-14 16:52:02
---

本文介绍 _slb 的代码规范。关于我的代码规范，她非常地正经、非常地合理。

基本参考了 [_rqy](https://rqy.moe/uncategorized/rqy-s-Code-Style-for-OI/) 的风格。

<!--more-->

## 概述

不使用 `#include <bits/stdc++.h>`，头文件顺序不做要求。

不使用 `using namespace std;`，可以使用 `using xxx::foo`。

尽量不使用 `#define`，多使用 `typedef`，`const`，`inline`。

一行长度不限，但不能太长，最多 $3/4$ 屏幕宽。要易于阅读。

## 缩进

每个代码块使用四个空格缩进。

## 空格及换行

大括号换行。

加空格地方：

- 二目、三目运算符两侧；
- `,`，`;` 的右边（前提是不处于行末）；
- 流程控制关键字与其后面的括号之间；
- `do-while` 中的 `while` 与其左面的大括号之间；
- 当整组大括号都在一行时，左大括号前打一个空格；
- 常成员函数的 `const` 两侧。

不加空格地方：

- 小括号、中括号与其内部表达式等之间；

- 函数名与左括号之间；
- 单目运算符前后；
- `.`，`::`，`->` 的两侧；
- `operator` 与重载的运算符之间。

表达式过长可以分多行——缩进的关键是对齐。

很短的函数可以写到一行。

## 空行

所有`#include <foobar>`与`using foo::bar;`之间不应空行，之后应空一行。

一系列常量定义的上下应有空行。

函数/结构体定义两侧大概可以有空行（一系列短小到可以写到一行的函数，如`min`, `max`，之间可以不空行）。

一系列全局变量定义的上下应有空行。

语句之间可根据其意义酌情空行。

任何位置**不能**出现连续的两个（或以上）空行。

## 函数

`main` 函数返回值最好是 `int`。

传参可以尽量传引用，心情好的话加 `const`。

单个函数不应过长。

## 命名

瞎命名，采用下划线命名法。

没啥讲究，看心情。

心情好类/结构体的开头字母大写。

基本都使用小写字母，少使用拼音，多使用英文。

为了区分可以用下划线作为一些变量的开头。

## 其他

**不进行压行**，一行最多一个语句。

如果语义连续，可以酌情使用逗号。

三目运算符合适就用。尽量做到易于阅读。

`if`，`while`，`for` 等流程控制语句，即使后面只有单个语句，也**不能**放在一行。

## example

```cpp
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <ctime>
#include <iostream>
#include <random>
using std::cout;
using std::endl;

const int maxn = 1e5 + 10;

class Treap
{
public:
    struct Node
    {
        Node *son[2];
        int num, rnd, siz;
        int val;
        void push_up()
        {
            siz = num;
            for (int i = 0; i < 2; i++)
                if (son[i] != nullptr)
                    siz += son[i]->siz;
        }
    } a[maxn];
    int tot;
    Node *root;
    Treap() : tot(0), root(nullptr) { memset(a, 0, sizeof(a)); }
    int get_size(const Node *x) { return x == nullptr ? 0 : x->siz; }
    /*
    void output(Node *&now, const int &indent)
    {
        putchar('>'), putchar(' ');
        for (int i = 0; i < indent; ++i)
            putchar('|'), putchar(' ');
        if (!now)
        {
            puts("NULL");
            return;
        }
        printf("%ld:%d %d,%d %u\n", now - a, now->val, now->siz, now->num, now->rnd);
        output(now->son[0], indent + 1);
        output(now->son[1], indent + 1);
    }
    inline void output() { output(root, 0); }
    */
    Node *new_node(int x)
    {
        tot++;
        a[tot].val = x, a[tot].num = a[tot].siz = 1, a[tot].rnd = rand();
        a[tot].son[0] = a[tot].son[1] = nullptr;
        return &a[tot];
    }
    void rotate(Node *&now, const bool d) // 1: left 0: right
    {
        Node *t = now->son[d];
        now->son[d] = t->son[d ^ 1];
        t->son[d ^ 1] = now;
        t->push_up(), now->push_up();
        now = t;
    }
    
    void insert(Node *&now, const int x)
    {
        if (now == nullptr)
        {
            now = new_node(x);
            return;
        }
        (now->siz)++;
        if (now->val != x)
        {
            bool d = now->val < x;
            insert(now->son[d], x);
            if (now->rnd < now->son[d]->rnd)
                rotate(now, d);
        }
        else
            ++(now->num);
        now->push_up();
    }
    inline void insert(int x) { insert(root, x); }
    
    void erase(Node *&now, int x)
    {
        if (now == nullptr)
            return;
        if (now->val != x)
            erase(now->son[now->val < x], x);
        else if (now->son[0] != nullptr && 
                (now->son[1] == nullptr || now->son[0]->rnd > now->son[1]->rnd))
            rotate(now, 0), erase(now->son[1], x);
        else if (now->son[1] != nullptr)
            rotate(now, 1), erase(now->son[0], x);
        else
        {
            --(now->siz), --(now->num);
            if (!now->num)
                now = nullptr;
            return;
        }
        now->push_up();
    }
    inline void erase(int x) { erase(root, x); }
    
    int get_rank(Node *&now, int x)
    {
        if (now == nullptr)
            return 0;
        if (now->val < x)
            return get_rank(now->son[1], x) + get_size(now->son[0]) + now->num;
        else if (now->val > x)
            return get_rank(now->son[0], x);
        else
            return get_size(now->son[0]) + 1;
    }
    inline int get_rank(int x) { return get_rank(root, x); }
    
    int rank(Node *now, int x)
    {
        if (now == nullptr)
            return 0;
        if (now->son[0] != nullptr && now->son[0]->siz >= x)
            return rank(now->son[0], x);
        else if (get_size(now->son[0]) + now->num < x)
            return rank(now->son[1], x - get_size(now->son[0]) - now->num);
        else
            return now->val;
    }
    inline int rank(int x) { return rank(root, x); }
    
    int prefix(Node *now, int x)
    {
        if (now == nullptr)
            return -(1 << 30);
        else if (now->val >= x)
            return prefix(now->son[0], x);
        else
            return std::max(now->val, prefix(now->son[1], x));
    }
    inline int prefix(int x) { return prefix(root, x); }
    
    int suffix(Node *now, int x)
    {
        if (now == nullptr)
            return 1 << 30;
        else if (now->val <= x)
            return suffix(now->son[1], x);
        else
            return std::min(now->val, suffix(now->son[0], x));
    }
    inline int suffix(int x) { return suffix(root, x); }
} treap;

int main()
{
    srand(time(0));
    int n;
    scanf("%d", &n);
    for (int i = 1; i <= n; i++)
    {
        int opt, x;
        scanf("%d%d", &opt, &x);
        if (opt == 1)
            treap.insert(x);
        else if (opt == 2)
            treap.erase(x);
        else if (opt == 3)
            printf("%d\n", treap.get_rank(x));
        else if (opt == 4)
            printf("%d\n", treap.rank(x));
        else if (opt == 5)
            printf("%d\n", treap.prefix(x));
        else
            printf("%d\n", treap.suffix(x));
    }
    return 0;
}

```

**此为 带旋Treap 模板**。

## 扩 展 阅 读

[传 世 经 典](https://www.luogu.com.cn/blog/113833/guan-yu-shr-di-du-liu-ma-feng-jpg)

[圣 经](https://www.luogu.com.cn/blog/236807/Code-style) 			（似乎有点年久失修了）

