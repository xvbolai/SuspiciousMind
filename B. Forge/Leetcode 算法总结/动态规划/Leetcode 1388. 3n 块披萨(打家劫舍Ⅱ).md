给你一个披萨，它由 3n 块不同大小的部分组成，现在你和你的朋友们需要按照如下规则来分披萨：

- 你挑选 **任意** 一块披萨。
- Alice 将会挑选你所选择的披萨逆时针方向的下一块披萨。
- Bob 将会挑选你所选择的披萨顺时针方向的下一块披萨。
- 重复上述过程直到没有披萨剩下。

每一块披萨的大小按顺时针方向由循环数组 `slices` 表示。

请你返回你可以获得的披萨大小总和的最大值。

**示例 1：**
![[Pasted image 20230818212934.png|400|center]]
```
**输入：**slices = [1,2,3,4,5,6]
**输出：**10
**解释：**选择大小为 4 的披萨，Alice 和 Bob 分别挑选大小为 3 和 5 的披萨。然后你选择大小为 6 的披萨，Alice 和 Bob 分别挑选大小为 2 和 1 的披萨。你获得的披萨总大小为 4 + 6 = 10 。
```

**示例 2：**

![[Pasted image 20230818213008.png]]

```
**输入：**slices = [8,9,8,6,1,1]
**输出：**16
**解释：**两轮都选大小为 8 的披萨。如果你选择大小为 9 的披萨，你的朋友们就会选择大小为 8 的披萨，这种情况下你的总和不是最大的。
```
**提示：**

- `1 <= slices.length <= 500`
- `slices.length % 3 == 0`
- `1 <= slices[i] <= 1000`

在$3n$个数中挑选处不相邻的$n$个数。定义$f[i][j]$表示在数组$nums$的前$i$个数中选择$j$个不相邻的数的最大和。
$$
f[i][j] = max(f[i - 1][j], f[i - 2][j - 1] + nums[i])
$$

```c++
class Solution {
public:

    int maxSizeSlices(vector<int>& slices) {
        int n = slices.size() / 3;
        auto fun = [&](vector<int>& nums) -> int {
            int m = nums.size();
            vector<vector<int>> f(m + 2, vector<int>(n + 1));
            for(int i = 0; i < m; ++i) {
                for(int j = 0; j < n; ++j) {
                    f[i + 2][j + 1] = max(f[i + 1][j + 1], f[i][j] + nums[i]);
                }
            }
            return f[m + 1][n];
        };
        vector<int> nums(slices.begin(), slices.end() - 1);
        int ans1 = fun(nums);

        nums = vector<int>(slices.begin() + 1, slices.end());
        int ans = fun(nums);
        return max(ans, ans1);
    }
};
```