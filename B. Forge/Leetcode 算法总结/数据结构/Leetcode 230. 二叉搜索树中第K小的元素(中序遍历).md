#### [230. 二叉搜索树中第K小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/)

给定一个二叉搜索树的根节点 `root` ，和一个整数 `k` ，请你设计一个算法查找其中第 `k` 个最小元素（从 1 开始计数）。

**示例 1：**

![[Pasted image 20230605132402.png|200]]
```
输入：root = [3,1,4,null,2], k = 1
输出：1
```

**示例 2：**

![[Pasted image 20230605132459.png|200]]

```
输入：root = [5,3,6,2,4,null,null,1], k = 3
输出：3
```

```c++
//二叉搜索树的中序遍历
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
    int findK(TreeNode* root, int &k) {
        if(root == nullptr) {
            return - 1;
        }
        int Knum = findK(root->left, k);
        if(Knum != -1) return Knum;
        if(--k == 0) return root->val;
        return findK(root->right, k);
    }

    int kthSmallest(TreeNode* root, int k) {
        return findK(root, k);
    }
};
```