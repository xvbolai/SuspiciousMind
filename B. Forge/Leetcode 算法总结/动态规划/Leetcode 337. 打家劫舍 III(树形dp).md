#### [337. 打家劫舍 III](https://leetcode.cn/problems/house-robber-iii/)

小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为 `root` 。

除了 `root` 之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果 **两个直接相连的房子在同一天晚上被打劫** ，房屋将自动报警。

给定二叉树的 `root` 。返回 ***在不触动警报的情况下** ，小偷能够盗取的最高金额* 。

**示例 1:**

![[rob1-tree.jpg|200]]

```
输入: root = [3,2,3,null,3,null,1]
输出: 7 
解释: 小偷一晚能够盗取的最高金额 3 + 3 + 1 = 7
```

**示例 2:**

![[rob2-tree.jpg|200]]

```
输入: root = [3,4,5,1,3,null,1]
输出: 9
解释: 小偷一晚能够盗取的最高金额 4 + 5 = 9
```

#### 题解



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
    pair<int, int> dfs(TreeNode* root) {
        if(root == nullptr) return {0, 0};
        auto [l, ln] = dfs(root->left);
        auto [r, rn] = dfs(root->right);
        int ro = ln + rn + root->val;
        int ron = max(l, ln) + max(r, rn);
        return {ro, ron};
    }
    int rob(TreeNode* root) {
        auto [u, v] = dfs(root);
        return max(u, v);
    }
};
```

#### 课后作业

+ 没有上司的舞会 https://www.luogu.com.cn/problem/P1352

- [1377. T 秒后青蛙的位置](https://leetcode.cn/problems/frog-position-after-t-seconds/solution/dfs-ji-yi-ci-you-qu-de-hack-by-endlessch-jtsr/)，[题解](https://leetcode.cn/problems/frog-position-after-t-seconds/solution/dfs-ji-yi-ci-you-qu-de-hack-by-endlessch-jtsr/)
- [2646. 最小化旅行的价格总和](https://leetcode.cn/problems/minimize-the-total-price-of-the-trips/)，[题解](https://leetcode.cn/problems/minimize-the-total-price-of-the-trips/solution/lei-si-da-jia-jie-she-iii-pythonjavacgo-4k3wq/)