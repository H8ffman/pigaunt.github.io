# 数学知识笔记


最近学到的数学知识笔记，如有谬误欢迎指正 :yum:。

## ExGCD

用于解不定方程 $ax + by = \gcd(a, b)$ 的一组整数解。

当 $b = 0$ 时，有一组解 $x = 1, y = 0$；我们研究能不能通过 $b, a \bmod b$ 的一组解得到 $a, b$ 的一组解，因为如果能，那么我们可以在辗转相除的过程中推出 $x, y$。

由于

{{< raw >}}
$$
\begin{cases}
ax' + by' = \gcd(a, b) \\ 
bx + (a \bmod b)y = \gcd(b, a \bmod b) \\
\gcd(a, b) = \gcd(b, a \bmod b)
\end{cases}
$$
{{< /raw >}}

因此

{{< raw >}}
$$
ax' + by' = bx + (a \bmod b)y \\
ax' + by' = bx + (a - b \times \lfloor{\frac a b}\rfloor)y \\ 
ax' + by' = ay + b(x - \lfloor{\frac a b}\rfloor y)
$$
{{< /raw >}}

得到

{{< raw >}}
$$
\begin{cases}
x' = y \\
y' = x - \lfloor{\frac a b}\rfloor y
\end{cases}
$$
{{< /raw >}}

我们在辗转相除的递归过程中不断修改 $x, y$，最后得到要求的一组特解。

参考代码：

```cpp
int ExGcd(int a, int b, int &x, int &y)
{
    if (b == 0)
    {
        x = 1, y = 0;
        return a;
    }
    int d = ExGcd(b, a % b, y, x);
    y -= a / b * x;
    return d;
}
```

现在将问题扩展为求 $ax + by = c$ 的一组特解，根据 Bézout 定理，方程有解的充要条件是 $\gcd(a, b) \mid c$。如果有解，求出 $ax + bx = \gcd(a, b)$ 的一组特解，然后 $x, y$ 同时乘 $\frac {c} {\gcd(a, b)}$ 即可。

## CRT

用于解形式如下的线性同余方程组：

{{< raw >}}
$$
\begin{cases}
    x \equiv a_1 \pmod{p_1} \\ 
    x \equiv a_2 \pmod{p_2} \\ 
    \dots                   \\
    x \equiv a_n \pmod{p_n}
\end{cases}
$$
{{< /raw >}}

其中 $p$ 两两互质。

对于这样一个方程组，我们可以按如下流程构造出一个特解：

1. 令 $P = \prod p_i$；
2. 由于 $p$ 两两互质，我们可以求出 $\frac {P} {p_i}$ 在模 $p_i$ 意义下的逆元 $inv_i$；
3. 令 $x_i = \frac {P} {p_i} \times inv_i$；
4. $x = \sum a_i x_i$。

这个构造是正确的，因为：

{{< raw >}}
$$
\begin{cases}
x_i \equiv 1 \pmod{p_i} \\
x_i \equiv 0 \pmod{p_j} (j \neq i)
\end{cases}
$$
{{< /raw >}}

$x_i$ 乘上 $a_i$， $x$ 是满足所有方程的一个特解。

现在要得到一个通解，$x$ 加上 $k$ 倍的 $\mathrm{lcm}(p_1, p_2, \dots, p_n)$ 即可，而 $p$ 两两互质，所以加上 $k$ 倍的 $P$ 就行了。

### ExCRT

问题升级为不要求模数两两互质，也就是不一定有逆元了，因此我们采用将线性同余方程不断合并的方式，最后得到一个线性同余方程，用 ExGCD 求解（与 CRT 采用的构造法完全不同）。

将两个线性同余方程合并：

{{< raw >}}
$$
\begin{cases}
    x \equiv a_1 \pmod{p_1} \\
    x \equiv a_2 \pmod{p_2}
\end{cases}
$$
{{< /raw >}}

令 $x = k_1 p_1 + a_1 = k_2 p_2 + a_2$，得到 $k_2 p_2 - k_1 p_1 = a_1 - a_2$，这个方程符合 $ax + by = c$ 的形式，可以用 ExGCD 求解。得到 $k_1, k_2$ 后，随便代入一个方程，比如第一个，得到 $x = k_1 p_1 + a_1$，那么：

