---
title: "数列极限"
date: 2022-02-10T16:50:42+08:00

tags : [
  "数学"
]

categories: [
  "数学"
]

---

# 数列极限

## 什么是一个数列

其实数列就是一个映射 $f:\mathbb{N} \rightarrow \mathbb{R}$，下标到值一一对应，也就是一个函数。当然下标可以从 $1$ 开始，值也不一定是实数，因此可以有灵活的变化，如 $g: \mathbb{N^+} \rightarrow \mathbb{C}$ 也可以表示一个数列。

本文为了统一，下面给出定义时下标均从 $0$ 开始。

为了表示方便，我们把 $f(n)$ 记作 $a_n$，把 $f$ 记作 $\{a_n\}_{n=0}^{+\infty}$，也可以简写成 $a_n$。

### 子数列

直观地看，就类似子序列，从原来的数列里拿几个数出来，组成一个数列。举个例子，$1, 2, 3, 4, 5, \dots$ 的一个子数列就是 $1, 3, 5, \dots$（奇数列）。

要严谨地定义，对于映射 $f:\mathbb{N} \rightarrow \mathbb{R}$（数列），它的子数列是一个映射 $g: L \rightarrow \mathbb{R}$，满足 $|L| = +\infty, L \subset \mathbb{N}$ 且 $\forall n \in L, f(n) = g(n)$
。

这里你可能已经发现了一个问题：从原数列里面取一些数，怎么编号表示呢？我们发现，上面的定义里，是使用了原来的编号的。

这里对于数列 $\{a_n\}_{n=0}^{+\infty}$，它的子数列表示为 $\{a_{n_k}\}_{k=0}^{+\infty}$，容易发现 $n_k \ge k$，因为就算 $a_1$ 到 $a_{k-1}$ 都要（排满），也只会有 $k - 1$ 个数。

## 数列的极限

什么是数列的极限？举个例子，现在有一个数列 $\{a_n\}_{n=0}^{+\infty} = 1, \frac 1 2, \frac 1 3, \dots$，直观地看，随着 $n$ 的增大，$a_n$ 的值越来越小，感觉上越来越趋近于 $0$。

但是怎么给出一个严谨的定义呢？

**柯西ε-δ语言定义** 对于数列 $\{a_n\}_{n=0}^{+\infty}$，若 $\exist b \in \mathbb{R}, \forall \epsilon > 0, \exist N > 0$，使得 $\forall n > N, |a_n - b| < \epsilon$，则 $\{a_n\}_{n=0}^{+\infty}$ 存在极限，且 $b$ 是它的极限，记作：

$$
\lim_{n \rightarrow +\infty} a_n = b
$$

其实这个定义的意思很清楚，不管 $\epsilon$ 有多小（大于0），总能找到和 $b$ 距离在 $\epsilon$ 内的 $a_n$，因此这个数列是无限趋近于 $b$ 的。

### 几个问题

#### **所有数列都有极限吗？**

否。举个例子，$a_n = (-1)^n$ 就不满足。可以发现，这个数列写出来就是 $-1, 1, -1, 1, \dots$，一直在 $0$ 的周围抖，这就没有极限。

怎么证明？把柯西ε-δ语言定义的条件全部反过来，对于数列 $\{a_n\}_{n=0}^{+\infty}$，若 $\forall b \in \mathbb{R}, \exist \epsilon > 0, \forall N > 0$，使得 $\exist n > N, |a_n - b| \ge \epsilon$，则 $\{a_n\}_{n=0}^{+\infty}$ 不存在极限。

这要求我们找一个（些） $\epsilon$ 和一个（些）$n$ 满足上面的条件。对于 $a_n = (-1)^n$，我们取 $\epsilon \in (1, 0]$，取 $n_1, n_2$ 使得 $a_{n_1} = a_{n_2} = 1$，如果存在极限，$|1 - b| < \epsilon$，根据三角形不等式 $|a| + |b| \ge |a \pm b|$，$2 \le |1 - b| + |1 - b| < 2\epsilon$，即 $\epsilon > 1$，与前面的条件矛盾，因此这个数列不存在极限。

另外，如果一个数列有极限，那么称这个数列是**收敛**的，否则称这个数列是**发散**的。

#### **一个数列可以有多个极限吗？**

否。直观地看，应该是没有的，总不能一会儿大一会儿小吧？

