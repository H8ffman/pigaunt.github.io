# 数位型动态规划


## 问题简介

此类问题一般是求在一个特定范围内满足某种特殊性质的数的数量，也有求其它信息的类型。数位型动态规划做法基本类似，大多是预处理-按位枚举范围中的数-计数。

## 技巧

1. 区间转换$[m, n] \Rightarrow f_n - f_{m - 1}$
2. 按位枚举数时先考虑该位小于最高位，再考虑相等的情况（就像一棵二叉树）。

对于技巧2，枚举时的基本模板如下（不能处理去除前导0的情况）

```cpp
int Dp(int n)
{
    if (n == 0)
        return 0; // 视具体情况而定
    
    std::vector<int> nums;
    while (n > 0)
    {
        nums.push_back(n % 10); // 模的数视具体情况而定
        n /= 10; // 同上
    }

    int res = 0, last = 0; // 枚举相等的情况时，可能需要记录上一个数来判断满足的性质
    for (int i = nums.size() - 1; i >= 0; i--)
    {
        int x = nums[i];
        for (int j = 0; j < x; j++)
        {
            if (...) // 满足条件，可能有多级
            {
                res += f[...][...]; // 加上预处理的数
            }
        }
    }
}
```

## 例题

### [LibreOJ-10163-Amount of Degrees](https://loj.ac/p/10163)

如果把这个数表示成B进制，问题就转化成了这个数的B进制下是否有K个1，而$i$位数中有$j$个1的数的数量正好是$i \choose j$。

首先要确定的是，本题的方案数仅仅和整个数中的1有关，因此我们不用循环，只需要判断等于0、等于1和大于1。

要求$0$到$n$中满足条件的数的个数$res$，先预处理出$i \choose j$，然后按位枚举$n$，枚举时记录$n$中1的个数$count$。如果这一位数是0，就到下一位去处理；如果这一位数是1，先让$res$加上预处理的数，再让$count$加一；如果这一位大于1，这一位后面的满足条件的数的个数就可以直接用预处理找到；最后由于$n$本身没有记录，如果满足条件，就让$res$加一。

```cpp
#include <iostream>
#include <vector>

const int N = 33, K = 23;

int x, y, k, b;
int f[N][K];

void Init()
{
    for (int i = 0; i < N; i++)
        for (int j = 0; j <= i; j++)
        {
            if (j == 0)
                f[i][j] = 1;
            else
                f[i][j] = f[i - 1][j] + f[i - 1][j - 1];
        }
}

int Dp(int n)
{
    if (n == 0)
        return 0;

    std::vector<int> nums;
    while (n > 0)
    {
        nums.push_back(n % b);
        n /= b;
    }

    int res = 0, count = 0;
    for (int i = nums.size() - 1; i >= 0; i--)
    {
        int x = nums[i];
        if (x > 0)
        {
            res += f[i][k - count];
            if (x > 1)
            {
                if (k - count - 1 >= 0)
                    res += f[i][k - count - 1];
                break;
            }
            else
            {
                count++;
                if (count > k)
                    break;
            }
        }

        if (i == 0 && count == k)
            res++;
    }

    return res;
}

int main()
{
    std::cin >> x >> y >> k >> b;
    Init();
    std::cout << Dp(y) - Dp(x - 1) << std::endl;
    return 0;
}
```

### [LibreOJ-10164-数字游戏](https://loj.ac/p/10164)

预处理：用$f_{i, j}$表示有$i$位，且第一位是$j$的不降数的个数。$f_{i, j} = \sum f_{i - 1, k} (j \ge k)$。

求$1$到$n$满足条件的数的数量时，枚举每一位$x$并记录上一位$last$，先找到数$j$使$last \le j < x$并加上预处理的答案，然后考虑$j = x$的情况（进入下一位）。但要注意，如果$n$本身再某一位不满足不降数了，就不必再进行枚举。

```cpp
#include <iostream>
#include <vector>

const int N = 15;

int f[N][10];

void Init()
{
    for (int i = 0; i <= 9; i++)
        f[1][i] = 1;

    for (int i = 2; i < N; i++)
        for (int j = 0; j <= 9; j++)
            for (int k = j; k <= 9; k++)
                f[i][j] += f[i - 1][k];
}

int Dp(int n)
{
    if (n == 0)
        return 1;

    std::vector<int> nums;
    while (n > 0)
    {
        nums.push_back(n % 10);
        n /= 10;
    }

    int res = 0, last = 0;
    for (int i = nums.size() - 1; i >= 0; i--)
    {
        int x = nums[i];
        for (int j = last; j < x; j++)
            res += f[i + 1][j];

        if (x < last)
            break;

        last = x;

        if (i == 0)
            res++;
    }

    return res;
}

int main()
{
    Init();

    int l, r;
    while (std::cin >> l >> r)
        std::cout << Dp(r) - Dp(l - 1) << std::endl;
    return 0;
}
```