{{< raw >}}
$$
x \equiv k_1 p_1 + a_1 \pmod{\mathrm{lcm}(p_1, p_2)}
$$
{{< /raw >}}

取模数为 $\mathrm{lcm}(p_1, p_2)$ 的目的同样是把 $x$ 的特解转换成一个通解的形式，并保证余数不变。

不断重复合并的过程，最终得到一个线性同余方程：

{{< raw >}}
$$
x \equiv a \pmod p
$$
{{< /raw >}}

时间复杂度 $\mathrm{O}(n \log_2 p)$。

## BSGS

用于解高次同余方程 $a^x \equiv b \pmod p$（离散对数问题），其中 $(a, p) = 1$。

我们令 $m = \sqrt p$，并令 $x = im - j$（用减法是为了后面不出现负号），那么原方程就变成了：

{{< raw >}}
$$
a^{im - j} \equiv b \pmod p
$$
{{< /raw >}}

由于 $(a, p) = 1$，$a$ 存在模 $p$ 意义下的逆元，因此我们可以把 $a^{im - j}$ 拆了，并在两边同时乘 $a^j$，得到：

{{< raw >}}
$$
a^{im} \equiv b \times a^j \pmod p
$$
{{< /raw >}}

现在我们就避免了同时枚举两个数。分别枚举 $i, j$，一次枚举时将得到的值存表，另一次查表，就能得到满足条件的解。

{{< admonition type=tip title="求最小 / 最大解的方法" open=true >}}
调整枚举顺序：

1. 求最小解：先从小到大枚举 $j$（即存表时如果冲突，保留 $j$ 更大的），再从小到大枚举 $i$，找到就退出；
2. 求最大解：先从大到小枚举 $j$（即存表时如果冲突，保留 $j$ 更小的），再从大到小枚举 $i$，找到就退出。
{{< /admonition >}}

时间复杂度取决于表的实现，如果使用哈希表，则时间复杂度 $\mathrm{O}(\sqrt p)$，如果使用平衡树（`std::map`），则时间复杂度 $\mathrm{O}(\sqrt p \log_2 p)$

参考代码：

```cpp
int BSGS(int a, int b, int p)
{
    if (1 % p == b % p)
        return 0;
    int k = std::sqrt(p) + 1;
    std::map<int, int> hash;
    for (int i = 0, j = b % p; i < k; i++)
    {
        hash[j] = i;
        j = (long long)j * a % p;
    }
    int ak = 1;
    for (int i = 1; i <= k; i++)
        ak = (long long)ak * a % p;
    for (int i = 1, j = ak; i <= k; i++)
    {
        if (hash.count(j))
            return (long long)i * k - hash[j];
        j = (long long)j * ak % p;
    }
    return -PosInf;
}
```

### ExBSGS

问题升级为不保证 $(a, p) = 1$，因此 $a$ 不一定存在模 $p$ 意义下的逆元。

为了解决这个问题，我们取 $d = \gcd(a, p)$。如果 $d = 1$，则套用 BSGS 算法求解；否则在同余方程中将 $d$ 消去（如果 $d \nmid b$，则无解），即：

{{< raw >}}
$$
a^{x - 1} \times \frac a d \equiv \frac b d \pmod{\frac p d}
$$
{{< /raw >}}

再求 $d = \gcd(a, \frac p d)$，重复上述过程，直至 $d = 1$，即我们最终做 BSGS 的高次同余方程是：

{{< raw >}}
$$
a^{x - k} \times \frac {a^k} {\prod d} \equiv \frac {b} {\prod d} \pmod{\frac {p} {\prod d}}
$$
{{< /raw >}}

但要注意，现在我们默认 $x \ge k$ 了，但有可能 $x < k$ 的时候有解。对于这个范围的 $x$，我们直接枚举求解（代入原方程），因为 $k$ 较小，因此并不会降低效率。

参考代码：

```cpp
int ExBSGS(int a, int b, int p)
{
    b = (b % p + p) % p;
    if (1 % p == b % p)
        return 0;
    int x, y, d = ExGcd(a, p, x, y);
    if (d > 1)
    {
        if (b % d != 0)
            return -PosInf;
        ExGcd(a / d, p / d, x, y);
        return ExBSGS(a, (long long)b / d * x % (p / d), p / d) + 1;
    }
    return BSGS(a, b, p);
}
```

