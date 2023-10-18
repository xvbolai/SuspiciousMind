Alice 有一棵 `n` 个节点的树，节点编号为 `0` 到 `n - 1` 。树用一个长度为 `n - 1` 的二维整数数组 `edges` 表示，其中 `edges[i] = [ai, bi]` ，表示树中节点 `ai` 和 `bi` 之间有一条边。

Alice 想要 Bob 找到这棵树的根。她允许 Bob 对这棵树进行若干次 **猜测** 。每一次猜测，Bob 做如下事情：

- 选择两个 **不相等** 的整数 `u` 和 `v` ，且树中必须存在边 `[u, v]` 。
- Bob 猜测树中 `u` 是 `v` 的 **父节点** 。

Bob 的猜测用二维整数数组 `guesses` 表示，其中 `guesses[j] = [uj, vj]` 表示 Bob 猜 `uj` 是 `vj` 的父节点。

Alice 非常懒，她不想逐个回答 Bob 的猜测，只告诉 Bob 这些猜测里面 **至少** 有 `k` 个猜测的结果为 `true` 。

给你二维整数数组 `edges` ，Bob 的所有猜测和整数 `k` ，请你返回可能成为树根的 **节点数目** 。如果没有这样的树，则返回 `0`。

**示例 1：**

![[Pasted image 20231016215242.png]]

```txt
**输入：**edges = [[0,1],[1,2],[1,3],[4,2]], guesses = [[1,3],[0,1],[1,0],[2,4]], k = 3
**输出：**3
**解释：**
根为节点 0 ，正确的猜测为 [1,3], [0,1], [2,4]
根为节点 1 ，正确的猜测为 [1,3], [1,0], [2,4]
根为节点 2 ，正确的猜测为 [1,3], [1,0], [2,4]
根为节点 3 ，正确的猜测为 [1,0], [2,4]
根为节点 4 ，正确的猜测为 [1,3], [1,0]
节点 0 ，1 或 2 为根时，可以得到 3 个正确的猜测。
```

**示例 2：**
![[Pasted image 20231016215312.png]]

```txt
**输入：**edges = [[0,1],[1,2],[2,3],[3,4]], guesses = [[1,0],[3,4],[2,1],[3,2]], k = 1
**输出：**5
**解释：**
根为节点 0 ，正确的猜测为 [3,4]
根为节点 1 ，正确的猜测为 [1,0], [3,4]
根为节点 2 ，正确的猜测为 [1,0], [2,1], [3,4]
根为节点 3 ，正确的猜测为 [1,0], [2,1], [3,2], [3,4]
根为节点 4 ，正确的猜测为 [1,0], [2,1], [3,2]
任何节点为根，都至少有 1 个正确的猜测。
```

**提示：**

- `edges.length == n - 1`
- `2 <= n <= 105`
- `1 <= guesses.length <= 105`
- `0 <= ai, bi, uj, vj <= n - 1`
- `ai != bi`

```c++
class Solution {
public:
    int rootCount(vector<vector<int>>& edges, vector<vector<int>>& guesses, int k) {
        vector<vector<int>> g(edges.size() + 1);
        for(const auto& e : edges) {
            g[e[0]].emplace_back(e[1]);
            g[e[1]].emplace_back(e[0]);
        }
        unordered_set<long> s;
        for(auto &e : guesses) {
            s.insert((long)e[0] << 32 | e[1]);
        }
        int ans = 0, cnt0 = 0;
        function<void(int, int)> dfs = [&](int x, int fa) {
            for(int y : g[x]) if(y != fa) {
                cnt0 += s.count((long)x << 32 | y);
                dfs(y, x);
            }
        };
        dfs(0, -1);
        function<void(int, int, int)> reroot = [&](int x, int fa, int cnt) {
            ans += cnt >= k;
            for(int y : g[x]) if(y != fa) {
                reroot(y, x, cnt - s.count((long)x << 32 | y) + s.count((long) y << 32 | x));
            }
        };
        reroot(0, -1, cnt0);
        return ans;
    }
};
```

### [834. 树中距离之和](https://leetcode.cn/problems/sum-of-distances-in-tree/)