### [洛谷-P2657-[SCOI2009] Windy数](https://www.luogu.com.cn/problem/P2657)

方法类似，先预处理，再按位枚举。

但本题需要注意的是：**不包含前导0**。这要求我们判断包含前导0的情况并进行特殊处理：
   1. 在枚举过程中，第一位数不能为0；
   2. 枚举结束后，重新考虑有前导0的情况。

```cpp
#include <iostream>
#include <vector>
#include <cmath>

const int N = 15;

int f[N][10];

void Init()
{
    for (int i = 0; i <= 9; i++)
        f[1][i] = 1;

    for (int i = 2; i < N; i++)
        for (int j = 0; j <= 9; j++)
            for (int k = 0; k <= 9; k++)
                if (abs(j - k) >= 2)
                    f[i][j] += f[i - 1][k];
}

int Dp(int n)
{
    if (n == 0)
        return 0;

    std::vector<int> nums;
    while (n > 0)
    {
        nums.push_back(n % 10);
        n /= 10;
    }

    int res = 0, last = -2;
    for (int i = nums.size() - 1; i >= 0; i--)
    {
        int x = nums[i];
        for (int j = (i == nums.size() - 1); j < x; j++)
            if (abs(j - last) >= 2)
                res += f[i + 1][j];

        if (abs(x - last) < 2)
            break;
        last = x;

        if (i == 0)
            res++;
    }

    // 有前导0
    for (int i = 1; i < nums.size(); i++)
        for (int j = 1; j <= 9; j++)
            res += f[i][j];

    return res;
}

int main()
{
    Init();
    int l, r;
    std::cin >> l >> r;
    std::cout << Dp(r) - Dp(l - 1) << std::endl;
    return 0;
}
```

### [LibreOJ-10166-数字游戏](https://loj.ac/p/10166)

```cpp
#include <iostream>
#include <cstring>
#include <vector>

const int N = 13, M = 110;

int MOD;
int f[N][10][M];

int PosMod(int n, int p)
{
    return (n % p + p) % p;
}

void Init()
{
    memset(f, 0, sizeof(f));

    for (int i = 0; i <= 9; i++)
        f[1][i][PosMod(i, MOD)]++;

    for (int i = 2; i < N; i++)
        for (int j = 0; j <= 9; j++)
            for (int k = 0; k < MOD; k++)
                for (int x = 0; x <= 9; x++)
                    f[i][j][k] += f[i - 1][x][PosMod(k - j, MOD)];
}

int Dp(int n)
{
    if (n == 0)
        return 1;

    std::vector<int> nums;
    while (n > 0)
    {
        nums.push_back(n % 10);
        n /= 10;
    }

    int res = 0, last = 0;
    for (int i = nums.size() - 1; i >= 0; i--)
    {
        int x = nums[i];
        for (int j = 0; j < x; j++)
            res += f[i + 1][j][PosMod(-last, MOD)];

        last += x;

        if (i == 0 && last % MOD == 0)
            res++;
    }

    return res;
}

int main()
{
    int l, r;
    while (std::cin >> l >> r >> MOD)
    {
        Init();
        std::cout << Dp(r) - Dp(l - 1) << std::endl;
    }

    return 0;
}
```

### [LibreOJ-10167-不要62](https://loj.ac/p/10167)

```cpp
#include <iostream>
#include <vector>

const int N = 7 + 2;

int f[N][10];

void Init()
{
    for (int i = 0; i <= 9; i++)
        if (i != 4)
            f[1][i] = 1;

    for (int i = 2; i < N; i++)
        for (int j = 0; j <= 9; j++)
            if (j != 4)
                for (int k = 0; k <= 9; k++)
                    if (k != 4 && !(j == 6 && k == 2))
                        f[i][j] += f[i - 1][k];
}

int Dp(int n)
{
    if (n == 0)
        return 1;

    std::vector<int> nums;
    while (n > 0)
    {
        nums.push_back(n % 10);
        n /= 10;
    }

    int res = 0, last = 0;
    for (int i = nums.size() - 1; i >= 0; i--)
    {
        int x = nums[i];
        for (int j = 0; j < x; j++)
            if (j != 4 && !(last == 6 && j == 2))
                res += f[i + 1][j];

        if (x == 4 || (last == 6 && x == 2))
            break;

        last = x;

        if (i == 0)
            res++;
    }

    return res;
}

int main()
{
    Init();
    int l, r;
    while (true)
    {
        std::cin >> l >> r;
        if (l == 0 && r == 0)
            return 0;

        std::cout << Dp(r) - Dp(l - 1) << std::endl;
    }
    return 0;
}
```