### 特殊的 BSGS

BSGS 还可以用来求解其它类型的简单高次方程，这里以矩阵为例：

{{< raw >}}
$$
A^x = B
$$
{{< /raw >}}

只要矩阵 $A$ 有逆元（$A \times A^{-1} = I$），我们可以完全套用 BSGS 的做法，只是将数改为矩阵（包括表中存储的内容）。

上述算法的一个应用是求 Fibonacci 数列在模 $p$ 意义下的最小循环节 $x$，即解这个方程：

{{< raw >}}
$$
{\begin{bmatrix} f_{i - 1} & f_i \end{bmatrix}} \times {\begin{bmatrix} 0 & 1 \cr 1 & 1 \end{bmatrix}}^x = {\begin{bmatrix} f_{i - 1} & f_i \end{bmatrix}}
$$
{{< /raw >}}

令 $A = \begin{bmatrix} 0 & 1 \cr 1 & 1 \end{bmatrix}$，我们发现 $A$ 存在逆元 $\begin{bmatrix} -1 & 1 \cr 1 & 0 \end{bmatrix}$，可以通过上述方法求解，即代入做 BSGS 求最小解即得到最小循环节。

## 原根和阶

### 阶 (order)

对于一个数 $a$ 和模数 $p$，我们称最小的满足下面条件的数 $k$ 是 $a$ 模 $p$ 的阶：$a^k \equiv 1 \pmod p$，写作 $\mathrm{ord}(a) = k$。

阶有一个性质：$\mathrm{ord}(a) \mid \varphi(a)$，因为 $\mathrm{ord}(a)$ 整除任意满足 $a^x \equiv 1 \pmod p$ 的数 $x$（可以利用 $x = k \times \mathrm{ord}(a) + r$ 证明出 $r = 0$），那么要求 $a$ 的阶就可以枚举 $\varphi(a)$ 的因数。

### 原根

对于一个数 $p$，我们称一个满足下面条件的数 $g$ 是它的原根：$g^1, g^2, \dots, g^{\varphi(p)}$ 互不相同，且是 $1$ 到 $p - 1$ 中与 $p$ 互质的数的一个排列。

例如，$3$ 是 $998244353$ 的一个原根。

结合上面阶的定义，$p$ 的原根的阶是 $\varphi(p)$，且阶为 $\varphi(p)$ 的数是 $p$ 的原根。因此要判断一个数是不是原根，我们可以验证它的阶是不是 $\varphi(p)$，但此时可以只枚举 $\frac {\varphi(p)} {x}$（$x$ 是 $p$ 的质因子，暂不能证明）。

经数学家证明，一个数如果存在原根，其最小原根是较小的，因此可以用枚举找原根。

## 筛法

如果要找一个范围内的质数，我们采用枚举每一个数并验证的方法是不好的，因为时间复杂度太高；但如果反过来，我们把不满足条件的删掉，就可以大大加速求解过程。

从 $2$ 开始枚举。当我们枚举到一个没有被标记的数时，我们确定它是一个质数；确定完一个数是不是质数后，我们标记它的倍数，流程如下：

```cpp
int primes[N], total;
bool del[N];

void GetPrimes(int maxVal)
{
    for (int i = 2; i <= maxVal; i++)
    {
        if (!del[i])
            primes[++total] = i;
        
        for (int j = 1; i * primes[j] <= maxVal && j <= total; j++)
            del[i * primes[j]] = true;
    }
}
```

上面的过程叫 Eratosthenes 筛法，时间复杂度 $\mathrm{O}(n \log_2 \log_2 n)$。我们发现时间复杂度的瓶颈在于一个数会被多个数标记，但我们希望一个数只被筛一次，因此我们规定：一个数只被它的最小质因子筛掉，于是有了下面的流程：

```cpp
int primes[N], total;
bool del[N];

void GetPrimes(int maxVal)
{
    for (int i = 2; i <= maxVal; i++)
    {
        if (!del[i])
            primes[++total] = i;
        
        for (int j = 1; i * primes[j] <= maxVal && j <= total; j++)
        {
            del[i * primes[j]] = true;
            if (i % primes[j] == 0)
                break;
        }
    }
}
```

