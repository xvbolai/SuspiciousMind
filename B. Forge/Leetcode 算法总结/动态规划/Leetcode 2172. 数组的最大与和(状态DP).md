给你一个长度为 `n` 的整数数组 `nums` 和一个整数 `numSlots` ，满足`2 * numSlots>= n` 。总共有 `numSlots` 个篮子，编号为 `1` 到 `numSlots` 。

你需要把所有 `n` 个整数分到这些篮子中，且每个篮子 **至多** 有 2 个整数。一种分配方案的 **与和** 定义为每个数与它所在篮子编号的 **按位与运算** 结果之和。

+ 比方说，将数字 `[1, 3]` 放入篮子 **_`1`_** 中，`[4, 6]` 放入篮子 **_`2`_** 中，这个方案的与和为 `(1 AND 1_) + (3 AND 1) + (4 AND 2) + (6 AND 2) = 1 + 1 + 0 + 2 = 4` 。

请你返回将 `nums` 中所有数放入 `numSlots` 个篮子中的最大与和。

**示例 1：**

```
输入：nums = [1,2,3,4,5,6], numSlots = 3
输出：9
解释：一个可行的方案是 [1, 4] 放入篮子 1 中，[2, 6] 放入篮子 2 中，[3, 5] 放入篮子 3 中。
最大与和为 (1 AND 1) + (4 AND 1) + (2 AND 2) + (6 AND 2) + (3 AND 3) + (5 AND 3) = 1 + 0 + 2 + 2 + 3 + 1 = 9 。
```


**转换一下**：视作有 `2⋅numSlots` 个篮子，每个篮子**至多**可以放 11 个整数。


```c++
class Solution {
public:
    int maximumANDSum(vector<int>& nums, int numSlots) {
        int ans = 0;
        vector<int> dp(1 << (2 * numSlots));
        for(int i = 0; i < dp.size(); ++i) {
            int c = __builtin_popcount(i);
            if(c >= nums.size()) continue;
            for(int j = 0; j < numSlots * 2; ++j) {
                if((i & (1 << j)) == 0) {
                    int s = i | (1 << j);
                    dp[s] =  max(dp[s], dp[i] + ((j / 2 + 1) & nums[c]));
                }
            } 
        }
        return *max_element(dp.begin(), dp.end());
    }
};
```