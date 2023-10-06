按照国际象棋的规则，皇后可以攻击与之处在同一行或同一列或同一斜线上的棋子。

**n 皇后问题** 研究的是如何将 `n` 个皇后放置在 `n×n` 的棋盘上，并且使皇后彼此之间不能相互攻击。

给你一个整数 `n` ，返回所有不同的 **n 皇后问题** 的解决方案。

每一种解法包含一个不同的 **n 皇后问题** 的棋子放置方案，该方案中 `'Q'` 和 `'.'` 分别代表了皇后和空位。

**示例 1：**

![[Pasted image 20230915101822.png]]
```txt
**输入：**n = 4
**输出：**[[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]
**解释：**如上图所示，4 皇后问题存在两个不同的解法。
```

```c++
class Solution {
public:
    vector<vector<string>> solveNQueens(int n) {
        vector<vector<string>> res;
        vector<int> col(n), path(n), dia1(2 * n - 1), dia2(2 * n - 1);
        function<void(int)> dfs = [&](int r) {
            if(r == n) {
                vector<string> t;
                for(auto &cnt : col) {
                    t.push_back(string(cnt, '.') + 'Q' + string(n - 1 - cnt, '.'));
                }
                res.push_back(t);
                return;
            }
            for(int i = 0; i < n; ++i) {
                int rc = r - i + n - 1;
                if(!path[i] && !dia1[r + i] && !dia2[rc]) {
                    path[i] = dia2[rc] = dia1[r + i] = true;
                    col[r] = i;
                    dfs(r+ 1);
                    path[i] = dia2[rc] = dia1[r + i] = false;
                }
            }
        };
        dfs(0);
        return res;
    }
};
```
