---
title: "矩阵乘法及应用"
date: 2022-03-18T15:36:15+08:00

tags : [
  "数学"
]

categories: [
  "数学"
]

---

# 矩阵乘法及应用

## 肤浅地认识下矩阵

至少从表面上看，矩阵是一个二维数组。矩阵的加减法就是在相同的位置上进行加减，即：

$$
C_{i, j} = A_{i, j} \pm B_{i, j}
$$

其中矩阵 $A$，$B$ 和 $C$ 的行数和列数都是相同的。

而矩阵的乘法显得有些不同，如果 $A$ 是 $n \times m$ 矩阵，$B$ 是 $m \times p$ 矩阵，那么 $C = A \times B$ 是 $n \times p$ 矩阵，且：

$$
C_{i, j} = \sum_{k = 1}^m A_{i, k} \times B_{k, j}
$$

和加减法相同位置操作不同，$C$ 中每个数 $x$ 是由 $A$ 中与 $x$ 同行的数和 $B$ 中与 $x$ 同列的数相乘的和，这实际上是在体现一种**变换**，把 $A$ 的一行看作一个向量（可以先理解成一行数，并把某一列的数称作**某一维**的数），那么 $A \times B$ 就是让 $A$ 中某一行的一个向量按照矩阵 $B$ 所体现的变换规则变换成另一个向量，因此让每一维的数去乘 $B$ 中每一维所需变换的值，就比较有道理了（由于本人水平太低，没法解释得很清楚，建议阅读神犇孟岩老师的 [理解矩阵](https://blog.csdn.net/myan/article/details/647511)）。

而我们利用矩阵乘法，正是通过构造矩阵来体现变换，从而实现一维状态的递推。而矩阵乘法满足交换律，这使得我们在需要多次递推时，将快速幂应用于矩阵乘法中，从而加速递推。

## 两个例题

### [AcWing-205-斐波那契](https://www.acwing.com/problem/content/207/)

本题是经典的求斐波那契数列第 $n$ 项的问题，但数据范围扩大到了 $10 ^ 9$，并且有多组数据，$O(n)$ 是会超时的。结合前面提到的矩阵，我们可以思考如何将斐波那契数列递推公式 $f_i = f_{i - 1} + f_{i - 2}$ 转换为向量 $f$ 与矩阵 $A$ 相乘的形式。

由于某一项实际上只与前两项相关，我们只需要在向量 $f$ 中保留两项，即 $f = (f_{i - 1}, f_i)$，这时我们要造出下一个 $f = (f_i, f_{i + 1})$，那么新的 $f$ 中第二个位置应当是原 $f$ 中两个数的和，为了保留它们，我们把 $A$ 的第二列构造为 $1$；而 $f$ 中第一个位置只需要保留原 $f$ 中的后一项，因此我们把 $A$ 的第一列构造为 $(0, 1)$，这样，原来的递推过程转化为了：

$$
f' = f \times \begin{pmatrix} 0 & 1 \cr 1 & 1 \end{pmatrix}
$$

这样我们可以对上述公式应用快速幂：矩阵乘法是 $O(n ^ 3)$ 的，在本题中 $n = 2$，复杂度为常数，因此总共的时间复杂度是 $O(\log_2 n)$，可以通过本题。

参考代码：

```cpp
#include <iostream>

class Matrix
{
public:
    const static int N = 5, MOD = 10000;
    int val[N][N], line, col;

    Matrix(int line, int col)
        : line(line), col(col)
    {
        for (int i = 0; i < line; i++)
            for (int j = 0; j < col; j++)
                val[i][j] = 0;
    }
    
    // Matrix operator+(const Matrix& mat) const
    // {
    //     if (line != mat.line || col != mat.col)
    //         return Matrix(0, 0);
    //     Matrix res(line, col);
    //     for (int i = 0; i < line; i++)
    //         for (int j = 0; j < col; j++)
    //             res.val[i][j] = ((long long)val[i][j] + mat.val[i][j]) % MOD;
    //     return res;
    // }

    // Matrix operator-(const Matrix& mat) const
    // {
    //     if (line != mat.line || col != mat.col)
    //         return Matrix(0, 0);
    //     Matrix res(line, col);
    //     for (int i = 0; i < line; i++)
    //         for (int j = 0; j < col; j++)
    //             res.val[i][j] = ((long long)val[i][j] - mat.val[i][j]) % MOD;
    //     return res;
    // }

    Matrix operator*(const Matrix& mat) const
    {
        if (col != mat.line)
            return Matrix(0, 0);
        Matrix res(line, mat.col);
        for (int i = 0; i < line; i++)
            for (int j = 0; j < mat.col; j++)
                for (int k = 0; k < col; k++)
                    res.val[i][j] = ((long long)res.val[i][j] + (long long)val[i][k] * mat.val[k][j]) % MOD;
        return res;
    }

    Matrix operator^(int k) const
    {
        if (line != col)
            return Matrix(0, 0);
        Matrix res(line, col), mat = (*this);
        for (int i = 0; i < line; i++)
            res.val[i][i] = 1;
        while (k > 0)
        {
            if (k & 1)
                res = (res * mat);
            mat = (mat * mat);
            k >>= 1;
        }
        return res;
    }
};

int n;
Matrix f(1, 2), A(2, 2);

int main()
{
    while (true)
    {
        std::cin >> n;
        if (n == -1)
            return 0;
            
        f.val[0][0] = 0, f.val[0][1] = 1;
        A.val[0][0] = 0, A.val[0][1] = 1, A.val[1][0] = 1, A.val[1][1] = 1;
    
        A = (A ^ n);
        f = (f * A);
    
        std::cout << f.val[0][0] << std::endl;
    }
    return 0;
}
```

### [AcWing-206-石头游戏](https://www.acwing.com/problem/content/208/)

现在是一个二维的棋盘了，是不是就不符合前面所说的一维递推了呢？确实，但是我们发现 $0 \le m, n \le 8$，因此可以采用二维化一维，即令 $\mathrm{Index}(x, y) = (x - 1) \times m + y$，给每个坐标一个一维的数字编号。

这时题目中的变换就不难构造了。首先，上下左右的移动，以向上为例，可以设置变换矩阵 $A$ 的第 $\mathrm{Index}(x, y)$ 行，第 $\mathrm{Index}(x - 1, y)$ 列（$x > 1$）为 $1$。

那么添加石头呢？我们需要一个不与其它坐标冲突的位置作为石头来源，这里让 $A_{0, 0}$ 为 $1$，为其它格子提供石头。如果要放 $i$ 个石头，就让 $A$ 的第 $0$ 行，第 $\mathrm{Index}(x, y)$ 列为 $i$。同时，为了保证原来的石头还在，$A$ 的第 $\mathrm{Index}(x, y)$ 行，第 $\mathrm{Index}(x, y)$ 列应当为 $1$。

$A$ 的其它位置均设置为 $0$。

这时仔细想想会发现，丢弃石头其实不需要考虑，因为只有上面两类变换保留了石头。

不过本题还有一个时间维度，只需要构造多个矩阵满足不同时刻的变换即可。这里有一个偷懒的地方：由于操作序列长度不超过 $6$，所有长度的最小公倍数最大就是 $60$，所以把前 $60$ 个时刻的变换矩阵求出来，后面一定都是可以复用的，省去了求最小公倍数的步骤。

最后，使用快速幂加速递推，我们便可以通过本题。

要特别注意的是：矩阵中的“$1$”（单位矩阵，只对于行数和列数相同的有定义）不是全为 $0$ 的矩阵，而是主对角线（左上到右下）为 $1$ 的矩阵（意义是保留向量每一维上的原有值），千万不要忘记初始化了。

参考代码：

```cpp
#include <iostream>

const int N = 8 + 5, K = 60;

class Matrix
{
public:
    const static int L = N * N;
    int line, col;
    long long val[L][L];

    Matrix() {}
    Matrix(int line, int col)
        : line(line), col(col)
    {
        for (int i = 0; i < line; i++)
            for (int j = 0; j < col; j++)
                val[i][j] = 0;
    }

    Matrix operator*(const Matrix& mat) const
    {
        if (col != mat.line)
            return Matrix(0, 0);
        Matrix res(line, mat.col);
        for (int i = 0; i < line; i++)
            for (int j = 0; j < mat.col; j++)
                for (int k = 0; k < col; k++)
                    res.val[i][j] += val[i][k] * mat.val[k][j];
        return res;
    }

    Matrix operator^(int k) const
    {
        if (line != col)
            return Matrix(0, 0);
        Matrix res(line, col), mat = (*this);
        for (int i = 0; i < line; i++)
            res.val[i][i] = 1;
        while (k > 0)
        {
            if (k & 1)
                res = (res * mat);
            mat = (mat * mat);
            k >>= 1;
        }
        return res;
    }
};

int n, m, len, T, act;
int opt[N][N];
Matrix trans[K + 5], transd, f;
std::string op[N];

inline int GetIndex(int x, int y) { return (x - 1) * n + y; }

void Init()
{
    transd = Matrix(len + 1, len + 1);
    f = Matrix(1, len + 1);
    f.val[0][0] = 1;
    // 周期最大就是 60，懒得求最小公倍数了。
    for (int ts = 0; ts < K; ts++)
    {
        trans[ts] = Matrix(len + 1, len + 1);
        trans[ts].val[0][0] = 1;
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= m; j++)
            {
                int pos = GetIndex(i, j);
                std::string& lop = op[opt[i][j]];
                char sop = lop[ts % lop.length()];
                if (sop >= '0' && sop <= '9')
                {
                    trans[ts].val[0][pos] = sop - '0';
                    trans[ts].val[pos][pos] = 1;
                }
                else if (sop == 'N' && i > 1)
                    trans[ts].val[pos][GetIndex(i - 1, j)] = 1;
                else if (sop == 'S' && i < n)
                    trans[ts].val[pos][GetIndex(i + 1, j)] = 1;
                else if (sop == 'W' && j > 1)
                    trans[ts].val[pos][GetIndex(i, j - 1)] = 1;
                else if (sop == 'E' && j < m)
                    trans[ts].val[pos][GetIndex(i, j + 1)] = 1;
            }
    }
}

long long Solve()
{
    int r = T % K;
    int d = T / K;
    // 一定要初始化 transd，否则乘出来都是 0。
    for (int i = 0; i <= len; i++)
        transd.val[i][i] = 1;
    for (int ts = 0; ts < K; ts++)
        transd = transd * trans[ts];
    f = f * (transd ^ d);
    for (int ts = 0; ts < r; ts++)
        f = f * trans[ts];
    long long res = 0;
    for (int i = 1; i <= len; i++)
        res = std::max(res, f.val[0][i]);
    return res;
}

int main()
{
    std::ios::sync_with_stdio(false);

    std::cin >> n >> m >> T >> act;
    len = n * m;
    for (int i = 1; i <= n; i++)
    {
        std::string str;
        std::cin >> str;
        for (int j = 1; j <= m; j++)
            opt[i][j] = str[j - 1] - '0';
    }
    for (int i = 0; i < act; i++)
        std::cin >> op[i];
    Init();

    std::cout << Solve() << std::endl;
    return 0;
}
```