增加的两行代码完成了这样一个事情：枚举 $\mathrm{primes}_j$ 就是在枚举最小质因子，一旦发现 $i$ 中包含了 $i \times \mathrm{primes}_j$ 的最小质因子 $\mathrm{primes}_j$，那么说明后面的数中 $\mathrm{primes}_j$ 不再是它们的最小质因子，因此直接退出。可以证明这样会不重不漏地筛掉所有合数（我证不来，欢迎指教）。这个过程叫线性筛，时间复杂度 $\mathrm{O}(n)$。

### 线性筛筛积性函数

由于积性函数 $f$ 满足 $\forall \gcd(x, y) = 1, f(xy) = f(x) f(y)$，那么对于一个数 $n = \prod p_i^{c_i}$，我们只需要知道每一个 $p_i^{c_i}$ 的函数值，就能得到 $n$ 的函数值，因此我们对线性筛的过程进行扩充，记录 $g_i$ 表示 $i$ 的最小质因子对应的幂（即 $p_1^{c_1}$），然后分类讨论求 $g_i$：

1. 如果 $i$ 是质数，那么 $g_i = i$；
2. 如果 $\mathrm{primes}_j \mid i$，说明应当累计 {{< raw >}}$i \times \mathrm{primes}_j${{< /raw >}} 的最小质因子，那么 {{< raw >}}$g_{i \times \mathrm{primes}_j} = g_i \times \mathrm{primes}_j${{< /raw >}}；
3. 如果 $\mathrm{primes}_j \nmid i$，说明枚举到了 {{< raw >}}$i \times \mathrm{primes}_j${{< /raw >}} 的最小质因子，那么 {{< raw >}}$g_{i \times \mathrm{primes}_j} = \mathrm{primes}_j${{< /raw >}}。

现在得到了 $g_i$，那么求 $f(i)$ 的过程也可以分类讨论：

1. 如果 $g_i = i$，说明 $i$ 是质数，按 $f$ 的定义直接求（数论函数一般在质数上的函数值都很好求，因为因子只有 $1$ 和自己）；
2. 如果 $\mathrm{primes}_j \nmid i$，那么它们互质，可以根据积性函数的性质，$f(i \times \mathrm{primes}_j) = f(i) \times f(\mathrm{primes}_j)$；
3. 如果 $\mathrm{primes}_j \mid i$，我们继续讨论：
    1. 如果 $g_i = i$，那么这是两个质数相乘的情况，直接求；
    2. 否则说明是往 $i$ 上累加一个质数 $\mathrm{primes}_j$，我们调整成两个质数的函数值相乘的情况，即 $f(i \times \mathrm{primes}_j) = f(\frac {i} {g_i}) \times f(g_i \times \mathrm{primes}_j)$。
   
现在我们以 $f = \mu * \mathrm{id}_k$ 的求值为例，参考代码：

```cpp
int f[N + 5], g[N + 5];
int primes[N + 5], total;

void GetPrimes(int maxVal)
{
    f[1] = g[1] = 1;
    for (int i = 2; i <= maxVal; i++)
    {
        if (g[i] == 0)
        {
            primes[++total] = i;
            f[i] = FastPow(i, K) - 1;
            g[i] = i;
        }

        for (int j = 1; i * primes[j] <= maxVal; j++)
        {
            if (i % primes[j] == 0)
            {
                if (g[i] == i) // p^{nk} * (p^k - 1).
                    f[i * primes[j]] = (long long)f[i] * (f[primes[j]] + 1) % MOD;
                else
                    f[i * primes[j]] = (long long)f[i / g[i]] * f[primes[j] * g[i]] % MOD;
                g[i * primes[j]] = g[i] * primes[j];
                break;
            }

            f[i * primes[j]] = (long long)f[i] * f[primes[j]] % MOD;
            g[i * primes[j]] = primes[j];
        }
    }
}
```

对于一些特殊的积性函数（如 $\varphi$，$\mu$），我们不必这么麻烦，可以利用它自己的特殊性质，修改线性筛的过程求值。

