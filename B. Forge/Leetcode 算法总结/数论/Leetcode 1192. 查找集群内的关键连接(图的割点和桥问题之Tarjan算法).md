力扣数据中心有 `n` 台服务器，分别按从 `0` 到 `n-1` 的方式进行了编号。它们之间以 **服务器到服务器** 的形式相互连接组成了一个内部集群，连接是无向的。用  `connections` 表示集群网络，`connections[i] = [a, b]` 表示服务器 `a` 和 `b` 之间形成连接。任何服务器都可以直接或者间接地通过网络到达任何其他服务器。

**关键连接** 是在该集群中的重要连接，假如我们将它移除，便会导致某些服务器无法访问其他服务器。

请你以任意顺序返回该集群内的所有 **关键连接** 。

**示例 1：**

![[Pasted image 20231016095422.png|center|300]]

```txt
**输入：**n = 4, connections = [[0,1],[1,2],[2,0],[1,3]]
**输出：**[[1,3]]
**解释：**[[3,1]] 也是正确的。
```
**示例 2:**
```txt
**输入：**n = 2, connections = [[0,1]]
**输出：**[[0,1]]
```
**提示：**
- `2 <= n <= 105`
- `n - 1 <= connections.length <= 105`
- `0 <= ai, bi <= n - 1`
- `ai != bi`
- 不存在重复的连接

#### 思路

按深度优先顺序遍历所有结点，给每个结点编号为now。

1. 对于当前结点$u$，其子树的能到达的最小编号，结点$u$肯定也能到达。即$low[u] = min(low[u], low[v])$

2. 对于当前结点$u$，如果其能直接访问到之前遍历过的结点，这个遍历过的结点就有可能是当前结点$u$能到达的最小编号。即$low[u] = min(low[u], dfn[v])$

3. 对于当前结点$u$到结点$v$这条边，如果从结点$v$继续遍历后，结点$v$不能到达比结点$u$编号更小的结点，则这$u->v$这条边就是桥了，也就是说，除了从$u$走到$v$这条边，从节点$v$开始走，永远走不到$u$和$u$之前走过的点了。即$if(low[v] > dfn[u]) ans.push_back({u,v})$

#### 代码

```c++
class Solution {
public:
    vector<vector<int>> criticalConnections(int n, vector<vector<int>>& connections) {
        vector<vector<int>> g(n), ans;
        vector<int> low(n), dfn(n);
        for(const auto& e : connections) {
            g[e[0]].emplace_back(e[1]);
            g[e[1]].emplace_back(e[0]);
        }
        int now = 0;
        function<void(int, int)> tarjan = [&](int x, int fa) -> void {
            dfn[x] = low[x] = ++now;
            for(int y : g[x]) if(y != fa) {
                if(!dfn[y]) {
                    tarjan(y, x);
                    low[x] = min(low[x], low[y]);
                    if(low[y] > dfn[x]) ans.push_back({x, y});
                } else low[x] = min(dfn[y], low[x]);
            }
        };
        tarjan(0, -1);
        return ans;
    }
};
```