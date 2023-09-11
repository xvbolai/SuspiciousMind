#### [2305. 公平分发饼干](https://leetcode.cn/problems/fair-distribution-of-cookies/)

给你一个整数数组 `cookies` ，其中 `cookies[i]` 表示在第 `i` 个零食包中的饼干数量。另给你一个整数 `k` 表示等待分发零食包的孩子数量，**所有** 零食包都需要分发。在同一个零食包中的所有饼干都必须分发给同一个孩子，不能分开。

分发的 **不公平程度** 定义为单个孩子在分发过程中能够获得饼干的最大总数。

返回所有分发的最小不公平程度。

**示例 1：**

```
输入：cookies = [8,15,10,20,8], k = 2
输出：31
解释：一种最优方案是 [8,15,8] 和 [10,20] 。
- 第 1 个孩子分到 [8,15,8] ，总计 8 + 15 + 8 = 31 块饼干。
- 第 2 个孩子分到 [10,20] ，总计 10 + 20 = 30 块饼干。
分发的不公平程度为 max(31,30) = 31 。
可以证明不存在不公平程度小于 31 的分发方案。
```


![[Pasted image 20230609143735.png]]
![[Pasted image 20230609143918.png]]

```c++
class Solution {
public:
    int distributeCookies(vector<int>& cookies, int k) {
        int n = cookies.size();
        vector<int> sum(1<<n);
        // for(int i = 1; i < 1<<n; ++i) {
        //     for(int j = 0; j < n; ++j) if(i >> j & 1) {
        //         sum[i] += cookies[j];
        //     }
        // }
        // vector<vector<int>> dp(k, vector<int>(1 << n));
        // dp[0] = sum;
        // for(int i = 1; i < k; ++i) {
        //     for(int j = 1; j < 1 << n; ++j) {
        //         dp[i][j] = INT_MAX;
        //         for(int s = j; s; s = (s - 1) & j) {
        //             dp[i][j] = min(dp[i][j], max(dp[i - 1][j ^ s], sum[s]));
        //         }
        //     }
        // }
        // return dp[k - 1][(1 << n) - 1];
        for(int i = 0; i < n; ++i) {
            for(int j = 0, bit = 1 << i; j < bit; ++j) {
                sum[bit | j] = sum[j] + cookies[i];
            }
        }
        vector<int> dp(sum);
        for(int i = 1; i < k; ++i) {
            for(int j = (1 << n) - 1; j; --j) {
                for(int s = j; s; s = (s - 1) & j) {
                    dp[j] = min(dp[j], max(dp[j ^ s], sum[s]));
                }
            }
        }
        return dp[(1 << n) - 1];
    }
};
```