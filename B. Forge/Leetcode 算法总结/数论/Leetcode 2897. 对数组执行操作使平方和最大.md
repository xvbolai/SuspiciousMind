#hard 
给你一个下标从 **0** 开始的整数数组 `nums` 和一个 **正** 整数 `k` 。

你可以对数组执行以下操作 **任意次** ：

- 选择两个互不相同的下标 `i` 和 `j` ，**同时** 将 `nums[i]` 更新为 `(nums[i] AND nums[j])` 且将 `nums[j]` 更新为 `(nums[i] OR nums[j])` ，`OR` 表示按位 **或** 运算，`AND` 表示按位 **与** 运算。

你需要从最终的数组里选择 `k` 个元素，并计算它们的 **平方** 之和。

请你返回你可以得到的 **最大** 平方和。

由于答案可能会很大，将答案对 `109 + 7` **取余** 后返回。

**示例 1：**

```txt
**输入：**nums = [2,6,5,8], k = 2
**输出：**261
**解释：**我们可以对数组执行以下操作：
- 选择 i = 0 和 j = 3 ，同时将 nums[0] 变为 (2 AND 8) = 0 且 nums[3] 变为 (2 OR 8) = 10 ，结果数组为 nums = [0,6,5,10] 。
- 选择 i = 2 和 j = 3 ，同时将 nums[2] 变为 (5 AND 10) = 0 且 nums[3] 变为 (5 OR 10) = 15 ，结果数组为 nums = [0,6,0,15] 。
从最终数组里选择元素 15 和 6 ，平方和为 152 + 62 = 261 。
261 是可以得到的最大结果。
```

**示例 2：**
```txt
**输入：**nums = [4,5,4,7], k = 3
**输出：**90
**解释：**不需要执行任何操作。
选择元素 7 ，5 和 4 ，平方和为 72 + 52 + 42 = 90 。
90 是可以得到的最大结果。
```

**提示：**

- `1 <= k <= nums.length <= 10^5`
- `1 <= nums[i] <= 10^9`

#### 提示 1
对于同一个比特，由于AND和OR不会改变都为0和都为1的的情况，所有操作等价。

把一个数的0和另一个数的同一个比特位上的1**交换**。

#### 提示 2

设交换前两个数是$x和y$ ，且$x > y$。把小的数上的给大的数，假设交换后$x$增加了$d$，那么$y$也减少了$d$。
交换前：$x^2 + y^2$.
交换后：$(x + d)^2 + (y - d)^2=x^2 + 2d(x - y) + y^2$.
这说明应该通过交换，让一个数越来越大好、
相当于把1都聚集在一个数中，比分散在不同的数更好。

#### 提示 3

由于可以操作任意次数，那么一定可以$\lceil组装\rfloor$出尽量大的数，做法如下：
1. 对于每个比特位，统计$nums$在这个比特位上有多少个1，记一个长度不超过30的数组$cnt$($10^9<2^{30}$).
2. 循环$k$次。
3. 每次循环，组装一个数(记作$x$)：遍历$cnt$，只要$cnt[i] > 0$就将其减一，同时将$2^i$加到$x$中。这样相当于把1尽量的聚集在一个数中。
4. 把$x^2$加到答案中。

```c++
class Solution {
public:
    int maxSum(vector<int>& nums, int k) {
        const int MOD = 1e9 + 7;
        int cnt[30]{};
        for(int x : nums) {
            for(int i = 0; i < 30; ++i) {
                cnt[i] += (x >> i) & 1;
            }
        }
        long long ans = 0;
        while(k--) {
            int x = 0;
            for(int i = 0; i < 30; ++i) {
                if(cnt[i] > 0) {
                    --cnt[i];
                    x |= 1 << i;
                }
            }
            ans = (ans + (long long) x * x) % MOD;
        }
        return ans;
    }
};
```