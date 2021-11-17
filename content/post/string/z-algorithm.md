---
title: "Z 函数及其求法（扩展 KMP）"
date: 2021-11-17T16:13:34+08:00

tags : [
  "字符串",
  "动态规划"
]

categories: [
  "字符串",
  "动态规划"
]

---

# Z 函数及其求法（扩展 KMP）

## Z 函数

（以下默认字符串下标从`0`开始）。

在 KMP 字符串匹配算法中，使用了$\pi$来表示一个字符串的最长公共前后缀，现在我们介绍一个与公共前缀有关的函数，Z 函数。我们用$z_i$表示一个字符串与它以$i$为起点的后缀的最长公共前缀（LCP）。此外，定义$z_0$为$0$。例如，字符串`abcabca`的$z$值分别为$0, 0, 0, 4, 0, 0, 1$。

## Z 函数的求法

根据 Z 函数的定义，我们可以枚举待求字符串的的每一个后缀，从头开始尝试匹配，得到 Z 函数的值。这样计算的时间复杂度是$O(n^2)$。

参考代码（`std::string`，字符串下标从`0`开始）：
```cpp
void GetZFunc(const std::string str)
{
    int len = str.length();
    for (int i = 1; i < len; i++)
        while (i + z[i] < len && str[z[i]] == str[i + z[i]]) 
            z[i]++;
    return;
}
```

## Z 算法（在$O(n)$时间内求出 Z 函数）

类似 KMP 字符串匹配中求$\pi$和 Manacher 算法中求最长回文时的优化思路，我们来想一想能不能利用已有信息减少暴力匹配的计算次数。

为了方便解释，我们把与$s_0 \dots s_{z_i}$匹配的子串$s_i \dots s_{i + z_i - 1}$称为“匹配子串”。我们计算 Z 的时候记录右端点最靠右的“匹配子串”$s_l \dots s_r$，这时如果$i \le r$，那么$s_i \dots s_r$一定与$s_{i - l} \dots s_{r - l}$相同。此时$z_{i - l}$与$z_i$是相同的。但要注意，如果$z_{i - l} \ge r - i + 1$，那么不能保证$r$后面的字符和$r - l$后面的字符匹配，需要暴力尝试。

如果$i \ge r$，由于没有已知信息可以利用，也需要暴力尝试。

以上过程可以用下面的图表示：
$$
\rlap{
    \underbrace{
    \phantom{
        s_0 \dots s_{i - l} \dots s_i \dots s_{r - l} \dots}}^\text{same}
    } 
    s_0 \dots s_{i - l} \dots 
    \overbrace{
        s_l \dots s_i \dots s_{r - l} \dots s_r}_\text{same} 
\dots
$$

由于$r$只增不减，Z 算法的时间复杂度为$O(n)$。

参考代码（`std::string`，字符串下标从`0`开始）：
```cpp
void ZAlgorithm(const std::string &str)
{
    int len = str.length();
    z[0] = 0;
    for (int i = 1, l = 0, r = 0; i < len; i++)
    {
        int k = z[i - l];
        if (i <= r && k < r - i + 1)
            z[i] = k;
        else
        {
            k = std::max(r - i + 1, 0);
            while (i + k < len && str[i + k] == str[k])
                k++;
            z[i] = k;
        }

        if (i + k - 1 > r)
        {
            l = i;
            r = i + k - 1;
        }
    }
}
```

### Z 算法的扩展

有些时候，我们需要求某个字符串与另一个字符串后缀的最长公共前缀，该问题的解决方法同上，只是匹配时不再进行自身比较，而是文本串（的后缀）和模式串进行比较。

但要注意三个细节：
+ <b>仍然要取`z[i - l]`（这意味着要先对文本串做 Z 算法）。</b>因为上面用于加速算法的性质是针对一个字符串的，不能用新的代替。
+ **匹配时注意边界，文本串和模式串都要判断。**
+ <b>单独处理$z_0$。</b>单个字符串与自己的最大公共前缀一定是它的长度（Z 函数中只是定义为$0$）扩展处理时如果不定义$z_0 = 0$，应当从头枚举计算$z_0$。

参考代码（`std::string`，字符串下标从`0`开始）：
```cpp
// `prev` means the `z` of `text`.
void ExZAlgorithm(const std::string &text, const std::string &pattern)
{
    int tLen = text.length(), pLen = pattern.length();
    z[0] = 0;
    for (int i = 0; i < std::min(tLen, pLen); i++)
    {
        if (text[i] == pattern[i])
            z[0]++;
        else
            break;
    }

    for (int i = 1, l = 0, r = 0; i < tLen; i++)
    {
        int k = prev[i - l]; // not `z`.
        if (i <= r && k < r - i + 1)
            z[i] = k;
        else
        {
            k = std::max(r - i + 1, 0);
            while (i + k < tLen && k < pLen && text[i + k] == pattern[k])
                k++;
            z[i] = k;
        }

        if (i + k - 1 > r)
        {
            l = i;
            r = i + k - 1;
        }
    }
}
```