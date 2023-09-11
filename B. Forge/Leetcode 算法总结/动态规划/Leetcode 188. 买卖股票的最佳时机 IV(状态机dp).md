#### [188. 买卖股票的最佳时机 IV](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/)

给定一个整数数组 `prices` ，它的第 `i` 个元素 `prices[i]` 是一支给定的股票在第 `i` 天的价格，和一个整型 `k` 。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 `k` 笔交易。也就是说，你最多可以买 `k` 次，卖 `k` 次。

**注意：** 2你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1：**

```
输入：k = 2, prices = [2,4,1]
输出：2
解释：在第 1 天 (股票价格 = 2) 的时候买入，在第 2 天 (股票价格 = 4) 的时候卖出，这笔交易所能获得利润 = 4-2 = 2 。
```

**示例 2：**

```
输入：k = 2, prices = [3,2,6,5,0,3]
输出：7
解释：在第 2 天 (股票价格 = 2) 的时候买入，在第 3 天 (股票价格 = 6) 的时候卖出, 这笔交易所能获得利润 = 6-2 = 4 。
     随后，在第 5 天 (股票价格 = 0) 的时候买入，在第 6 天 (股票价格 = 3) 的时候卖出, 这笔交易所能获得利润 = 3-0 = 3 。
```

**请注意：由于最后未持有股票，手上的股票一定会卖掉，所以 j-1 可以是在买股票的时候，也可以是在卖股票的时候，这两种写法都是可以的。**

```python
class Solution:
    def maxProfit(self, k: int, prices: List[int]) -> int:
        n = len(prices)
        @cache
        def dfs(i: int, j: int, hold: bool) -> int:
            if j < 0:
                return -inf
            if i < 0:
                return -inf if hold else 0
            if hold:
                return max(dfs(i - 1, j, True), dfs(i - 1, j - 1, False) - prices[i])
            return max(dfs(i - 1, j, False), dfs(i - 1, j, True) + prices[i])
        return dfs(n - 1, k, False)
```



```c++
class Solution {
public:
    int maxProfit(int k, vector<int>& prices) {
        // int n = prices.size(), dp[n + 1][k + 2][2];
        // memset(dp, -0x3f3f3f, sizeof(dp));
        // for(int i = 1; i < k + 2; ++i) dp[0][i][0] = 0;
        // for(int i = 0; i < n; ++i) {
        //     for(int j = 1; j <= k + 1; ++j) {
        //         dp[i + 1][j][0] = max(dp[i][j][0], dp[i][j][1] + prices[i]);
        //         dp[i + 1][j][1] = max(dp[i][j][1], dp[i][j - 1][0] - prices[i]);
        //     }
        // }
        // return dp[n][k + 1][0];
         int n = prices.size(), dp[k + 2][2];
        memset(dp, -0x3f3f3f, sizeof(dp));
        for(int i = 1; i < k + 2; ++i) dp[i][0] = 0;
        for(int i = 0; i < n; ++i) {
            for(int j = k + 1; j > 0; --j) {
                dp[j][0] = max(dp[j][0], dp[j][1] + prices[i]);
                dp[j][1] = max(dp[j][1], dp[j - 1][0] - prices[i]);
            }
        }
        return dp[k + 1][0];
    }
};
```

### 思考题

如果改成「恰好」完成 *k* 笔交易要怎么做？

递归到 *i*<0 时，只有 j=0 才是合法的，**j>0 是不合法的**。

```python
# 恰好
class Solution:
    def maxProfit(self, k: int, prices: List[int]) -> int:
        # 递推
        n = len(prices)
        f = [[[-inf] * 2 for _ in range(k + 2)] for _ in range(n + 1)]
        f[0][1][0] = 0  # 只需改这里
        for i, p in enumerate(prices):
            for j in range(1, k + 2):
                f[i + 1][j][0] = max(f[i][j][0], f[i][j][1] + p)
                f[i + 1][j][1] = max(f[i][j][1], f[i][j - 1][0] - p)
        return f[-1][-1][0]

        # 记忆化搜索
        # @cache
        # def dfs(i: int, j: int, hold: bool) -> int:
        #     if j < 0:
        #         return -inf
        #     if i < 0:
        #         return -inf if hold or j > 0 else 0
        #     if hold:
        #         return max(dfs(i - 1, j, True), dfs(i - 1, j - 1, False) - prices[i])
        #     return max(dfs(i - 1, j, False), dfs(i - 1, j, True) + prices[i])
        # return dfs(n - 1, k, False)
```

