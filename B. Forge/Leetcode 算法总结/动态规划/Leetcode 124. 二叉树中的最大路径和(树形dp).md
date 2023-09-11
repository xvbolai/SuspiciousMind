#### [124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/)

二叉树中的 **路径** 被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中 **至多出现一次** 。该路径 **至少包含一个** 节点，且不一定经过根节点。

**路径和** 是路径中各节点值的总和。

给你一个二叉树的根节点 `root` ，返回其 **最大路径和** 。

**示例 1：**

![[exx1.jpg|100]]

```
输入：root = [1,2,3]
输出：6
解释：最优路径是 2 -> 1 -> 3 ，路径和为 2 + 1 + 3 = 6
```

**示例 2：**

![[exx2.jpg|150]]

```
输入：root = [-10,9,20,null,null,15,7]
输出：42
解释：最优路径是 15 -> 20 -> 7 ，路径和为 15 + 20 + 7 = 42
```

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
    int maxPathSum(TreeNode* root) {
        int ans = INT_MIN;
        function<int(TreeNode*)> dfs = [&](TreeNode* node) -> int {
            if(node == nullptr) return 0;
            int l_val = dfs(node->left);
            int r_val = dfs(node->right);
            ans = max(ans, l_val + r_val + node->val);
            return max(max(l_val, r_val) + node->val, 0); // 选择还是不选这个子树
        };
        dfs(root);
        return ans;
    }
};
```



##### 相关题目

 [543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/solution/shi-pin-che-di-zhang-wo-zhi-jing-dpcong-taqma/)

 [124. 二叉树中的最大路径和](https://leetcode.cn/problems/binary-tree-maximum-path-sum/solution/shi-pin-che-di-zhang-wo-zhi-jing-dpcong-n9s91/)

[2246. 相邻字符不同的最长路径](https://leetcode.cn/problems/longest-path-with-different-adjacent-characters/solution/by-endlesscheng-92fw/) 

课后作业

 [687. 最长同值路径](https://leetcode.cn/problems/longest-univalue-path/) 

[1617. 统计子树中城市之间最大距离](https://leetcode.cn/problems/count-subtrees-with-max-distance-between-cities/) 

[2538. 最大价值和与最小价值和的差值](https://leetcode.cn/problems/difference-between-maximum-and-minimum-price-sum/)

