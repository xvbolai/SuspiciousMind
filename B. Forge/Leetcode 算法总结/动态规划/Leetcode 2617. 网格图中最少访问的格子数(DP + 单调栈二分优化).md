#hard 

给你一个下标从 **0** 开始的 `m x n` 整数矩阵 `grid` 。你一开始的位置在 **左上角** 格子 `(0, 0)` 。

当你在格子 `(i, j)` 的时候，你可以移动到以下格子之一：

- 满足 `j < k <= grid[i][j] + j` 的格子 `(i, k)` （向右移动），或者
- 满足 `i < k <= grid[i][j] + i` 的格子 `(k, j)` （向下移动）。

请你返回到达 **右下角** 格子 `(m - 1, n - 1)` 需要经过的最少移动格子数，如果无法到达右下角格子，请你返回 `-1` 。

**示例 1：**

![[Pasted image 20231004231518.png]]

```txt
**输入：**grid = [[3,4,2,1],[4,2,3,1],[2,1,0,0],[2,4,0,0]]
**输出：**4
**解释：**上图展示了到达右下角格子经过的 4 个格子。
```

**示例 2：**

![[Pasted image 20231004231543.png]]

```txt
**输入：**grid = [[3,4,2,1],[4,2,1,1],[2,1,1,0],[3,4,1,0]]
**输出：**3
**解释：**上图展示了到达右下角格子经过的 3 个格子。
```

**示例 3：**

![[Pasted image 20231004231604.png]]

```txt
**输入：**grid = [[2,1,0],[1,0,0]]
**输出：**-1
**解释：**无法到达右下角格子。
```

**提示：**

- `m == grid.length`
- `n == grid[i].length`
- `1 <= m, n <= 105`
- `1 <= m * n <= 105`
- `0 <= grid[i][j] < m * n`
- `grid[m - 1][n - 1] == 0`

### 思路

暴力做法是从 $(0, 0)$出发，向右/向下尝试移动到每个满足要求的格子。假设移动到了$(i, j)$，那么问题就变成从 $(i, j)$ 出发，到右下角的最少移动格子数。这是一个和原问题相似的子问题，启发我们用递归来思考。

定义$f[i][j]$表示从$(i, j)$出发，到右下角最少移动格子数。

设$g=grid[i][j]$，有

$$
f[i][j]=min\{min^{j + g}_{k = j + 1}f[i][k], min^{i + g}_{k = i + 1}f[k][j]\} + 1
$$
$i$和$j$均倒序遍历。答案为$f[0][0]$。时间复杂度$O(mn(m + n))$

由于涉及到**区间查询和单点更新**这两个操作，可以选择**线段树**优化。

此题可以采用另外一种方式，**单调栈**。

对于$min^{j + g}_{{k = j + 1}}f[i][k]$ 来说，在倒序遍历$j$时，$k$的左边界$j + 1$ 是在单调减小的，我们可以用一个$f$值底小顶大的单调栈来维护$f[i][k]$及其下表$k$。由于是倒序遍历，单调栈中的下标是底大顶小的，因此可以采用二分查找的方式查找不超过$j + g$的下标$k$，对应的$f[i][k]$就是$[j + 1, j + g]$范围内的最小值。

同理，对于$min^{i + g}_{k = i + 1}f[k][j]$也是需要单调栈来维护。

```c++
class Solution {
public:
    int minimumVisitedCells(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size(), mn;
        vector<vector<pair<int, int>>> cols_st(n);
        for(int i = m - 1; i >= 0; --i) {
            vector<pair<int, int>> st;
            for(int j = n - 1; j >= 0; --j) {
                auto &st2 = cols_st[j];
                mn = INT_MAX;
                if(i == m - 1 && j == n - 1) mn = 0;
                else if(int g = grid[i][j]; g) {
                    auto it = lower_bound(st.begin(), st.end(), j + g, [](const auto &a, const int b) {
                        return a.second > b;
                    });
                    if(it < st.end()) mn = min(mn, it->first);
                    it = lower_bound(st2.begin(), st2.end(), i + g, [](const auto &a, const int b) {
                    // 如果符合要求就一直查找，直到找到第一个不符合要求的情况；即a.second <= d。
                        return a.second > b;
                    });
                    if(it < st2.end()) mn = min(mn, it->first);
                }
                if(mn == INT_MAX) continue;
                ++mn;
                while(!st.empty() && mn <= st.back().first) st.pop_back();
                st.emplace_back(mn, j);
                while(!st2.empty() && mn <= st2.back().first) st2.pop_back();
                st2.emplace_back(mn, i);
            }
        }
        return mn < INT_MAX ? mn : -1;
    }
};
```