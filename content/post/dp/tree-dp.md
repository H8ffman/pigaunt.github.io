---
title: "树形动态规划"
date: 2021-10-18T10:50:57+08:00

tags : [
  "动态规划"
]

categories: [
  "动态规划"
]

---

# 树形动态规划

## 问题简介

需要用动态规划解决的问题被搬到了树上（不再是线性或区间的），此类问题与其他的区别仅仅在于遍历所有状态需要在树上进行，而不是用一个循环。

## 例题

### [AcWing-1072-树的最长路径](https://www.acwing.com/problem/content/1074/)

树的直径：树上距离最远的两个点。

**方法一：两次搜索**

任取一个点，找到与这个点距离最远的点$u$，然后从$u$出发，找到与$u$距离最远的点$v$，$u \rightarrow v$就是直径。可以用反证法证明。

**方法二：树形动态规划**

以一个点为根，将这棵树提起来，根向下找到路径长的最大值$d1$，和路径长的次大值$d2$，直径就是以各个点为根产生的$d1 + d2$的最大值。

```cpp
#include <iostream>

const int N = 500005, M = N << 1;

int n, diam = 0;
int head[N], total;

struct Edge
{
    int to, w;
    int next;
} edges[M];

void Add(int a, int b, int c)
{
    edges[total].to = b;
    edges[total].w = c;
    edges[total].next = head[a];
    head[a] = total++;
}

int Search(int u, int parent)
{
    int d1 = 0, d2 = 0;

    for (int i = head[u]; i != -1; i = edges[i].next)
    {
        int v = edges[i].to, w = edges[i].w;
        if (v == parent)
            continue;

        int d = Search(v, u) + w;

        if (d >= d1)
        {
            d2 = d1;
            d1 = d;
        }
        else if (d > d2)
            d2 = d;
    }

    diam = std::max(diam, d1 + d2);
    return d1;
}

int main()
{
    for (int i = 0; i < N; i++)
        head[i] = -1;

    std::cin >> n;
    for (int i = 1; i <= n - 1; i++)
    {
        int a, b, c;
        std::cin >> a >> b >> c;
        Add(a, b, c);
        Add(b, a, c);
    }

    Search(1, 0);

    std::cout << diam << std::endl;
    return 0;
}
```

### [LibreOJ-10155-数字转换](https://loj.ac/p/10155)

数字间的转换可以看作节点之间的连接。由于一个数$u$的约数之和$v$只有一个确定的值，我们可以把$v$构造成$u$的父节点。最长的转换就是树的最长路径。

求约数之和时为了快，可以反向求（先找约数是哪些数的约数，再构造数），这样时间复杂度就从$O(n \sqrt n)$降至$O(n \ln n)$。

```cpp
#include <iostream>

const int N = 50005, PosInf = 1e9;

struct Edge
{
    int to, next;
} edges[N];

int n, head[N], total, ans = -PosInf;
int sum[N]; // 约数之和
bool hasParent[N];

void Add(int a, int b)
{
    Edge &e = edges[total];
    e.to = b;
    e.next = head[a];
    head[a] = total++;
}

void Init()
{
    for (int i = 1; i <= n; i++)
        for (int j = 2; j <= n / i; j++) // 用n / i防止溢出
            sum[i * j] += i;

    for (int i = 0; i < N; i++)
        head[i] = -1;

    for (int i = 2; i <= n; i++)
        if (i > sum[i])
        {
            Add(sum[i], i);
            hasParent[i] = true;
        }
}

int Search(int u)
{
    int d1 = 0, d2 = 0;
    for (int i = head[u]; i != -1; i = edges[i].next)
    {
        int v = edges[i].to;
        int dis = Search(v) + 1;

        if (dis >= d1)
        {
            d2 = d1;
            d1 = dis;
        }
        else if (dis > d2)
            d2 = dis;
    }

    ans = std::max(ans, d1 + d2);
    return d1;
}

int main()
{
    std::cin >> n;
    Init();

    for (int i = 1; i <= n; i++)
        if (!hasParent[i])
            Search(i);

    std::cout << ans << std::endl;
    return 0;
}
```

### [洛谷-P2015-二叉苹果树](https://www.luogu.com.cn/problem/P2015)

本题类似有依赖的背包问题，用$f_{i, j}$表示在以$i$为根的子树中选$j$个树枝的最大价值。

