#简单 #数位DP 
你在一家生产小球的玩具厂工作，有 `n` 个小球，编号从 `lowLimit` 开始，到`highLimit` 结束（包括 `lowLimit` 和 `highLimit` ，即 `n == highLimit -lowLimit + 1`）。另有无限数量的盒子，编号从 `1` 到 `infinity` 。
你的工作是将每个小球放入盒子中，其中盒子的编号应当等于小球编号上每位数字的和。例如，编号 `321` 的小球应当放入编号 `3 + 2 + 1 = 6` 的盒子，而编号 `10`的小球应当放入编号 `1 + 0 = 1` 的盒子。

给你两个整数 `lowLimit` 和 `highLimit` ，返回放有最多小球的盒子中的小球数量。如果有多个盒子都满足放有最多小球，只需返回其中任一盒子的小球数量。

**示例 1：**

```
输入：lowLimit = 1, highLimit = 10
输出：2
解释：
盒子编号：1 2 3 4 5 6 7 8 9 10 11 ...
小球数量：2 1 1 1 1 1 1 1 1 0  0  ...
编号 1 的盒子放有最多小球，小球数量为 2 。
```

**示例 2：**

```
输入：lowLimit = 5, highLimit = 15
输出：2
解释：
盒子编号：1 2 3 4 5 6 7 8 9 10 11 ...
小球数量：1 1 1 1 2 2 1 1 1 0  0  ...
编号 5 和 6 的盒子放有最多小球，每个盒子中的小球数量都是 2 。
```


```c++
class Solution {
public:
    int countBalls(int lowLimit, int highLimit) {
        string a = to_string(lowLimit), b = to_string(highLimit);
        int n = b.length();
        a.insert(0, n - a.length(), '0');
        function<int(int,int,bool,bool)> dfs = [&] (int i,int sum,bool is_up_limit,bool is_down_limit) -> int {
            if(sum < 0) return 0;
            if(i == n)  return sum == 0;
            int up = is_up_limit ? b[i] - '0' : 9;
            int down = is_down_limit ? a[i] - '0' : 0;
            int ans = 0;
            for(int j = down; j <= up; ++j) {
                ans += dfs(i + 1, sum - j, j == up && is_up_limit, is_down_limit && j == down);
            }
            return ans;
        };
        int ans = 0;
        for(int i = 0; i < 46; ++i) {
            ans = max(ans, dfs(0, i, true, true));
        }
        return ans;
    }
};
```

```python
class Solution:
    def countBalls(self, lowLimit: int, highLimit: int) -> int:
        a, b = str(lowLimit), str(highLimit)
        n = len(b)
        a = '0' * (n - len(a)) + a
        
        @cache
        def dfs(i, sum, is_up_limit, is_down_limit):
            if i < 0: 
                return 0
            if i == n:
                return int(sum == 0)

            up = int(b[i]) if is_up_limit else 9
            down = int(a[i]) if is_down_limit else 0
            ans = 0
            for j in range(down, up + 1):
                ans += dfs(i + 1, sum - j, j == up and is_up_limit, j == down and is_down_limit)
            return ans

        return max(dfs(0, i, True, True) for i in range(46))
```