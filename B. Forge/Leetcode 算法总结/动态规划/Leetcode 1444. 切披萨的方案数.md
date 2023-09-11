给你一个 `rows x cols` 大小的矩形披萨和一个整数 `k` ，矩形包含两种字符： `'A'` （表示苹果）和 `'.'` （表示空白格子）。你需要切披萨 `k-1` 次，得到 `k` 块披萨并送给别人。

切披萨的每一刀，先要选择是向垂直还是水平方向切，再在矩形的边界上选一个切的位置，将披萨一分为二。如果垂直地切披萨，那么需要把左边的部分送给一个人，如果水平地切，那么需要把上面的部分送给一个人。在切完最后一刀后，需要把剩下来的一块送给最后一个人。

请你返回确保每一块披萨包含 **至少** 一个苹果的切披萨方案数。由于答案可能是个很大的数字，请你返回它对 $10^9 + 7$ 取余的结果。

**示例 1：**

![[Pasted image 20230817212524.png|400|center]]

```
输入：pizza = ["A..","AAA","..."], k = 3
输出：3 
解释：上图展示了三种切披萨的方案。注意每一块披萨都至少包含一个苹果。
```

**示例 2：**

```
输入：pizza = ["A..","AA.","..."], k = 3
输出：1
```

**示例 3：**

```
输入：pizza = ["A..","A..","..."], k = 1
输出：1
```

**提示：**

- `1 <= rows, cols <= 50`
- `rows == pizza.length`
- `cols == pizza[i].length`
- `1 <= k <= 10`
- `pizza` 只包含字符 `'A'` 和 `'.'` 。

$$
f[k][i][j] = \sum_{j < j_{2} < n}f[k - 1][i][j_{2}] + \sum_{i < i_{2} < m} f[k - 1][i_{2}][j]
$$
边界条件：
+ 如果左上角在(i, j)和右下角(m - 1, n - 1)的子矩阵存在苹果，则$f[0][i][j]=1$。
+ 否则 $f[0][i][j]=0$.

答案为$f[k-1][0][0]$.

```c++
class Solution {
public:
    vector<vector<int>> sum;

    void matrixsum(vector<string>& matrix) {
        int m = matrix.size(), n = matrix[0].size();
        sum = vector<vector<int>>(m + 1, vector<int>(n + 1));
        for(int i = 0; i < m; ++i) {
            for(int j = 0; j < n; ++j) {
                sum[i + 1][j + 1] = sum[i + 1][j] + sum[i][j + 1] - sum[i][j] + (matrix[i][j] & 1);
            }
        }
    } 
    int query(int r1, int c1, int r2, int c2) {
        return sum[r2][c2] - sum[r2][c1] - sum[r1][c2] + sum[r1][c1];
    }

    int ways(vector<string>& pizza, int k) {
        const int MOD = 1e9 + 7;
        matrixsum(pizza);
        int m = pizza.size(), n = pizza[0].size();
        // function<int(int, int, int)> dfs = [&](int c, int i, int j) -> int {
        //     if(c == 0) return query(i, j, n, m) ? 1 : 0;
        //     int res = 0;
        //     for(int j2 = j + 1; j2 < n; ++j2) if(query(i, j, m, j2)) {
        //         res = (res + dfs(c - 1, i, j2))% MOD;
        //     }
        //     for(int i2 = i + 1; i2 < m; ++i2) if(query(i, j, i2, n)) {
        //         res = (res + dfs(c - 1, i2, j))% MOD;
        //     }
        //     return res;
        // };
        // return dfs(k - 1, 0, 0); 
        int f[k][m][n];
        for(int c = 0; c < k; ++c) 
            for(int i = 0; i < m; ++i) 
                for(int j = 0; j < n; ++j) {
                    if(c == 0) {
                        f[c][i][j] = query(i, j, m, n) ? 1 : 0;
                        continue;
                    }
                    int res = 0;
                    for(int i2 = i + 1; i2 < m; ++i2) if(query(i, j, i2, n)) {
                        res = (res + f[c - 1][i2][j]) % MOD;
                    }
                    for(int j2 = j + 1; j2 < n; ++j2) if(query(i, j, m, j2)) {
                        res = (res + f[c - 1][i][j2]) % MOD;
                    }
                    f[c][i][j] = res;
                }
        return f[k - 1][0][0];
    }
};
```

```c++
class Solution {
public:

    int ways(vector<string>& matrix, int k) {
        const int MOD = 1e9 + 7;
        int m = matrix.size(), n = matrix[0].size();
        vector<vector<int>> sum(m + 1, vector<int>(n + 1));
        vector<vector<int>> f(m + 1, vector<int>(n + 1));
        for(int i = m - 1; i >= 0; --i) {
            for(int j = n - 1; j >= 0; --j) {
                sum[i][j] = sum[i][j + 1] + sum[i + 1][j] - sum[i + 1][j + 1] + (matrix[i][j] & 1);
                if(sum[i][j]) f[i][j] = 1;
            }
        }
        while(--k) {
            vector<int> col(n);
            for(int i = m - 1; i >= 0; --i) {
                int row = 0;
                for(int j = n - 1; j >= 0; --j) {
                    int tmp = f[i][j];
                    if(sum[i][j] == sum[i + 1][j]) 
                        f[i][j] = f[i + 1][j];
                    else if(sum[i][j] == sum[i][j + 1])
                        f[i][j] = f[i][j + 1];
                    else 
                        f[i][j] = (row + col[j]) % MOD;
                    row = (row + tmp) % MOD;
                    col[j] = (col[j] + tmp) % MOD;
                }
            }
        }
        return f[0][0];
    }
};
```