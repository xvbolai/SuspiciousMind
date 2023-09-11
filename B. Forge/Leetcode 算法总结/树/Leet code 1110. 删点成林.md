## 数据结构之树
#数据结构 #后序遍历 #中等
给出二叉树的根节点 `root`，树上每个节点都有一个不同的值。
如果节点值在 `to_delete` 中出现，我们就把该节点从树上删去，最后得到一个森林（一些不相交的树构成的集合）。
返回森林中的每棵树。你可以按任意顺序组织答案。
**示例 1：**
![[Pasted image 20230530145116.png|300]]
```
输入：root = [1,2,3,4,5,6,7], to_delete = [3,5]
输出：[[1,2,null,4],[6],[7]]
```
**示例 2：**
```
输入：root = [1,2,4,null,3], to_delete = [3]
输出：[[1,2,4]]
```

**思路：**

1.  一个节点会不会变成树取决于两点。
	+ 是否在删除点行列。
	+ 是否父节点被删除。


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
    vector<TreeNode*> delNodes(TreeNode* root, vector<int>& to_delete) {
        vector<TreeNode*> ans;
        unordered_set<int> s(to_delete.begin(), to_delete.end());
        function<TreeNode*(TreeNode*, bool)> dfs  = [&] (TreeNode* node, bool is_root) -> TreeNode* {
            if(node == nullptr) return nullptr;
            bool dt = s.count(node->val) == 1 ? true : false;
            node->left = dfs(node->left, dt);
            node->right = dfs(node->right, dt);
            if(is_root && !dt) ans.push_back(node);
            return dt ? nullptr : node;
        };
        dfs(root, true);
        return ans;
    }
};
```
