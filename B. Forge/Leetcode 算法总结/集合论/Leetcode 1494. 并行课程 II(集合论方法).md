#### [1494. 并行课程 II](https://leetcode.cn/problems/parallel-courses-ii/)

给你一个整数 `n` 表示某所大学里课程的数目，编号为 `1` 到 `n` ，数组 `relations` 中， `relations[i] = [xi, yi]`  表示一个先修课的关系，也就是课程 `xi` 必须在课程 `yi` 之前上。同时你还有一个整数 `k` 。

在一个学期中，你 **最多** 可以同时上 `k` 门课，前提是这些课的先修课在之前的学期里已经上过了。

请你返回上完所有课最少需要多少个学期。题目保证一定存在一种上完所有课的方式。

**示例 1：**
![[Pasted image 20230616213232.png|300|center]]

```
输入：n = 4, relations = [[2,1],[3,1],[1,4]], k = 2
输出：3 
解释：上图展示了题目输入的图。在第一个学期中，我们可以上课程 2 和课程 3 。然后第二个学期上课程 1 ，第三个学期上课程 4 。
```

**示例 2：**

![[Pasted image 20230616213300.png|300|center]]
```
输入：n = 5, relations = [[2,1],[3,1],[4,1],[1,5]], k = 2
输出：4 
解释：上图展示了题目输入的图。一个最优方案是：第一学期上课程 2 和 3，第二学期上课程 4 ，第三学期上课程 1 ，第四学期上课程 5 。
```

```c++
class Solution {
public:
    int minNumberOfSemesters(int n, vector<vector<int>>& relations, int k) {
        int pre[n];
        memset(pre, 0, sizeof(pre));
        for(auto &r : relations)    pre[r[1] - 1] |= 1 << (r[0] - 1);
        int u = (1 << n) - 1; // 全集
        // int mem[1 << n];
        // memset(mem, -1, sizeof(mem));
        // function<int(int)> dfs = [&] (int i) -> int {
        //     if( i == 0) return 0;
        //     int &res = mem[i];
        //     if(res != -1) return res;
        //     int i0 = 0, ci = u ^ i;
        //     for(int j = 0; j < n; ++j) {
        //         if(i >> j & 1 && (pre[j] | ci) == ci) {
        //             i0 |= (1 << j);
        //         }
        //     }
        //     if(__builtin_popcount(i0) <= k) return dfs(i ^ i0) + 1;
        //     res = INT_MAX;
        //     for(int j = i0; j; j = (j - 1) & i0) {
        //         if(__builtin_popcount(j) == k) {
        //             res = min(res, dfs(i ^ j) + 1);
        //         }
        //     }
        //     return res;
        // };
        // return dfs(u);
        int dp[1 << n];
        dp[0] = 0;
        for(int i = 1; i < 1 << n; ++i) {
            int i0 = 0, ci = u ^ i;
            for(int j = 0; j < n; ++j) {
                if((i >> j) & 1 && (pre[j] | ci) == ci) {
                    i0 |= (1 << j);
                }
            }
            if(__builtin_popcount(i0) <= k) {
                dp[i] = dp[i ^ i0] + 1;
                continue;
            }
            dp[i] = INT_MAX;
            for(int j = i0; j; j = (j - 1) & i0) if(__builtin_popcount(j) <= k) {
                dp[i] = min(dp[i], dp[i ^ j] + 1);
            }
        }
        return dp[u];
    }
};
```

#### 相似题目（状压 DP）

- [1879. 两个数组最小的异或值之和](https://leetcode.cn/problems/minimum-xor-sum-of-two-arrays/)
- [2172. 数组的最大与和](https://leetcode.cn/problems/maximum-and-sum-of-array/)
- [1125. 最小的必要团队](https://leetcode.cn/problems/smallest-sufficient-team/)，[题解](https://leetcode.cn/problems/smallest-sufficient-team/solution/zhuang-ya-0-1-bei-bao-cha-biao-fa-vs-shu-qode/)
- [1986. 完成任务的最少工作时间段](https://leetcode.cn/problems/minimum-number-of-work-sessions-to-finish-the-tasks/)