---
title: "数学知识笔记"
subtitle: ""
date: 2022-07-20T19:51:52+08:00
lastmod: 2022-07-20T19:51:52+08:00
draft: false
author: "PigAunt"
authorLink: ""
description: "数学知识笔记"
license: ""
images: []

tags: [
  "数学"
]
categories: [
  "数学"
]
---

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