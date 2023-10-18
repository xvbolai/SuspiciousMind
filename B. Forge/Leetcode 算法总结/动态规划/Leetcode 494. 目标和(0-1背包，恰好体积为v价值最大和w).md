#### [494. 目标和](https://leetcode.cn/problems/target-sum/)

给你一个整数数组 `nums` 和一个整数 `target` 。

向数组中的每个整数前添加 `'+'` 或 `'-'` ，然后串联起所有整数，可以构造一个 **表达式** ：

- 例如，`nums = [2, 1]` ，可以在 `2` 之前添加 `'+'` ，在 `1` 之前添加 `'-'` ，然后串联起来得到表达式 `"+2-1"` 。

返回可以通过上述方法构造的、运算结果等于 `target` 的不同 **表达式** 的数目。

**示例 1：**

```
输入：nums = [1,1,1,1,1], target = 3
输出：5
解释：一共有 5 种方法让最终目标和为 3 。
-1 + 1 + 1 + 1 + 1 = 3
+1 - 1 + 1 + 1 + 1 = 3
+1 + 1 - 1 + 1 + 1 = 3
+1 + 1 + 1 - 1 + 1 = 3
+1 + 1 + 1 + 1 - 1 = 3
```

**示例 2：**

```
输入：nums = [1], target = 1
输出：1
```

原问题等同于： 找到nums一个正子集和一个负子集，使得总和等于target

我们假设P是正子集，N是负子集 例如： 假设nums = [1, 2, 3, 4, 5]，target = 3，一个可能的解决方案是+1-2+3-4+5 = 3 这里正子集P = [1, 3, 5]和负子集N = [2, 4]

那么让我们看看如何将其转换为子集求和问题：

```
                  sum(P) - sum(N) = target
sum(P) + sum(N) + sum(P) - sum(N) = target + sum(P) + sum(N)
                       2 * sum(P) = target + sum(nums)
```

因此，原来的问题已转化为一个求子集的和问题： 找到nums的一个子集 P，使得sum(P) = (target + sum(nums)) / 2

请注意，上面的公式已经证明$target + sum(nums)$必须是偶数，否则输出为0。


```c++
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int sum = 0;
        for(auto &v : nums)  sum += v;
        if(sum < target || (sum + target) % 2 || sum + target < 0) return 0;
        int n = (target + sum) / 2;
        vector<int> dp(n + 1);
        dp[0] = 1;
        for(auto v : nums) {
            for(int i = n; i >= v; --i) {
                dp[i] = dp[i] + dp[i - v];
            }
        }
        return dp[n];
    }
};
```

### 0-1背包三种问题

#### 1. 至多capacity，求方案数或最大价值和。

```python
@cache
def dfs(i, c):
	if i < 0:
		return 1
	if c < nums[i]:
		return dfs(i - 1, c)
	return dfs(i - 1, c) + dfs(i - 1, c - nums[i])

# 换成递推式                                           

f = [1] * (target + 1)

for x in nums:
	for c in range(target, x - 1, -1):
		f[c] = f[c] + f[c - x];
```

#### 2.恰好capacity，求方案数最大\\最少价值和。

```python
@cache
def dfs(i, c):
	if i < 0:
		return 1 if c == 0 else 0
	if c < nums[i]:
		return dfs(i - 1, c)
	return dfs(i - 1, c) + dfs(i - 1, c - nums[i])

# 换成递推式

f = [1] + [0] * target

for x in nums:
	for c in range(target, x - 1, -1):
		f[c] = f[c] + f[c - x];
```

#### 2. 至少capacity，求方案数最小价值和。

```python
@cache
def dfs(i, c):
	if i < 0:
		return 1 if c <= 0 else 0
	return dfs(i - 1, c) + dfs(i - 1, c - nums[i])

# 换成递推式

f = [1] + [0] * target

for x in nums:
	for c in range(target, -1, -1):
		f[c] = f[c] + f[max(c - x, 0)];
```