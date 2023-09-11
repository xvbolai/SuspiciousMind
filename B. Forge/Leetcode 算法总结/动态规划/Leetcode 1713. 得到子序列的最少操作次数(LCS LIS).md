给你一个数组 `target` ，包含若干 **互不相同** 的整数，以及另一个整数数组 `arr` ，`arr` **可能** 包含重复元素。

每一次操作中，你可以在 `arr` 的任意位置插入任一整数。比方说，如果 `arr = [1,4,1,2]` ，那么你可以在中间添加 `3` 得到 `[1,4,**3**,1,2]` 。你可以在数组最开始或最后面添加整数。

请你返回 **最少** 操作次数，使得 `target` 成为 `arr` 的一个子序列。

一个数组的 **子序列** 指的是删除原数组的某些元素（可能一个元素都不删除），同时不改变其余元素的相对顺序得到的数组。比方说，`[2,7,4]` 是 `[4,**2**,3,**7**,2,1,**4**]` 的子序列（加粗元素），但 `[2,4,2]` 不是子序列。

**示例 1：**

```
输入：target = [5,1,3], arr = [9,4,2,3,4]
输出：2
解释：你可以添加 5 和 1 ，使得 arr 变为 [5,9,4,1,2,3,4] ，target 为 arr 的子序列。
```

**示例 2：**

```
输入：target = [6,4,8,1,3,2], `arr` = [4,7,6,2,3,8,6,1]
输出：3
```

### 最长公共子序列(LCS)

```C++
class Solution {
public:
    int minOperations(vector<int>& target, vector<int>& arr) {
        const int n = target.size();
        const int m = arr.size();
        int dp[2][n];   //这是咱的表格
        memset(dp, 0, sizeof(dp));  //先都填上零
        for(int i = 1; i <= n; ++i){  //从第一个元素开始遍历
            int k = i & 1, pre = !k;   //滚动数组，k与pre总是不同的两行
            for(int j = 1; j <= m; ++j){
                if(target[i - 1] == arr[j - 1])   //如果两元素相等
                    dp[k][j] = dp[pre][j - 1] + 1;   //来自于前一元素的最长长度+1  
                else
                    dp[k][j] = max(dp[k][j - 1], dp[pre][j]);//否则我们取两者前一元素的最长长度
            }
        }
        return n - dp[n & 1][m];
    }
};
```

### 最长上升子序列(LIS)

我们可以把target里的元素替换成它们的下标，这样最长公共子序列就变成了LIS求最长升序子序列了

```C++
class Solution { 
	public: int minOperations(vector<int>& target, vector<int>& arr) { 
		const int n = target.size(); 
		const int INF = 0x3f3f3f3f; 
		unordered_map<int, int> mp; 
		for(int i = 0; i < n; ++ i) { 
			mp[target[i]] = i; 
		} 
		int ans[n]; 
		memset(ans, INF, sizeof(ans)); 
		for(const int val : arr) { 
			if(mp.count(val)) { 
				*lower_bound(ans, ans + n, mp[val]) = mp[val]; 
			} 
		} 
		return n - (lower_bound(ans, ans + n, INF) - ans); 
	} 
};
```
