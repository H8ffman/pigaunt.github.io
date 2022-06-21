---
title: "回文字符串和 Manacher 算法"
date: 2021-11-16T20:03:14+08:00

tags : [
  "字符串",
  "动态规划"
]

categories: [
  "字符串",
  "动态规划"
]

---

## 回文字符串及回文中心表示法

形如$s_0, s_1, \dots,s_{i - 1}, s_i, s_{i - 1} \dots, s_1, s_0$或$s_0, s_1, \dots, s_i, s_i \dots, s_1, s_0$的，正着写和倒着写相同的字符串叫做**回文字符串**。`abcba`、`aabbaa`、`c`都是回文字符串。如果一个字符串的子串正着写和倒着写相同，这个字串叫做**回文子串**。

### 回文中心表示法

用$d_i$表示从$s_i$到以它为回文中心的回文子串的边界的**最长距离**（包括$s_i$）。最长距离意味着边界以内的部分也是回文子串。

例如，`abcba`的$d$可以表示为$1, 1, 3, 1, 1$。

## 求一个字符串的所有$d$

首先可以以每一个点为中心向外拓展，因为枚举每一个点的复杂度是$O(n)$，向外拓展的复杂度也是$O(n)$，因此这样做的时间复杂度是$O(n^2)$。

如果字符串下标从`0`开始，代码可以这样写：
```cpp
for (int i = 0; i < n; i++)
{
    d[i] = 1;
    while (0 <= i - d[i] && i + d[i] < n && str[i - d[i]] == str[i + d[i]])
        d[i]++;
}
```
你可能已经注意到上面的代码处理不了回文中心为空的情况，我们待会儿来考虑这个（因为可以通过插入字符的方法将它转化为回文中心不为空的情况）。

## Manacher 算法

现在要给这个过程加速，我们来看看能不能利用已知信息。

如果待求点$i$前面有回文子串的最右端点$r$大于$i$，我们把这个回文子串的回文中心记为$t$，最左端点记为$l$，最右端点记为$r$，那么$s_t \dots s_i$这一段必然可以以$t$为中心向前面翻折，$i$的对应位置记为$j$，相应的，以$j$为中心的回文子串也与以$i$为中心的回文子串关于$t$对称，那么此时可以确定的$d_i$大小就是$d_j$。当然，以上的结论还有一个限制：以$j$为中心的回文子串的左端点不能超过以$t$为中心的回文子串的左端点$l$，因为不能保证$l$左边仍然可以和$r$右边关于$t$对称。因此我们要设置一个$d_i$转移时的最大值。这个过程可以用下图表示（$\LaTeX$来源：[OI Wiki](https://oi-wiki.org/string/manacher/)）：
{{< raw >}}
$$
\ldots
\overbrace{
    s_l \ldots
    \underbrace{
        s_{j - d_j + 1} \ldots s_j \ldots s_{j + d_j - 1}
    }_\text{palindrome}
    \ldots
    \underbrace{
        s_{i - d_j + 1} \ldots s_i \ldots s_{i + d_j - 1}
    }_\text{palindrome}
    \ldots\ s_r
}^\text{palindrome}
\ldots
$$
{{< /raw >}}

那么$r$右边的部分呢？这时可以用上面的办法向外拓展，最后更新回文子串的最右端点。
由于端点$r$只增不减（类似KMP字符串匹配的原理），该算法的时间复杂度为$O(n)$。
本算法由**Glenn K. Manacher**（~~不是Face Off里的Glenn~~）发明。

参考代码（以`0`为字符串下标起点）：
```cpp
for (int i = 0, l = 0, r = -1; i < len; i++)
{
    int k = 1;
    // `r - i + 1` is the limit.
    if (i <= r)
        k = std::min(d[l + r - i], r - i + 1);

    // `i - k` and `i + k` are the next positions.
    while (i - k >= 0 && i + k < len && str[i - k] == str[i + k])
        k++;

    d[i] = k;

    if (i + k - 1 > r)
    {
        l = i - k + 1;
        r = i + k - 1;
    }
}
```

如果字符串下标从`1`开始，需要做一点调整：
```cpp
for (int i = 1, l = 1, r = 0; i <= n; i++)
{
    int k = 1;
    if (i <= r)
        k = std::min(d[l + r - i], r - i + 1);
        
    while (i - k >= 1 && i + k <= n && str[i - k] == str[i + k])
        k++;
    
    d[i] = k;
    if (i + k - 1 > r)
    {
        l = i - k + 1;
        r = i + k - 1;
    }
}
```

### 回文中心为空的处理办法

我们不想给回文中心为空单独添加一份代码，因此可以在原字符串每个字符之间和首尾添加一个特殊的字符（比如`#`），此时原字符数$n$，添加字符数$n + 1$，新字符串的字符数$2n + 1$，一定是一个奇数。此时回文子串一定每个$d$值减去$1$就是整个回文子串的真正长度（`#`其实代表了字符间的空位）。

添字处理参考代码（C风格字符串，下标从`1`开始）：
```cpp
int initLen = std::strlen(initial + 1);
for (int i = 1; i <= initLen; i++)
{
    str[++n] = '#';
    str[++n] = initial[i];
}
str[++n] = '#';
str[++n] = '\0'; // `n` stands for the length of `str`.
```

使用`std::string`（下标从`0`开始）的参考代码：
```cpp
const std::string unit("#");
int tLen = initial.length();
for (int i = 0; i < tLen; i++)
    str.append(unit + initial[i]);
str.append(unit);
```