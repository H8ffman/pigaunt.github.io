# 区间型动态规划


## 石子合并问题

有若干个石子排成一行，每个石子有一定的质量，现在要将它们合并成一堆，每一次合并产生的价值是两堆石子的质量和（只能合并相邻的石子），求最大/最小价值。

以最小值为例，对于这个问题，如果我们用$f_{i, j}$表示从$i$到$j$的代价最小值，那么这一块区域可以被分割成两个小的区域，通过不断调整分割点，我们就可以找到$f_{i, j}$，转移方程如下：
$$
f_{i, j} = \min(f_{i, j}, f_{i, k} + f_{k + 1, j} + s[j] - s[i - 1]) (i \le k < j)
$$
初始值：
$$
f_{i, i} = 0
$$
时间复杂度$O(n ^ 3)$。

如果石子排成了一个环呢？

## 环形石子合并问题

一般来说，有下面几种转换方法：

1. 把环变成链，枚举那个缺口，时间复杂度$O(n ^ 4)$；
2. 考虑到变成环之后，问题转换为求$n$条链上的石子合并问题，那么可以把长度为$n$的链拆开，复制一次，变成长度为$2n$的链，用石子合并的方法求出$f_{i, j}$后，求出$\min(f_{i, i + n - 1})$，时间复杂度$O(8n ^ 3)$；

显然第二种更好。