### [LibreOJ-10168-恨7不成妻](https://loj.ac/p/10168)

本题的难点在于要求所有数的平方和。这需要我们记录数的个数、和以及平方和。

```cpp
#include <iostream>
#include <vector>

const int N = 18 + 2, MOD = 1e9 + 7;

struct Pack
{
    int count, sum, pow;
} f[N][10][7][7];

int t7[N], tMod[N];

int PosMod(long long n, int p)
{
    return (n % p + p) % p;
}

void Init()
{
    t7[0] = tMod[0] = 1;
    for (int i = 1; i < N; i++)
    {
        t7[i] = t7[i - 1] * 10 % 7;
        tMod[i] = (long long)tMod[i - 1] * 10 % MOD;
    }

    for (int i = 0; i <= 9; i++)
    {
        if (i == 7)
            continue;
        Pack &v = f[1][i][i % 7][i % 7];
        v.count++;
        v.sum += i;
        v.pow += i * i;
    }

    for (int i = 2; i < N; i++)
        for (int j = 0; j <= 9; j++)
        {
            if (j == 7)
                continue;
            for (int a = 0; a < 7; a++)
                for (int b = 0; b < 7; b++)
                    for (int k = 0; k <= 9; k++)
                    {
                        if (k == 7)
                            continue;
                        Pack &p = f[i][j][a][b];
                        Pack &l = f[i - 1][k][PosMod(a - j * t7[i - 1], 7)][PosMod(b - j, 7)];
                        p.count = (p.count + l.count) % MOD;
                        p.sum = (p.sum + (long long)j * tMod[i - 1] % MOD * l.count % MOD + l.sum) % MOD;
                        p.pow = (p.pow +
                                 (long long)j * j * tMod[i - 1] % MOD * tMod[i - 1] % MOD * l.count % MOD +
                                 (long long)2 * j * tMod[i - 1] % MOD * l.sum % MOD + l.pow) % MOD;
                    }
        }
}

Pack Match(int i, int j, int notA, int notB)
{
    int count = 0, sum = 0, pow = 0;
    for (int a = 0; a < 7; a++)
        for (int b = 0; b < 7; b++)
        {
            if (a == notA || b == notB)
                continue;
            Pack &v = f[i][j][a][b];
            count = (count + v.count) % MOD;
            sum = (sum + v.sum) % MOD;
            pow = (pow + v.pow) % MOD;
        }
    return {count, sum, pow};
}

long long Dp(long long n)
{
    if (n == 0)
        return 0;

    long long backup = n % MOD;
    std::vector<int> nums;
    while (n > 0)
    {
        nums.push_back(n % 10);
        n /= 10;
    }

    long long res = 0;
    long long lastA = 0, lastB = 0;
    for (int i = nums.size() - 1; i >= 0; i--)
    {
        int x = nums[i];
        for (int j = 0; j < x; j++)
        {
            if (j == 7)
                continue;
            int notA = PosMod(-lastA % 7 * t7[i + 1], 7);
            int notB = PosMod(-lastB, 7);

            Pack v = Match(i + 1, j, notA, notB);

            res = (res +
                   (lastA % MOD) * (lastA % MOD) % MOD * tMod[i + 1] % MOD * tMod[i + 1] % MOD * v.count % MOD +
                   2 * (lastA % MOD) % MOD * tMod[i + 1] % MOD * v.sum % MOD + v.pow) % MOD;
        }

        if (x == 7)
            break;
        lastA = lastA * 10 + x;
        lastB += x;

        if (i == 0 && lastA % 7 != 0 && lastB % 7 != 0)
            res = (res + backup * backup) % MOD;
    }

    return res;
}

int main()
{
    Init();
    int T;
    std::cin >> T;
    for (int test = 1; test <= T; test++)
    {
        long long l, r;
        std::cin >> l >> r;
        std::cout << PosMod(Dp(r) - Dp(l - 1), MOD) << std::endl;
    }
    return 0;
}
```
