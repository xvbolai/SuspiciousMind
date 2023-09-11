#### [1483. 树节点的第 K 个祖先](https://leetcode.cn/problems/kth-ancestor-of-a-tree-node/)

给你一棵树，树上有 `n` 个节点，按从 `0` 到 `n-1` 编号。树以父节点数组的形式给出，其中 `parent[i]` 是节点 `i` 的父节点。树的根节点是编号为 `0` 的节点。

树节点的第 _`k`_ 个祖先节点是从该节点到根节点路径上的第 `k` 个节点。

实现 `TreeAncestor` 类：
- `TreeAncestor（int n， int[] parent）` 对树和父数组中的节点数初始化对象。
- `getKthAncestor``(int node, int k)` 返回节点 `node` 的第 `k` 个祖先节点。如果不存在这样的祖先节点，返回 `-1` 。
**示例 1：**
![[Pasted image 20230609121657.png|center|200]]


```
输入：
["TreeAncestor","getKthAncestor","getKthAncestor","getKthAncestor"]
[[7,[-1,0,0,1,1,2,2]],[3,1],[5,2],[6,3]]

输出：
[null,1,0,-1]

解释：
TreeAncestor treeAncestor = new TreeAncestor(7, [-1, 0, 0, 1, 1, 2, 2]);

treeAncestor.getKthAncestor(3, 1);  // 返回 1 ，它是 3 的父节点
treeAncestor.getKthAncestor(5, 2);  // 返回 0 ，它是 5 的祖父节点
treeAncestor.getKthAncestor(6, 3);  // 返回 -1 因为不存在满足要求的祖先节点
```
Binary Lifting 的本质其实是 dp。dp\[node\]\[j\] 存储的是 node 节点距离为 2^j 的祖先是谁。

根据定义，dp\[node\]\[0\] 就是 parent\[node\]，即 node 的距离为 1 的祖先是 parent\[node\]。
状态转移是： `dp[node][j] = dp[dp[node][j - 1]][j - 1]`。
意思是：要想找到 node 的距离 2^j 的祖先，先找到 node 的距离 2^(j - 1) 的祖先，然后，再找这个祖先的距离 2^(j - 1) 的祖先。两步得到 node 的距离为 2^j 的祖先。

所以，我们要找到每一个 node 的距离为 1, 2, 4, 8, 16, 32, ... 的祖先，直到达到树的最大的高度。树的最大的高度是 logn 级别的。

这样做，状态总数是 O(nlogn)，可以使用 O(nlogn) 的时间做预处理。

之后，根据预处理的结果，可以在 O(logn) 的时间里完成每次查询：对于每一个查询 k，把 k 拆解成二进制表示，然后根据二进制表示中 1 的位置，累计向上查询。


```c++
class TreeAncestor {
    vector<vector<int>> dp;
public:
    TreeAncestor(int n, vector<int>& parent): dp(n) {
        // dp.resize(n);
        for(int i = 0; i < n; ++i) dp[i].push_back(parent[i]);
        bool allneg = true;
        for(int j = 1; ; ++j) {
            allneg = true;
            for(int i = 0; i < n; ++i) {
                int t = dp[i][j - 1] != -1 ? dp[dp[i][j - 1]][j - 1] : -1;
                dp[i].push_back(t);
                if(t != -1) allneg = false;
            }
            if(allneg) break;
        }
    }
    
    int getKthAncestor(int node, int k) {
        // if(k == 0 || node == -1) return node;
        // int pos = ffs(k) - 1;
        // return pos < dp[node].size() ? getKthAncestor(dp[node][pos], k - (1 << pos)) : -1;
        int res = node, pos = 0;
        while(k && res != -1) {
            if(pos >= dp[res].size()) return -1;
            if(k & 1) res = dp[res][pos];
            k >>= 1;
            ++pos;
        }
        return res;
    }
};

/**
 * Your TreeAncestor object will be instantiated and called as such:
 * TreeAncestor* obj = new TreeAncestor(n, parent);
 * int param_1 = obj->getKthAncestor(node,k);
 */
```


### LCA(最近公共祖先)
![[Pasted image 20230609113946.png]]
![[Pasted image 20230609114002.png]]
```c++
from collections import defaultdict

class LCA:
    def __init__(self, n, edges):
        self.n = n
        self.edges = edges
        self.depth = [0] * n
        self.parent = [[-1] * int(log2(n)) for i in range(n)]
        self.build()

    def build(self):
        graph = defaultdict(list)
        for u, v in self.edges:
            graph[u].append(v)
            graph[v].append(u)
        self.dfs(graph, 0, -1, 0)
        for k in range(1, int(log2(self.n))):
            for i in range(self.n):
                if self.parent[i][k-1] != -1:
                    self.parent[i][k] = self.parent[self.parent[i][k-1]][k-1]

    def dfs(self, graph, u, p, d):
        self.parent[u][0] = p
        self.depth[u] = d
        for v in graph[u]:
            if v != p:
                self.dfs(graph, v, u, d+1)

    def query(self, u, v):
        if self.depth[u] < self.depth[v]:
            u, v = v, u
        for k in range(int(log2(self.n))-1, -1, -1):
            if self.depth[u] - 2**k >= self.depth[v]:
                u = self.parent[u][k]
        if u == v:
            return u
        for k in range(int(log2(self.n))-1, -1, -1):
            if self.parent[u][k] != self.parent[v][k]:
                u = self.parent[u][k]
                v = self.parent[v][k]
        return self.parent[u][0]

```