[洛谷-P1880-[NOI1995] 石子合并](https://www.luogu.com.cn/problem/P1880)

```cpp
#include <iostream>

const int N = 205, PosInf = 1e9;

int n;
int a[N], s[N], f[N][N], g[N][N];

int main()
{
    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++)
        {
            if (i == j)
                f[i][i] = g[i][i] = 0;
            else
            {
                f[i][j] = PosInf;
                g[i][j] = -PosInf;
            }
        }

    std::cin >> n;
    for (int i = 1; i <= n; i++)
    {
        std::cin >> a[i];
        a[i + n] = a[i];
    }
    for (int i = 1; i <= n * 2; i++)
        s[i] = s[i - 1] + a[i];

    for (int len = 1; len <= n; len++)
        for (int l = 1; l + len - 1 <= n * 2; l++)
        {
            int r = l + len - 1;
            for (int k = l; k < r; k++)
            {
                f[l][r] = std::min(f[l][r], f[l][k] + f[k + 1][r] + s[r] - s[l - 1]);
                g[l][r] = std::max(g[l][r], g[l][k] + g[k + 1][r] + s[r] - s[l - 1]);
            }
        }

    int wMin = PosInf, wMax = -PosInf;
    for (int l = 1; l <= n; l++)
    {
        wMin = std::min(wMin, f[l][l + n - 1]);
        wMax = std::max(wMax, g[l][l + n - 1]);
    }
    std::cout << wMin << std::endl;
    std::cout << wMax << std::endl;
    return 0;
}
```

### 问题拓展

[AcWing-320-能量项链](https://www.acwing.com/problem/content/322/)

本题和前面的合并略有不同，待合并的两个区间不是由点构成，而是由点与点之间的空隙构成。因此区间长度至少是3，至多是$n + 1$，且断点$k$不能和左端点相等。

```cpp
#include <iostream>

const int N = 205, PosInf = 1e9;

int n;
int a[N], f[N][N];

int main()
{
    std::cin >> n;
    for (int i = 1; i <= n; i++)
    {
        std::cin >> a[i];
        a[i + n] = a[i];
    }

    for (int len = 3; len <= n + 1; len++)
        for (int l = 1; l + len - 1 <= n * 2; l++)
        {
            int r = l + len - 1;
            for (int k = l + 1; k < r; k++)
                f[l][r] = std::max(f[l][r], f[l][k] + f[k][r] + a[l] * a[k] * a[r]);
        }

    int wMax = -PosInf;
    for (int l = 1; l <= n; l++)
        wMax = std::max(wMax, f[l][l + n]);
    std::cout << wMax << std::endl;
    return 0;
}
```

[LibreOJ-10149-凸多边形的划分](https://loj.ac/p/10149)

可以将一个多边形的划分拆为三部分：左，中（三角形），右。最小价值即为这三者的价值和，于是可以发现转移方程和能量项链类似：
$$
f_{i, j} = \min(f_{i, j}, f_{i, k} + f{k, j} + w_i w_k w_j)
$$
注意，本题数据较大，需要做高精度。

```cpp
#include <iostream>
#include <cstring>

const int N = 55, M = 35, PosInf = 1e9;

int n, w[N];
long long f[N][N][M];

void Add(long long a[], long long b[])
{
    static long long c[M];
    memset(c, 0, sizeof(c));
    for (int i = 0, t = 0; i < M; i++)
    {
        t += a[i] + b[i];
        c[i] = t % 10;
        t /= 10;
    }
    memcpy(a, c, sizeof(c));
}

void Mul(long long a[], long long b)
{
    static long long c[M];
    memset(c, 0, sizeof(c));
    long long t = 0;
    for (int i = 0; i < M; i++)
    {
        t += a[i] * b;
        c[i] = t % 10;
        t /= 10;
    }
    memcpy(a, c, sizeof(c));
}

int Comp(long long a[], long long b[])
{
    for (int i = M - 1; i >= 0; i--)
        if (a[i] > b[i])
            return 1;
        else if (a[i] < b[i])
            return -1;
    return 0;
}

int main()
{
    std::cin >> n;
    for (int i = 1; i <= n; i++)
        std::cin >> w[i];

    long long temp[M];
    for (int len = 3; len <= n + 1; len++)
        for (int l = 1; l + len - 1 <= n; l++)
        {
            int r = l + len - 1;
            f[l][r][M - 1] = 1;
            for (int k = l + 1; k < r; k++)
            {
                memset(temp, 0, sizeof(temp));
                temp[0] = w[l];
                Mul(temp, w[k]);
                Mul(temp, w[r]);
                Add(temp, f[l][k]);
                Add(temp, f[k][r]);
                if (Comp(f[l][r], temp) > 0)
                    memcpy(f[l][r], temp, sizeof(temp));
            }
        }

    int k = M - 1;
    while (k > 0 && f[1][n][k] == 0)
        k--;
    for (int i = k; i >= 0; i--)
        std::cout << f[1][n][i];
    std::cout << std::endl;
    return 0;
}
```

[AcWing-479-加分二叉树](https://www.acwing.com/problem/content/481/)

这个题乍一看似乎和动态规划没有关系，但我们仔细分析“中序”，可以发现，正如区间中用点分割，中序遍历中，左右子树和根正好可以通过区间型动态规划的经典处理方式划分。因此用$f_{i, j}$表示中序遍历为输入数组的$i$到$j$位的二叉树中加分的最大值，但由于要输出具体方案，还要用$g_{i, j}$表示中序遍历为输入数组的$i$到$j$位的二叉树的根的序号。

```cpp
#include <iostream>

const int N = 35;

int n, w[N];
int f[N][N], g[N][N];

void Print(int l, int r)
{
    if (l > r)
        return;

    int root = g[l][r];
    std::cout << root << " ";
    Print(l, root - 1);
    Print(root + 1, r);
}

int main()
{
    std::cin >> n;
    for (int i = 1; i <= n; i++)
        std::cin >> w[i];

    for (int len = 1; len <= n; len++)
        for (int l = 1; l + len - 1 <= n; l++)
        {
            int r = l + len - 1;
            if (len == 1)
            {
                f[l][r] = w[l];
                g[l][r] = l;
            }
            else
            {
                for (int k = l; k <= r; k++)
                {
                    int subl = (k == l) ? 1 : f[l][k - 1];
                    int subr = (k == r) ? 1 : f[k + 1][r];
                    int score = subl * subr + w[k];
                    if (f[l][r] < score)
                    {
                        f[l][r] = score;
                        g[l][r] = k;
                    }
                }
            }
        }

    std::cout << f[1][n] << std::endl;
    Print(1, n);
    return 0;
}
```

[AcWing-321-棋盘分割](https://www.acwing.com/problem/content/323/)

本题的划分分为横向和纵向，同时区间也由一维扩展到了二维，因此实现时可以用记忆化搜索的方式减少代码量。用$f_{x1, y1, x2, y2, k}$表示$(x1, y1)$到$(x2, y2)$这个二维区间分割$k$次的最小均方差，那么这个区间可以横向或纵向划分，且可以任意选取划分后的两个子区间，按照这个逻辑，可以写出下面的代码：

```cpp
#include <iostream>
#include <iomanip>
#include <cmath>
#include <cstring>

const int m = 8, M = 9, N = 20, PosInf = 1e9;

int n, s[M][M];
double f[M][M][M][M][N];
double average;

double Calc(int x1, int y1, int x2, int y2)
{
    double sum = s[x2][y2] - s[x2][y1 - 1] - s[x1 - 1][y2] + s[x1 - 1][y1 - 1];
    return (sum - average) * (sum - average) / n;
}

double Search(int x1, int y1, int x2, int y2, int k)
{
    double &v = f[x1][y1][x2][y2][k];
    if (v >= 0)
        return v;
    if (k == 1)
        return v = Calc(x1, y1, x2, y2);

    v = PosInf;
    for (int i = x1; i < x2; i++)
    {
        v = std::min(v, Search(x1, y1, i, y2, k - 1) + Calc(i + 1, y1, x2, y2));
        v = std::min(v, Search(i + 1, y1, x2, y2, k - 1) + Calc(x1, y1, i, y2));
    }
    for (int i = y1; i < y2; i++)
    {
        v = std::min(v, Search(x1, y1, x2, i, k - 1) + Calc(x1, i + 1, x2, y2));
        v = std::min(v, Search(x1, i + 1, x2, y2, k - 1) + Calc(x1, y1, x2, i));
    }

    return v;
}

int main()
{
    std::cin >> n;
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= m; j++)
        {
            std::cin >> s[i][j];
            s[i][j] += s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1];
        }

    memset(f, -1, sizeof(f));
    average = (double)s[m][m] / n;

    std::cout << std::fixed << std::setprecision(3) << sqrt(Search(1, 1, 8, 8, n)) << std::endl;
    return 0;
}
```

[洛谷-P1436-棋盘分割](https://www.luogu.com.cn/problem/P1436)

这个跟上面一题其实是一样的，经过数学推导，上题求均方差，也就是求平方和，但均方差可以直接求。

```cpp
#include <iostream>
#include <cstring>

const int m = 8, M = 9, N = 20, PosInf = 1e9;

int n, s[M][M];
double f[M][M][M][M][N];

int Calc(int x1, int y1, int x2, int y2)
{
    double sum = s[x2][y2] - s[x2][y1 - 1] - s[x1 - 1][y2] + s[x1 - 1][y1 - 1];
    return sum * sum;
}

double Search(int x1, int y1, int x2, int y2, int k)
{
    double &v = f[x1][y1][x2][y2][k];
    if (v >= 0)
        return v;
    if (k == 1)
        return v = Calc(x1, y1, x2, y2);

    v = PosInf;
    for (int i = x1; i < x2; i++)
    {
        v = std::min(v, Search(x1, y1, i, y2, k - 1) + Calc(i + 1, y1, x2, y2));
        v = std::min(v, Search(i + 1, y1, x2, y2, k - 1) + Calc(x1, y1, i, y2));
    }
    for (int i = y1; i < y2; i++)
    {
        v = std::min(v, Search(x1, y1, x2, i, k - 1) + Calc(x1, i + 1, x2, y2));
        v = std::min(v, Search(x1, i + 1, x2, y2, k - 1) + Calc(x1, y1, x2, i));
    }

    return v;
}

int main()
{
    std::cin >> n;
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= m; j++)
        {
            std::cin >> s[i][j];
            s[i][j] += s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1];
        }

    memset(f, -1, sizeof(f));

    std::cout << Search(1, 1, 8, 8, n) << std::endl;
    return 0;
}
```



## 区间型动态规划的两种实现方式

### 迭代式（推荐）

```cpp
for (int len = 1; len <= n; len++)
    for (int l = 1; l + len - 1 <= n; l++)
    {
        r = l + len - 1;
        for (int k = l; k < r; k++)
            // do something
    }
```

### 记忆化搜索式（适合状态定义较为复杂的情况）

如果状态定义中数组维数太多，需要些很多重循环，这时可以用记忆化搜索的形式来写。