### 杜教筛

有时我们不需要得到积性函数的值，而是它的前缀和，在线性筛 $\mathrm{O}(n)$ 的基础上，杜教筛可以在 $\mathrm{O}(n^{\frac 2 3})$ 的时间内求得积性函数的前缀和。

令 $s(n) = \sum_{i = 1}^n f(i)$，若 $f * g = h$，那么：

{{< raw >}}
$$
\sum_{i = 1}^n h(n) = \sum_{i = 1}^n \sum_{d \mid i} f(d) g(\frac i d) \\
= \sum_{d = 1}^n g(d) \sum_{i = 1}^{\lfloor \frac n d \rfloor} f(i) \\ 
= g(1)s(n) + \sum_{d = 2}^n g(d) s(\lfloor \frac n d \rfloor)
$$
{{< /raw >}}

移项后得到：

{{< raw >}}
$$
g(1)s(n) = \sum_{i = 1}^n h(i) - \sum_{d = 2}^n g(d) s(\lfloor \frac n d \rfloor)
$$
{{< /raw >}}

如果 $g, h$ 的前缀和很好算，那么 $s(n)$ 就好求了。常见的 $f * g = h$ 有 $\mu * \mathrm{I} = \epsilon$，$\varphi * \mathrm{I} = \mathrm{id}$，$\mathrm{id} * \mu = \varphi$（$\mathrm{I}, \mathrm{id}$ 的前缀和都很好算）。

具体流程：先线性筛预处理 $f$ 的前 $\mathrm{O}(\sqrt n)$ 项，再递归实现上述过程，并用一个表存储 $s$ 的值。

下面以求 $\mu, \varphi$ 的前缀和为例（[洛谷 P4213 【模板】杜教筛（Sum）](https://www.luogu.com.cn/problem/P4213)），参考代码：

```cpp
#include <iostream>
#include <map>

const int M = 1e6;

struct Info
{
    long long mu, phi;
    Info()
    {
        mu = phi = 0;
    }
    Info(long long _mu, long long _phi)
        : mu(_mu), phi(_phi) {}

    Info operator+(const Info &x) const
    {
        return {mu + x.mu, phi + x.phi};
    }
    Info operator-(const Info &x) const
    {
        return {mu - x.mu, phi - x.phi};
    }
    Info operator*(const long long x) const
    {
        return {mu * x, phi * x};
    }
} val[M + 5], bSum[M + 5];

int primes[M + 5], total;
bool del[M + 5];
std::map<long long, Info> sum;

void GetPrimes(int maxVal)
{
    val[1].phi = val[1].mu = 1;
    for (int i = 2; i <= maxVal; i++)
    {
        Info &u = val[i];
        if (!del[i])
        {
            primes[++total] = i;
            u.mu = -1;
            u.phi = i - 1;
        }

        for (int j = 1; i * primes[j] <= maxVal; j++)
        {
            del[i * primes[j]] = true;
            Info &v = val[i * primes[j]];
            if (i % primes[j] == 0)
            {
                v.phi = u.phi * primes[j];
                v.mu = 0;
                break;
            }
            v.phi = u.phi * (primes[j] - 1);
            v.mu = -u.mu;
        }
    }

    bSum[1] = val[1];
    for (int i = 2; i <= maxVal; i++)
        bSum[i] = bSum[i - 1] + val[i];
}

Info GetSum(long long x)
{
    if (x <= M)
        return bSum[x];
    if (sum.count(x) > 0)
        return sum[x];
    
    Info res(0, 0);
    res.mu = 1, res.phi = (long long)x * (x + 1) / 2;
    for (long long l = 2, r = 0; l <= x && r <= x; l = r + 1)
    {
        r = x / (x / l);
        res = res - GetSum(x / l) * (r - l + 1);
    }
    sum[x] = res;
    return res;
}

int main()
{
    std::ios::sync_with_stdio(false);

    int T, n;
    std::cin >> T;
    GetPrimes(M);
    for (int test = 1; test <= T; test++)
    {
        std::cin >> n;
        Info res = GetSum(n);
        std::cout << res.phi << ' ' << res.mu << '\n';
    }
    return 0;
}
```
