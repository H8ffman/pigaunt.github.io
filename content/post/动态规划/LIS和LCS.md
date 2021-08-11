title: "LIS和LCS"
date: 2021-08-11T10:50:50+08:00

tags : [
  "动态规划"
]

categories: [
  "动态规划"
]


# LIS和LCS

## 最长上升子序列（LIS）问题

给出一个由数字组成的序列，要求从中挑选尽量多的数，组成一个单调上升的子序列。

要解决这个问题，我们可以这样想：

如果现在有了一个最长上升子序列，我们要在后面加一个数。如果这个数比最长上升子序列的最后一个数大，我们可以将它加入其中；如果更小，那只能去找找稍微短一些的上升子序列，看看能不能加入；实在没有，那就保持现状。

这个思路很清晰，但由于添加的数是否在序列中未知，处理起来比较麻烦。此时我们**要求以加入的数结尾**，再递推就简单许多了，可以写出如下的转移方程：

$$
f_i = \max(1, (a_j < a_i)(f_j + 1))
$$

初始值为$0$，答案$\max(f_i)$

因此我们可以把代码写出来：

```cpp
#include <iostream>

const int N = 1005;

int n;
int a[N], f[N];

int main()
{
	std::cin >> n;
	for (int i = 1; i <= n; i++)
	{
		std::cin >> a[i];
	}
	
	for (int i = 1; i <= n; i++)
	{
		f[i] = 1;
		for (int j = 1; j < i; j++)
		{
			if (a[j] < a[i])
				f[i] = std::max(f[i], f[j] + 1);

		}
	}
	
	int ans = 0;
	for (int i = 1; i <= n; i++)
		ans = std::max(ans, f[i]);
	std::cout << ans << std::endl;
	return 0;
}
```

### 例题

[AcWing-482-合唱队形](https://www.acwing.com/problem/content/484/)

本题求最少同学出列，其实就是求最长上升后下降子序列，然后用总数减就是了。

求最长上升后下降子序列时，只需要分别求上升和下降，但一定要注意，要保证两次递推时，每次控制作为结尾的数要是“峰顶”，我之前图方便，想在一个循环里搞定，就出错了。

AC代码：

```cpp
#include <iostream>

const int N = 1005;

int n, h[N];
int up[N], down[N];

int main()
{
    int n;
    std::cin >> n;
    for (int i = 1; i <= n; i++)
        std::cin >> h[i];
    
    for (int i = 1; i <= n; i++)
    {
        up[i] = 1;
        for (int j = 1; j < i; j++)
            if (h[i] > h[j])
                up[i] = std::max(up[i], up[j] + 1);
    }

    for (int i = n; i > 0; i--)
    {
        down[i] = 1;
        for (int j = n; j > i; j--)
            if (h[i] > h[j])
                down[i] = std::max(down[i], down[j] + 1);
    }

    int res = 0;
    for (int i = 1; i <= n; i++)
        res = std::max(res, up[i] + down[i] - 1);
    std::cout << n - res << std::endl;
    return 0;
}
```

## 最长公共子序列（LCS）问题

给出两个由数字组成的序列，要求从中挑选尽量多的两个序列共有的数，组成子序列。

要考虑这个问题，我们可以顺着刚才的思路，用已知信息推出未知。

代码如下：

```cpp
#include <iostream>

const int N = 1005;

int n, m, d[N][N];
char a[N], b[N];

int main()
{
	std::cin >> n >> m >> a + 1 >> b + 1;
	// getline(std::cin, a);
	// getline(std::cin, b);
	
	for (int i = 1; i <= n; i++)
		for (int j = 1; j <= m; j++)
		{
			d[i][j] = std::max(d[i - 1][j], d[i][j - 1]);
			if (a[i] == b[j])
				d[i][j] = std::max(d[i][j], d[i - 1][j - 1] + 1);
		}
	
	std::cout << d[n][m] << std::endl;
	return 0;
}
```

## 两个问题的联系

我们可以发现，两个问题中状态表示的关键都在“最后一个”，或者说“新加入的一个”，这样处理方便了对转移方向的判断。

### 例题

[AcWing-274-最长公共上升子序列](https://www.acwing.com/problem/content/274/)

```cpp
#include <iostream>

const int N = 3005;

int n, a[N], b[N];
int f[N][N];

int main()
{
    std::cin >> n;
    for (int i = 1; i <= n; i++)
        std::cin >> a[i];
    for (int i = 1; i <= n; i++)
        std::cin >> b[i];
    
    for (int i = 1; i <= n; i++)
    {
        int maxv = 0;
        for (int j = 1; j <= n; j++)
        {
            f[i][j] = f[i - 1][j];
            if (a[i] == b[j])
                f[i][j] = std::max(f[i][j], maxv + 1);
            if (a[i] > b[j])
                maxv = std::max(maxv, f[i][j]);
        }
    }
    
    int res = 0;
    for (int i = 1; i <= n; i++)
        res = std::max(res, f[n][i]);
    std::cout << res << std::endl;
    return 0;
}
```