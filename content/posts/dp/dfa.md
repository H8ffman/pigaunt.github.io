---
title: "状态机"
date: 2021-08-11T10:50:50+08:00

tags : [
  "动态规划"
]

categories: [
  "动态规划"
]

---

## 状态机是数学模型

有限状态自动机（以下简称状态机），是用来抽象事物运行规则的数学模型，我们可以把状态机看成一个图，结点代表各个状态，边代表状态间的转换。

我们来看一个最简单的例子：自动门。自动门只有开和关两种状态，关可以转换到开，开可以转换到关。

## 状态机模型在动态规划中的运用

状态间的转换可以看作状态转移，因此，使用状态机模型将问题分析清楚后，可以自然地写出转移方程。

例题：[AcWing-1057-股票买卖 IV](https://www.acwing.com/problem/content/1059/)

用$f_{i, j, 0}$表示在第$i$天，交易次数为$j$，不持有股票时的最大收益，$f_{i, j, 1}$表示持有股票的最大收益。

那么我们可以把这两种状态构建成状态机模型：

+ 不持有股票，可以继续不持有，也可以买入
+ 持有股票，可以继续持有，也可以卖出

因此可以写出转移方程：

$$
f_{i, j, 0} = \max(f_{i - 1, j, 0}, f_{i - 1, j, 1} + w_i) \\
f_{i, j, 1} = \max(f_{i - 1, j, 1}, f_{i - 1, j - 1, 0} - w_i)
$$

```cpp
#include <iostream>
#include <cstring>

const int N = 1e5 + 5, M = 105;

int n, m;
int w[N];
int f[N][M][2];

int main()
{
    std::cin >> n >> m;
    for (int i = 1; i <= n; i++)
        std::cin >>w[i];
    
    memset(f, -0x3f, sizeof(f));
    for (int i = 0; i <= n; i++)
        f[i][0][0] = 0;

    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
        {
            f[i][j][0] = std::max(f[i - 1][j][0], f[i - 1][j][1] + w[i]);
            f[i][j][1] = std::max(f[i - 1][j][1], f[i - 1][j - 1][0] - w[i]);
        }
    
    int res = 0;
    for (int i = 0; i <= m; i++)
        res = std::max(res, f[n][i][0]);
    std::cout << res << std::endl;
    return 0;
}
```