```cpp
#include <iostream>

const int N = 105, M = 205;

struct Edge
{
    int to, w;
    int next;
} edges[M];

int n, m;
int head[N], total;
int f[N][N];

void Add(int a, int b, int c)
{
    Edge &e = edges[total];
    e.to = b;
    e.w = c;
    e.next = head[a];
    head[a] = total++;
}

void Search(int u, int parent)
{
    for (int i = head[u]; i != -1; i = edges[i].next) // 分组
    {
        int v = edges[i].to;
        if (v == parent)
            continue;

        Search(v, u);

        for (int j = m; j >= 0; j--) // 体积
            for (int k = 0; k < j; k++) // 决策
                f[u][j] = std::max(f[u][j], f[u][j - k - 1] + f[v][k] + edges[i].w);
    }
}

int main()
{
    for (int i = 0; i < N; i++)
        head[i] = -1;

    std::cin >> n >> m;

    for (int i = 1; i <= n - 1; i++)
    {
        int a, b, c;
        std::cin >> a >> b >> c;
        Add(a, b, c);
        Add(b, a, c);
    }

    Search(1, -1);

    std::cout << f[1][m] << std::endl;
    return 0;
}
```

### [AcWing-323-战略游戏](https://www.acwing.com/problem/content/325/)

一个点可以放或者不放，我们用$0$和$1$表示，那么$f_{i, j}$可以表示在以$i$为根的子树中放，且$i$的摆放情况是$j$的最少摆放个数。因此在以$u$为根的子树中（$u \rightarrow v$）$f_{u, 0} = \sum f_{v, 1}$，$f_{u, 1} = \sum \min(f_{v, 0}, f_{v, 1})$

```cpp
#include <iostream>
#include <cstdio>

const int N = 1505;

struct Edge
{
    int to, next;
} edges[N];

int n, head[N], total;
int f[N][2];
bool isNotRoot[N];

void Add(int a, int b)
{
    Edge &e = edges[total];
    e.to = b;
    e.next = head[a];
    head[a] = total++;
}

void Search(int u)
{
    f[u][0] = 0;
    f[u][1] = 1;
    for (int i = head[u]; i != -1; i = edges[i].next)
    {
        int v = edges[i].to;
        Search(v);

        f[u][0] += f[v][1];
        f[u][1] += std::min(f[v][0], f[v][1]);
    }
}

int main()
{
    while (scanf("%d", &n) != -1)
    {
        for (int i = 0; i < N; i++)
        {
            head[i] = -1;
            isNotRoot[i] = false;
        }
        total = 0;

        for (int i = 1; i <= n; i++)
        {
            int u, count;
            scanf("%d:(%d)", &u, &count);
            for (int j = 1; j <= count; j++)
            {
                int v;
                scanf("%d", &v);
                Add(u, v);
                isNotRoot[v] = true;
            }
        }

        int root = 0;
        while (isNotRoot[root])
            root++;

        Search(root);

        printf("%d\n", std::min(f[root][1], f[root][0]));
    }
    return 0;
}
```

### [LibreOJ-10157-皇宫看守](https://loj.ac/p/10157)

本题与上一题的唯一区别在于点上带了权值。但要注意，如果点上不带权值，最佳情况一定是有一个点看其它的点，这时两个数可以完美地表示一个点的状态；但本题由于权值的加入，最佳情况可能是两个点同时看中间的点，这时需要三个数才能满足状态的表示。

用$0$表示该点被父节点看到、$1$表示被子节点看到、$2$表示在该点放看守，$f_{i, j}$的定义和上题相同。可以得到$f_{u, 0} = \sum \min(f_{v, 1}, f_{v, 2})$，$f_{u, 2} = \sum \min(f_{v, 0}, f_{v, 1}, f_{v, 2})$和$f_{u, 1} = \min(f_{v, 2} + f_{u, 0} - \min(f_{v, 1}, f_{v, 2}))$。


```cpp
#include <iostream>

const int N = 1505, PosInf = 1e9;

struct Edge
{
    int to;
    int next;
} edges[N];

int n, head[N], total;
int f[N][3], cost[N];
bool isNotRoot[N];

void Add(int a, int b)
{
    Edge &e = edges[total];
    e.to = b;
    e.next = head[a];
    head[a] = total++;
}

void Search(int u)
{
    f[u][2] = cost[u];
    for (int i = head[u]; i != -1; i = edges[i].next)
    {
        int v = edges[i].to;
        Search(v);

        f[u][0] += std::min(f[v][1], f[v][2]);
        f[u][2] += std::min(std::min(f[v][0], f[v][1]), f[v][2]);
    }

    f[u][1] = PosInf;
    for (int i = head[u]; i != -1; i = edges[i].next)
    {
        int v = edges[i].to;
        f[u][1] = std::min(f[u][1], f[v][2] + f[u][0] - std::min(f[v][1], f[v][2]));
    }
}

int main()
{
    for (int i = 0; i < N; i++)
        head[i] = -1;

    std::cin >> n;
    for (int i = 1; i <= n; i++)
    {
        int u, w, count;
        std::cin >> u >> w >> count;
        cost[u] = w;
        for (int j = 1; j <= count; j++)
        {
            int v;
            std::cin >> v;
            Add(u, v);
            isNotRoot[v] = true;
        }
    }

    int root = 1;
    while (isNotRoot[root])
        root++;

    Search(root);

    std::cout << std::min(f[root][1], f[root][2]) << std::endl;
    return 0;
}
```