给定一个无向、连通的树。树中有 `n` 个标记为 `0...n-1` 的节点以及 `n-1` 条边 。

给定整数 `n` 和数组 `edges` ， `edges[i] = [ai, bi]`表示树中的节点 `ai` 和 `bi` 之间有一条边。

返回长度为 `n` 的数组 `answer` ，其中 `answer[i]` 是树中第 `i` 个节点与所有其他节点之间的距离之和。

**示例 1:**

![[Pasted image 20231016215504.png]]

```txt
**输入:** n = 6, edges = [[0,1],[0,2],[2,3],[2,4],[2,5]]
**输出:** [8,12,6,10,10,10]
**解释:** 树如图所示。
我们可以计算出 dist(0,1) + dist(0,2) + dist(0,3) + dist(0,4) + dist(0,5) 
也就是 1 + 1 + 2 + 2 + 2 = 8。 因此，answer[0] = 8，以此类推。
```

**示例 2:**

![[Pasted image 20231016215530.png]]

```txt
**输入:** n = 1, edges = []
**输出:** [0]
```

**提示:**

- `1 <= n <= 3 * 104`
- `edges.length == n - 1`
- `edges[i].length == 2`
- `0 <= ai, bi < n`
- `ai != bi`
- 给定的输入保证为有效的树

![[Pasted image 20231016215637.png]]


```c++
class Solution {
public:
    vector<int> sumOfDistancesInTree(int n, vector<vector<int>>& edges) {
        vector<int> g[n];
        for(auto &e : edges) {
            int x = e[0], y = e[1];
            g[x].push_back(y);
            g[y].push_back(x);
        }
        vector<int> ans(n);
        vector<int> size(n, 1);
        function<void(int, int, int)> dfs = [&](int x, int f, int d) {
            ans[0] += d;
            for(int y : g[x]) if(y != f) {
                dfs(y, x, d + 1);
                size[x] += size[y];
            }
        };
        dfs(0, -1, 0);
        function<void(int, int)> reroot = [&](int x, int f) {
            for(int y : g[x]) if(y != f) {
                ans[y] = ans[x] + n - 2 * size[y];
                reroot(y, x);
            }
        };
        reroot(0, -1);
        return ans;
    }
};
```

练习：换根 DP
310. 最小高度树（做法不止一种）
2581. 统计可能的树根数目
Codeforces 771C. Bear and Tree Jumps（本题进阶版）

### [979. 在二叉树中分配硬币](https://leetcode.cn/problems/distribute-coins-in-binary-tree/)

给你一个有 `n` 个结点的二叉树的根结点 `root` ，其中树中每个结点 `node` 都对应有 `node.val` 枚硬币。整棵树上一共有 `n` 枚硬币。

在一次移动中，我们可以选择两个相邻的结点，然后将一枚硬币从其中一个结点移动到另一个结点。移动可以是从父结点到子结点，或者从子结点移动到父结点。

返回使每个结点上 **只有** 一枚硬币所需的 **最少** 移动次数。

**示例 1：**
![[Pasted image 20231016215838.png]]
```txt
**输入：**root = [3,0,0]
**输出：**2
**解释：**一枚硬币从根结点移动到左子结点，一枚硬币从根结点移动到右子结点。
```
**示例 2：**
![[Pasted image 20231016215912.png]]
```txt
**输入：**root = [0,3,0]
**输出：**3
**解释：**将两枚硬币从根结点的左子结点移动到根结点（两次移动）。然后，将一枚硬币从根结点移动到右子结点。
```
**提示：**

- 树中节点的数目为 `n`
- `1 <= n <= 100`
- `0 <= Node.val <= n`
- 所有 `Node.val` 的值之和是 `n`

![[Pasted image 20231016215948.png]]

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */

class Solution {
public:
    int distributeCoins(TreeNode* root) {
        int ans = 0;
        function<int(TreeNode*)> dfs = [&](TreeNode* node) -> int {
            if(node == nullptr) return 0;
            int d = dfs(node->left) + dfs(node->right) + node->val - 1;
            ans += abs(d);
            return d;
        };
        dfs(root);
        return ans;
    }
};
```