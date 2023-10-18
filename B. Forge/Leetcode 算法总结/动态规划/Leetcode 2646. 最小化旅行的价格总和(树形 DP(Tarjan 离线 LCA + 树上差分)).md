现有一棵无向、无根的树，树中有 `n` 个节点，按从 `0` 到 `n - 1` 编号。给你一个整数 `n` 和一个长度为 `n - 1` 的二维整数数组 `edges` ，其中 `edges[i] = [ai, bi]` 表示树中节点 `ai` 和 `bi` 之间存在一条边。

每个节点都关联一个价格。给你一个整数数组 `price` ，其中 `price[i]` 是第 `i` 个节点的价格。

给定路径的 **价格总和** 是该路径上所有节点的价格之和。

另给你一个二维整数数组 `trips` ，其中 `trips[i] = [starti, endi]` 表示您从节点 `starti` 开始第 `i` 次旅行，并通过任何你喜欢的路径前往节点 `endi` 。

在执行第一次旅行之前，你可以选择一些 **非相邻节点** 并将价格减半。

返回执行所有旅行的最小价格总和。

**示例 1：**

![[Pasted image 20231013163635.png]]

```txt
**输入：**n = 4, edges = [[0,1],[1,2],[1,3]], price = [2,2,10,6], trips = [[0,3],[2,1],[2,3]]
**输出：**23
**解释：**
上图表示将节点 2 视为根之后的树结构。第一个图表示初始树，第二个图表示选择节点 0 、2 和 3 并使其价格减半后的树。
第 1 次旅行，选择路径 [0,1,3] 。路径的价格总和为 1 + 2 + 3 = 6 。
第 2 次旅行，选择路径 [2,1] 。路径的价格总和为 2 + 5 = 7 。
第 3 次旅行，选择路径 [2,1,3] 。路径的价格总和为 5 + 2 + 3 = 10 。
所有旅行的价格总和为 6 + 7 + 10 = 23 。可以证明，23 是可以实现的最小答案。
```

**示例 2：**

![[Pasted image 20231013163707.png]]

```txt
**输入：**n = 2, edges = [[0,1]], price = [2,2], trips = [[0,0]]
**输出：**1
**解释：**
上图表示将节点 0 视为根之后的树结构。第一个图表示初始树，第二个图表示选择节点 0 并使其价格减半后的树。 
第 1 次旅行，选择路径 [0] 。路径的价格总和为 1 。 
所有旅行的价格总和为 1 。可以证明，1 是可以实现的最小答案。
```

**提示：**

- `1 <= n <= 50`
- `edges.length == n - 1`
- `0 <= ai, bi <= n - 1`
- `edges` 表示一棵有效的树
- `price.length == n`
- `price[i]` 是一个偶数
- `1 <= price[i] <= 1000`
- `1 <= trips.length <= 100`
- `0 <= starti, endi <= n - 1`

#### 提示 1

The final answer is the $price[i] * freq[i]$, where $freq[i]$ is the number of times node i was visited during the trip, and $price[i]$ is the final price.

#### 提示 2

To find $freq[i]$ we will use dfs or bfs for each trip and update every node on the path start and end.

#### 提示 3

Finally, to find the final $price[i]$ we will use dynamic programming on the tree. Let dp(v, 0/1) denote the minimum total price with the node v’s price being halved or not.

```c++
class Solution {
public:
    int minimumTotalPrice(int n, vector<vector<int>>& edges, vector<int>& price, vector<vector<int>>& trips) {
          vector<vector<int>> g(n);
          for(const auto& x : edges) {
              g[x[0]].emplace_back(x[1]);
              g[x[1]].emplace_back(x[0]);
          }
          vector<int> freq(n);
          
          for(const auto& x : trips) {
              int end = x[1];
              function<bool(int, int)> dfs = [&](int u, int fa) -> bool {
                if(u == end) {
                    ++freq[u];
                    return true;
                }
                for(const auto & v : g[u]) if(v != fa && dfs(v, u)) {
                    ++freq[u];
                    return true;
                }
                return false;
              };
              dfs(x[0], -1);
          }
          function<pair<int, int>(int, int)> dfs = [&](int u, int fa) -> pair<int, int> {
              int nhalf = price[u] * freq[u];
              int half = nhalf / 2;
              for(const auto& v : g[u]) if(v != fa) {
                  auto [nh, h] = dfs(v, u);
                  nhalf += min(nh, h);
                  half += nh;
              }
              return {nhalf, half};
          };
          auto [nh, h] = dfs(0, -1);
          return min(nh, h);
    }
};
```

###  Tarjan 离线 LCA + 树上差分 

#### Tarjan算法

![[Leetcode 1192. 查找集群内的关键连接(图的割点和桥问题之Tarjan算法)]]

#### 代码

```c++
class Solution {
public:
    int minimumTotalPrice(int n, vector<vector<int>>& edges, vector<int>& price, vector<vector<int>>& trips) {
        vector<vector<int>> g(n);
        for(const auto& e : edges) {
            g[e[0]].emplace_back(e[1]);
            g[e[1]].emplace_back(e[0]);
        }
        vector<vector<int>> qs(n);
        for(const auto& x : trips) {
            qs[x[0]].push_back(x[1]);
            if(x[0] != x[1]) qs[x[1]].emplace_back(x[0]);
        }
        int pa[n];
        iota(pa, pa + n, 0);
        function<int(int)> find = [&](int x) -> int { return x == pa[x] ? x : pa[x] = find(pa[x]); };
        int  diff[n], father[n], color[n];
        memset(diff, 0, sizeof(diff));
        memset(color, 0, sizeof(color));
        function<void(int, int)> tarjan = [&](int x, int fa) {
            father[x] = fa;
            color[x] = 1; // 递归中
            for(int y : g[x]) if(color[y] == 0) { // 未递归到
                tarjan(y, x);
                pa[y] = x; // 相当于把y的子树节点全部merge到x
            }
            color[x] = 2; // 准备回溯
            for(int y : qs[x]) {
                if(color[y] == 2) {
                    ++diff[x];
                    ++diff[y];
                    int lca = find(y);
                    --diff[lca];
                    int f = father[lca];
                    if(f >= 0) --diff[f];
                }
            }
        };
        tarjan(0, -1);
        function<tuple<int, int, int>(int, int)> dfs = [&](int x, int fa) -> tuple<int, int, int> {
            int nhalf = 0, half = 0, cnt = diff[x];
            for(int y : g[x]) if(y != fa) {
                auto [nh, h, c] = dfs(y, x);
                nhalf += min(nh, h);
                half += nh;
                cnt += c;
            }
            nhalf += price[x] * cnt;
            half += price[x] * cnt / 2;
            return {nhalf, half, cnt};
        };
        auto [nh, h, _] = dfs(0, -1);
        return min(nh, h);
    }
};
```

