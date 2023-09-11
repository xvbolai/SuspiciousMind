给你二叉树的根节点 `root` 和一个整数 `limit` ，请你同时删除树中所有`不足节点` ，并返回最终二叉树的根节点。

假如通过节点 `node` 的每种可能的 “根-叶” 路径上值的总和全都小于给定的 `limit`，则该节点被称之为 **`不足节点`** ，需要被删除。

**`叶子节点`**，就是没有子节点的节点。

**示例 1：**

![[insufficient-11.png|400]]

>**输入：root = [1,2,3,4,-99,-99,7,8,9,-99,-99,12,13,-99,14], limit = 1
>输出：**[1,2,3,4,null,null,7,8,9,null,14]

**示例 2：**

![[insufficient-3.png|400]]

>**输入：**root = [5,4,8,11,null,17,4,7,1,null,null,5,3], limit = 22
>**输出：**[5,4,8,11,null,17,4,7,null,null,null,5]

**题解**
本题为从下往上遍历树，删去不符合**limit**的分支。

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
    TreeNode* sufficientSubset(TreeNode* root, int limit) {
        limit -= root->val;
        if(root->left == root->right) {
            return limit > 0 ? nullptr : root;
        }
        root->left =  root->left == nullptr ? nullptr : sufficientSubset(root->left, limit);
        root->right = root->right == nullptr ? nullptr : sufficientSubset(root->right, limit);
        return root->left || root->right ? root : nullptr;
    }
};
```

