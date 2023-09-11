### 分组背包

有 ![](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7 "n") $n$ 件物品和一个大小为 $m$ 的背包，第 $i$ 个物品的价值为 $w_{i}$，体积为 $v_i$。同时，每个物品属于一个组，同组内最多只能选择一个物品。求背包能装载物品的最大总价值。

这种题怎么想呢？其实是从「在所有物品中选择一件」变成了「从当前组中选择一件」，于是就对每一组进行一次 0-1 背包就可以了。

再说一说如何进行存储。我们可以将 $t_{k, i}$ 表示第 $k$ 组的第 $i$ 件物品的编号是多少，再用 $cnt_k$ 表示第 $k$ 组物品有多少个。

### 实现

```c++
for (int k = 1; k <= ts; k++)           // 循环每一组
  for (int i = m; i >= 0; i--) // 循环背包容量
    for (int j = 1; j <= cnt[k]; j++)   // 循环该组的每一个物品
      if (i >= w[t[k][j]])  // 背包容量充足
        dp[i] = max(dp[i], dp[i - w[t[k][j]]] + c[t[k][j]]);  // 像0-1背包一样状态转移
```


一张桌子上总共有 `n` 个硬币 **栈** 。每个栈有 **正整数** 个带面值的硬币。

每一次操作中，你可以从任意一个栈的 **顶部** 取出 1 个硬币，从栈中移除它，并放入你的钱包里。

给你一个列表 `piles` ，其中 `piles[i]` 是一个整数数组，分别表示第 `i` 个栈里 **从顶到底** 的硬币面值。同时给你一个正整数 `k` ，请你返回在 **恰好** 进行 `k` 次操作的前提下，你钱包里硬币面值之和 **最大为多少** 。

**示例 1：**

![[Pasted image 20230730122944.png|center]]
```
输入：piles = [[1,100,3],[7,8,9]], k = 2
输出：101
解释：
上图展示了几种选择 k 个硬币的不同方法。
我们可以得到的最大面值为 101 。
```

**示例 2：**

```
输入：piles = [[100],[100],[100],[100],[100],[100],[1,1,1,1,1,1,700]], k = 7
输出：706
解释：
如果我们所有硬币都从最后一个栈中取，可以得到最大面值和。
```

**提示：**

- $n == piles.length$
- $1 <= n <= 1000$
- $1 <= piles[i][j] <= 10^5$
- $1 <= k <= sum(piles[i].length) <= 2000$

```c++
class Solution {
public:
    int maxValueOfCoins(vector<vector<int>>& piles, int k) {
        int sumN = 0;
        vector<int> f(k + 1);
        for(auto &v : piles) {
            int n = v.size();
            for(int i = 1; i < n; ++i)
                v[i] = v[i - 1] + v[i];
            sumN = min(k, n + sumN);
            for(int j = sumN; j; --j) {
                for(int w = 1; w <= min(n, j); ++w) {
                    f[j] = max(f[j], f[j - w] + v[w - 1]);
                }
            }
        }
        return f[k];
    }
};
```