怎么证明？依然利用三角形不等式，假设 $b, c$ 是数列 $\{a_n\}_{n=0}^{+\infty}$ 的两个极限，那么对于 $\forall \epsilon > 0$，$\exist N_1 > 0$，使得 $\forall n > N_1, |a_n - b| < \epsilon$ 同时 $\exist N_2 > 0$，使得 $\forall n > N_2, |a_n - c| < \epsilon$，那么取 $n_0 > \max\{N_1, N_2\}$，$|c - b| < |a_{n_0} - b| + |a_{n_0} - c| < 2\epsilon$，所以 $|c - b| < 2\epsilon$，又因为正数 $\epsilon$ 可以任意地小，所以只有 $b = c$ 才能使上式满足。

因此，如果一个数列有极限，那这个极限就是唯一的。

#### **如果一个数列有极限，它的任何一个子数列也有同样的极限吗？**

是。还是直观地看，一个子数列既然是从原数列取出部分值构成的，那么趋向原数列的极限应该更“快”一些，这样看，子数列应当有同样的极限。

怎么证明？我们要利用上面说到的 $n_k \ge k$。如果数列 $\{a_n\}_{n=0}^{+\infty}$ 存在极限 $b$，满足 $\forall \epsilon > 0, \exist N > 0$ 使得 $\forall n > N, |a_n - b| < \epsilon$，它的一个子数列是 $\{a_{n_k}\}_{k=0}^{+\infty}$，那么取 $K = N$，此时对于 $\forall k > K$，因为 $n_k \ge k$，所以 $n_k > K$，也就是 $n_k > N$，因此 $\forall k > K, |a_{n_k} - b| < \epsilon$ 是成立的。也就是说，它们的极限都是 $b$。

现在你可能已经发现，证明一个数列不存在极限还有另一条路径。还是拿 $a_n = (-1)^n$ 为例，它的奇数列 $a_n = -1$ 的极限是 $1$，而它的偶数列 $a_n = 1$ 的极限是 $-1$，由于只要原数列存在极限，它的任何一个子数列都有同样的极限，这里的两个极限不同，不满足上述条件，因此原数列没有极限。

## 数列级数

先看一个说法：

> **刘翔追不上乌龟**
> 
> 假如乌龟在刘翔前 $d$ 米，然后二者同时起跑，刘翔速度 $v_1$，乌龟速度 $v_2$（$v_1 > v_2$），刘翔若想追上乌龟，那么就必须先跑到乌龟最开始在的位置。可等到刘翔花 $t_1$ 的时间跑到那个位置后，乌龟已经向前了一段距离，此时要追上那就必须要花 $t_2$ 的时间跑到乌龟现在的距离，而等刘翔再次赶到时乌龟又会向前一段距离。以此类推，刘翔比乌龟跑得快，那么二者之间的距离只会无限缩短，刘翔永远追不上乌龟。

这么说似乎很有道理，但我们很清楚，追上只需要 $t = \frac {d} {v_1 - v_2}$。

这是为什么呢？我们要清楚，**无限个数加在一起不一定是无限的（不确定的）**。

要解释这个东西，我们引入**数列级数**：对于数列 $\{a_n\}_{n=0}^{+\infty}$，它的数列级数是：

$$
\sum_{n=0}^{+\infty} a_n = a_0 + a_1 + \dots
$$

这就是无限个数加在一起，可是怎么求呢？我们只知道数列的极限的定义。那么就来转化一下：

定义上式的值为数列 $\{s_n\}_{n=0}^{+\infty}$ 的极限，其中 $s_n = \sum_{k=0}^n a_k$，也就是一个前缀和。如果 $s_n$ 有极限，$s_n$ 的极限就自然是上式的和，如果没有，上式求不出和。

利用上面的结论，我们现在可以证明开始的 $t_1 + t_2 + \dots$ 就等于 $t = \frac {d} {v_1 - v_2}$：由于 $t_n = \frac {d} {v_1} \times (\frac {v_2} {v_1})^{n-1}$，那么得到：

$$
s_n = \frac {d} {v_1} \times \sum_{i=1}^n (\frac {v_2} {v_1})^{i-1} \\ = \frac {d} {v_1} \times \frac {1 - (\frac {v_2} {v_1})^{n+1}} {1 - \frac {v_2} {v_1}}
$$

因为 $0 < \frac {v_2} {v_1} < 1$，所以：

$$
\lim_{n \rightarrow +\infty} (\frac {v_2} {v_1})^{n+1} = 0
$$

因此：

$$
\sum_{n=1}^{+\infty} t_n = \lim_{n \rightarrow +\infty} s_n = \frac {d} {v_1 - v_2} = t
$$

所以刘翔追得上乌龟。