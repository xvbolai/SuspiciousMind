你有一个凸的 `n` 边形，其每个顶点都有一个整数值。给定一个整数数组 `values` ，其中 `values[i]` 是第 `i` 个顶点的值（即 **顺时针顺序** ）。

假设将多边形 **剖分** 为 `n - 2` 个三角形。对于每个三角形，该三角形的值是顶点标记的**乘积**，三角剖分的分数是进行三角剖分后所有 `n - 2` 个三角形的值之和。

返回 *多边形进行三角剖分后可以得到的最低分* 。

**示例 1：**

![[shape1.jpg]]

```
输入：values = [1,2,3]
输出：6
解释：多边形已经三角化，唯一三角形的分数为 6。
```

**示例 2：**

![[shape2.jpg]]

```
输入：values = [3,7,4,5]
输出：144
解释：有两种三角剖分，可能得分分别为：3*7*5 + 4*5*7 = 245，或 3*4*5 + 3*4*7 = 144。最低分数为 144。
```

**示例 3：**

![[shape3.jpg]]

```
输入：values = [1,3,1,4,1,5]
输出：13
解释：最低分数三角剖分的得分情况为 1*1*3 + 1*1*4 + 1*1*5 + 1*1*1 = 13。
```

![[1680388698-XNaKai-1039-cut.png]]

```c++
class Solution {
public:
    int minScoreTriangulation(vector<int>& values) {
        int n = values.size(), dp[n][n];
        memset(dp, 0, sizeof(dp));
        for(int i = n - 3; i >= 0; --i) {
            for(int j = i + 2; j < n; ++j) {
                int res = INT_MAX;
                for(int k = i + 1; k < j; ++k) {
                    res = min(res, dp[i][k] + dp[k][j] + values[i] * values[j] * values[k]);
                }
                dp[i][j] = res;
            }
        }
        return dp[0][n - 1];
    }
};
```

##### 相关题目 

[516. 最长回文子序列]( https://leetcode.cn/problems/longest-palindromic-subsequence/solution/shi-pin-jiao-ni-yi-bu-bu-si-kao-dong-tai-kgkg/)

[1039. 多边形三角剖分的最低得分](https://leetcode.cn/problems/minimum-score-triangulation-of-polygon/solution/shi-pin-jiao-ni-yi-bu-bu-si-kao-dong-tai-aty6/)

 课后作业： 

[375. 猜数字大小 II](https://leetcode.cn/problems/guess-number-higher-or-lower-ii/)

[1312. 让字符串成为回文串的最少插入次数](https://leetcode.cn/problems/minimum-insertion-steps-to-make-a-string-palindrome/) 

[1771. 由子序列构造的最长回文串的长度](https://leetcode.cn/problems/maximize-palindrome-length-from-subsequences/) 

[1547. 切棍子的最小成本](https://leetcode.cn/problems/minimum-cost-to-cut-a-stick/)

[1000. 合并石头的最低成本](https://leetcode.cn/problems/minimum-cost-to-merge-stones/)

[所有题目+题解汇总](https://github.com/EndlessCheng/codeforces-go/blob/master/leetcode/README.md)

#### [1130. 叶值的最小代价生成树](https://leetcode.cn/problems/minimum-cost-tree-from-leaf-values/)

给你一个正整数数组 `arr`，考虑所有满足以下条件的二叉树：
- 每个节点都有 `0` 个或是 `2` 个子节点。
- 数组 `arr` 中的值与树的中序遍历中每个叶节点的值一一对应。
- 每个非叶节点的值等于其左子树和右子树中叶节点的最大值的乘积。
在所有这样的二叉树中，返回每个非叶节点的值的最小可能总和。这个和的值是一个 32 位整数。

如果一个节点有 0 个子节点，那么该节点为叶节点。

**示例 1：**

![[Pasted image 20230531125603.png|350]]

```
输入：arr = [6,2,4]
输出：32
解释：有两种可能的树，第一种的非叶节点的总和为 36 ，第二种非叶节点的总和为 32 。
```

**示例 2：**

![[Pasted image 20230531125652.png|200]]
```
输入：arr = [4,11]
输出：44
```

**状态转移方程：**
$$ dp[i][j] = \min_{k = i}^{j - 1}dp[i][k] + dp[k + 1][j] + maxnn[i][k]*maxnn[k + 1][j]$$
```c++
class Solution {
public:

    int mctFromLeafValues(vector<int>& arr) {
        int n = arr.size();
        //  vector<vector<int>>  maxnn(n, vector<int>(n, 0)), dp(n, vector<int>(n));
        // function<int(int, int)> dfs = [&](int i, int j) -> int {
        //     if(i == j) {
        //         maxnn[i][j] = arr[i];
        //         return 0;
        //     }
        //     int ans = INT_MAX;
        //     for(int k = i; k < j; ++k) {
        //         ans = min(dfs(i, k) + dfs(k + 1, j) + maxnn[i][k] * maxnn[k + 1][j], ans);
        //         maxnn[i][j] = max(maxnn[i][k], maxnn[k + 1][j]);
        //     }
        //     return ans;
        // };
        // int ans = INT_MAX;
        // for(int i = 0; i < n - 1; ++i) {
        //     ans = min(ans, dfs(0, i) + dfs(i + 1, n - 1) + maxnn[0][i] * maxnn[i + 1][n - 1]);
        // }
        // return ans;
        // 动态规划
        // for(int j = 0; j < n; ++j) {
        //     maxnn[j][j] = arr[j];
        //     dp[j][j] = 0;
        //     for(int i = j - 1; i >= 0; --i) {
        //         int ans = INT_MAX;
        //         for(int k = i; k < j; ++k) {
        //             maxnn[i][j] = max(maxnn[i][k], maxnn[k + 1][j]);
        //             ans = min(ans, dp[i][k] + dp[k + 1][j] + maxnn[i][k] * maxnn[k + 1][j]);              
        //         }
        //         dp[i][j] = ans;
        //     }
        // }
        // return dp[0][n - 1];
        // 仿哈夫曼编码
        int ans = 0, pos = 0;
        while(arr.size() > 1) {
            int minn = INT_MAX;
            for(int i = 0; i < arr.size() - 1; ++i) {
                if(minn > arr[i] * arr[i + 1]) {
                    pos = arr[i] < arr[i + 1] ? i : i + 1;
                    minn = arr[i] * arr[i + 1];
                }
            }
            ans += minn;
            arr.erase(arr.begin() + pos);
        }
        return ans;
    }
};
```

