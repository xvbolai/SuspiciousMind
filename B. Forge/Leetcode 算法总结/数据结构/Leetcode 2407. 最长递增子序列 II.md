给你一个整数数组 `nums` 和一个整数 `k` 。

找到 `nums` 中满足以下要求的最长子序列：

- 子序列 **严格递增**
- 子序列中相邻元素的差值 **不超过** `k` 。

请你返回满足上述要求的 **最长子序列** 的长度。

**子序列** 是从一个数组中删除部分元素后，剩余元素不改变顺序得到的数组。
**示例 1：**
```txt
输入：nums = [4,2,1,4,3,4,5,8,15], k = 3
输出：5
解释：
满足要求的最长子序列是 [1,3,4,5,8]。
子序列长度为 5 ，所以我们返回 5 。
注意子序列 [1,3,4,5,8,15] 不满足要求，因为 15 - 8 = 7 大于 3 。
```

对于本题，由于有一个差值不超过 $k$ 的约束，用线段树更好处理。
具体来说，定义$f[i][j]$表示$nums$的前$i$个元素中，以元素$j$(注意不是$nu ms[j]$)结尾的满足题目两个条件的子序列的最长长度。
当$j\neq nums[i]$时，$f[i][j]=f[i-1][j]$。
当$j = nums[i]$时，我们可以从$f[i - 1][j^{'}]$转移过来，这里$j - k \leq j^{'}\leq j$，取最大值，得
$$
f[i][j]=1+max_{j^{'}=j-k}^{j-1} f[i - 1][j^{'}]
$$
上式有一个**区间求最大值** 的过程，这非常适合**线段树**计算，且由于$f[i]$只会从$f[i-1]$转移过来，我们可以把$f$的第一个维度优化掉。这样我们就可以用线段树来表示整个$f$数组，在上边的查询和更新。
最后的答案为$max(f[n - 1])$，对应到线段树则是根节点。

```c++
class Solution {
private:
    vector<int> max;
    
    void modify(int o, int l, int r, int i, int val) {
        if(l == r) {
            max[o] = val;
            return ;
        }
        int m = (l + r) / 2;
        if(i <= m) modify(o << 1, l, m, i, val);
        else modify(o << 1 | 1, m + 1, r, i, val);
        max[o] = std::max(max[o << 1], max[o << 1 | 1]);
    }

    int query(int o, int l, int r, int L, int  R) {
        if(L <= l && r <= R) return max[o];
        int res = 0;
        int m = (l + r) / 2;
        if(L <= m) res = query(o << 1, l, m, L, R);
        if(R > m) res = std::max(res, query(o << 1 | 1, m + 1, r, L, R));
        return res;
    }

public:
    int lengthOfLIS(vector<int>& nums, int k) {
        int u = *max_element(nums.begin(), nums.end());
        max.resize(u * 4);
        for(auto x : nums) {
            if(x == 1) modify(1, 1, u, 1, 1);
            else {
                int res = 1 + query(1, 1, u, std::max(x - k, 1), x - 1);
                modify(1, 1, u, x, res);
            }
        }
        return max[1];
    }
};
```