如果改成「至少」完成 *k* 笔交易要怎么做？

递归到「至少 0 次」时，它等价于「交易次数没有限制」，那么这个状态的计算方式和 [122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/solution/shi-pin-jiao-ni-yi-bu-bu-si-kao-dong-tai-o3y4/) 是一样的。

```python
# 至少
class Solution:
    def maxProfit(self, k: int, prices: List[int]) -> int:
        # 递推
        n = len(prices)
        f = [[[-inf] * 2 for _ in range(k + 1)] for _ in range(n + 1)]
        f[0][0][0] = 0
        for i, p in enumerate(prices):
            f[i + 1][0][0] = max(f[i][0][0], f[i][0][1] + p)
            f[i + 1][0][1] = max(f[i][0][1], f[i][0][0] - p)  # 无限次
            for j in range(1, k + 1):
                f[i + 1][j][0] = max(f[i][j][0], f[i][j][1] + p)
                f[i + 1][j][1] = max(f[i][j][1], f[i][j - 1][0] - p)
        return f[-1][-1][0]

        # 记忆化搜索
        # @cache
        # def dfs(i: int, j: int, hold: bool) -> int:
        #     if i < 0:
        #         return -inf if hold or j > 0 else 0
        #     if hold:
        #         return max(dfs(i - 1, j, True), dfs(i - 1, j - 1, False) - prices[i])
        #     return max(dfs(i - 1, j, False), dfs(i - 1, j, True) + prices[i])
        # return dfs(n - 1, k, False)
```

**如果这题改为有一个初始资金，比如10000，然后连续N天第i天每一股的股票价格分别是prices[i]，最大交易次数是K（买卖一共算一次，买卖可以分开）,求怎么买卖收益最高，应该怎么改，今天机试碰到这样的题了。**

>如果允许交易小数股，在这题的基础上，把初始值 0 改成初始资金，把 +- 改成 \*，最后返回答案的时候减去初始资金。

#### [122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

**不限制次数**

给你一个整数数组 `prices` ，其中 `prices[i]` 表示某支股票第 `i` 天的价格。

在每一天，你可以决定是否购买和/或出售股票。你在任何时候 **最多** 只能持有 **一股** 股票。你也可以先购买，然后在 **同一天** 出售。

返回 *你能获得的 **最大** 利润* 。

**示例 1：**

```
输入：prices = [7,1,5,3,6,4]
输出：7
解释：在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6 - 3 = 3 。
     总利润为 4 + 3 = 7 。
```

**示例 2：**

```
输入：prices = [1,2,3,4,5]
输出：4
解释：在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5 - 1 = 4 。
     总利润为 4 。
```

**示例 3：**

