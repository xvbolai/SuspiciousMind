给你两个整数 `m` 和 `n` ，分别表示一块矩形木块的高和宽。同时给你一个二维整数数组 `prices` ，其中 $prices[i] = [h_i, w_i, price_i]$ 表示你可以以 $price_i$ 元的价格卖一块高为 $h_{i}$ 宽为 $w_i$ 的矩形木块。

每一次操作中，你必须按下述方式之一执行切割操作，以得到两块更小的矩形木块：

- 沿垂直方向按高度 **完全** 切割木块，或
- 沿水平方向按宽度 **完全** 切割木块

在将一块木块切成若干小木块后，你可以根据 `prices` 卖木块。你可以卖多块同样尺寸的木块。你不需要将所有小木块都卖出去。你 **不能** 旋转切好后木块的高和宽。

请你返回切割一块大小为 `m x n` 的木块后，能得到的 **最多** 钱数。

注意你可以切割木块任意次。

**示例 1：**

![[Pasted image 20230801161925.png|center|200]]
```
输入：m = 3, n = 5, prices = [[1,4,2],[2,2,7],[2,1,3]]
输出：19
解释：上图展示了一个可行的方案。包括：
- 2 块 2 x 2 的小木块，售出 2 * 7 = 14 元。
- 1 块 2 x 1 的小木块，售出 1 * 3 = 3 元。
- 1 块 1 x 4 的小木块，售出 1 * 2 = 2 元。
总共售出 14 + 3 + 2 = 19 元。
19 元是最多能得到的钱数。
```

**示例 2：**

![[Pasted image 20230801162008.png|center|200]]
```
输入：m = 4, n = 6, prices = [[3,2,10],[1,4,2],[4,1,3]]
输出：32
解释：上图展示了一个可行的方案。包括：
- 3 块 3 x 2 的小木块，售出 3 * 10 = 30 元。
- 1 块 1 x 4 的小木块，售出 1 * 2 = 2 元。
总共售出 30 + 2 = 32 元。
32 元是最多能得到的钱数。
注意我们不能旋转 1 x 4 的木块来得到 4 x 1 的木块。
```

定义 $f[i][j]$ 表示对一块高 $i$ 宽 $j$ 的木块，切割后能得到的最多钱数。那么答案就是 $f[m][n]$. 

如果直接售卖，则收益为对应的 $price_i$​（如果存在的话）。

如果垂直切割，枚举切割的宽度 $k$，则最大收益为:

$$
max_{k = 1}^{j - 1} f[i][k] + f[i][j - k]
$$
如果水平切割，枚举切割的高度$k$，则最大的收益为
$$
max_{k = 1}^{i - 1}f[k][j] + f[i - k][j]
$$
取上述三种情况的最大值，即为$f[i][j]$。

```c++
class Solution {
public:
    long long sellingWood(int m, int n, vector<vector<int>>& prices) {
        long long f[m + 1][n + 1];
        memset(f, 0, sizeof(f));
        for(auto& v : prices) f[v[0]][v[1]] = v[2];
        for(int i = 1; i <= m; ++i) {
            for(int j = 1; j <= n; ++j) {
                for(int k = 1; k <= j / 2; ++k) f[i][j] = max(f[i][j], f[i][k] + f[i][j - k]);
                for(int k = 1; k <= i / 2; ++k) f[i][j] = max(f[i][j], f[k][j] + f[i - k][j]);
            }
        }
        return f[m][n];
    }
};
```