```
输入：prices = [7,6,4,3,1]
输出：0
解释：在这种情况下, 交易无法获得正利润，所以不参与交易可以获得最大利润，最大利润为 0 。
```

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        // vector<vector<int>> dp(n + 1, vector<int>(2, 0));
        // dp[0][1] = INT_MIN;
        // for(int i = 1; i <= n; ++i) {
        //     dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i - 1]);
        //     dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i - 1]);
        // }
        // return dp[n][0];
        int d1 = 0, d2 = INT_MIN;
        for(int i = 0; i < n; ++i) {
            int new_d1 = max(d1, d2 + prices[i]);
            d2 = max(d2, d1 - prices[i]);
            d1 = new_d1;
        }
        return d1;
    }
};
```

##### 相关题目

[122. 买卖股票的最佳时机 II]( https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/solution/shi-pin-jiao-ni-yi-bu-bu-si-kao-dong-tai-o3y4/)

[309. 最佳买卖股票时机含冷冻期]( https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/solution/shi-pin-jiao-ni-yi-bu-bu-si-kao-dong-tai-0k0l/ )

[188. 买卖股票的最佳时机 IV]( https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/solution/shi-pin-jiao-ni-yi-bu-bu-si-kao-dong-tai-kksg/)

课后作业： 

[121. 买卖股票的最佳时机]( https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/ )

[123. 买卖股票的最佳时机 III]( https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)

[714. 买卖股票的最佳时机含手续费](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

[1911. 最大子序列交替和 ](https://leetcode.cn/problems/maximum-alternating-subsequence-sum/)



#### [309. 最佳买卖股票时机含冷冻期](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

给定一个整数数组`prices`，其中第 `prices[i]` 表示第 `*i*` 天的股票价格 。

设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:

- 卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。

**注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

**示例 1:**

```
输入: prices = [1,2,3,0,2]
输出: 3 
解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]
```

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        // vector<vector<int>> dp(n + 2, vector<int>(2));
        // dp[1][1] = INT_MIN;
        // for(int i = 0; i < n; ++i) {
        //     dp[i + 2][0] = max(dp[i + 1][0], dp[i + 1][1] + prices[i]);
        //     dp[i + 2][1] = max(dp[i + 1][1], dp[i][0] - prices[i]);
        // }
        // return dp[n + 1][0];
        int pre = 0, f0 = 0, f1 = INT_MIN;
        for(int &p : prices) {
            int new_f0 = max(f0, f1 + p);
            f1 = max(f1, pre - p);
            pre = f0;
            f0 = new_f0;
        }
        return f0;
    }
};
```



#### [123. 买卖股票的最佳时机 III](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)

给定一个数组，它的第 `i` 个元素是一支给定的股票在第 `i` 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 **两笔** 交易。



```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        // int k = 2;
        // int n = prices.size(), dp[k + 2][2];
        // memset(dp, -0x3f3f3f, sizeof(dp));
        // for(int i = 1; i < k + 2; ++i) dp[i][0] = 0;
        // for(int i = 0; i < n; ++i) {
        //     for(int j = k + 1; j > 0; --j) {
        //         dp[j][0] = max(dp[j][0], dp[j][1] + prices[i]);
        //         dp[j][1] = max(dp[j][1], dp[j - 1][0] - prices[i]);
        //     }
        // }
        // return dp[k + 1][0];
        int n = prices.size(), dp[4][2];
        memset(dp, -0x3f3f3f3f, sizeof(dp));
        for(int i = 1; i < 4; ++i) dp[i][0] = 0;
        for(int i = 0; i < n; ++i) {
            for(int j = 3; j > 0; --j) {
                dp[j][0] = max(dp[j][0], dp[j][1] + prices[i]);
                dp[j][1] = max(dp[j][1], dp[j - 1][0] - prices[i]);
            }
        }
        return dp[3][0];
    }
};
```

#### [714. 买卖股票的最佳时机含手续费](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

给定一个整数数组 `prices`，其中 `prices[i]`表示第 `i` 天的股票价格 ；整数 `fee` 代表了交易股票的手续费用。

你可以无限次地完成交易，但是你每笔交易都需要付手续费。如果你已经购买了一个股票，在卖出它之前你就不能再继续购买股票了。

返回获得利润的最大值。

**注意：**这里的一笔交易指买入持有并卖出股票的整个过程，每笔交易你只需要为支付一次手续费。

**示例 1：**

```
输入：prices = [1, 3, 2, 8, 4, 9], fee = 2
输出：8
解释：能够达到的最大利润:  
在此处买入 prices[0] = 1
在此处卖出 prices[3] = 8
在此处买入 prices[4] = 4
在此处卖出 prices[5] = 9
总利润: ((8 - 1) - 2) + ((9 - 4) - 2) = 8
```

**示例 2：**

```
输入：prices = [1,3,7,5,10,3], fee = 3
输出：6
```

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int n = prices.size();
        // vector<vector<int>> dp(n + 1, vector<int>(2, 0));
        // dp[0][1] = -0x3f3f3f;
        // for(int i = 1; i <= n; ++i) {
        //     dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i - 1] - fee);
        //     dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i - 1]);
        // }
        // return dp[n][0];
        int f0 = 0, f1 = -prices[0];
        for(int i = 0; i < n; ++i) {
            int t = f0;
            f0 = max(f1 + prices[i] - fee, f0);
            f1 = max(f1, t - prices[i]);
        }
        return f0;
    }
};
```

