#### [6396. 统计整数数目](https://leetcode.cn/problems/count-of-integers/)

#数位DP #困难 

给你两个数字字符串 `num1` 和 `num2` ，以及两个整数 `max_sum` 和 `min_sum` 。如果一个整数 `x` 满足以下条件，我们称它是一个好整数：

- `num1 <= x <= num2`
- `min_sum <= digit_sum(x) <= max_sum`.

请你返回好整数的数目。答案可能很大，请返回答案对 `109 + 7` 取余后的结果。

注意，`digit_sum(x)` 表示 `x` 各位数字之和。

**示例 1：**

```
输入：num1 = "1", num2 = "12", min_num = 1, max_num = 8
输出：11
解释：总共有 11 个整数的数位和在 1 到 8 之间，分别是 1,2,3,4,5,6,7,8,10,11 和 12 。所以我们返回 11 。
```

**示例 2：**

```
输入：num1 = "1", num2 = "5", min_num = 1, max_num = 5
输出：5
解释：数位和在 1 到 5 之间的 5 个整数分别为 1,2,3,4 和 5 。所以我们返回 5 。
```


```c++
class Solution {
    const int MOD = 1e9 + 7;
public:
    int f(string s, int min_sum, int max_sum) {
        int n = s.length(), mem[n][min(9 * n, max_sum) + 1];
        memset(mem, -1, sizeof(mem));
        function<int(int, int, bool)> dfs = [&](int i, int sum, bool is_limit) -> int {
            if(sum > max_sum) return 0;
            if(i == n) return sum >= min_sum;
            if(!is_limit && mem[i][sum] != -1) return mem[i][sum];
            int res = 0;
            int up = is_limit ? s[i] - '0' : 9;
            for(int d = 0; d <= up; ++d) {
                res = (res + dfs(i + 1, sum + d, is_limit && d == up)) % MOD;
            }
            if(!is_limit) mem[i][sum] = res;
            return res;
        };
        return dfs(0, 0, true);
    }

    int count(string num1, string num2, int min_sum, int max_sum) {
        int ans = f(num2, min_sum, max_sum) - f(num1, min_sum, max_sum) + MOD;
        int sum = 0;
        for(char c : num1) sum += c - '0';
        ans += min_sum <= sum && sum <= max_sum;
        return ans % MOD;
    }
};
```

```python
class Solution:
    def count(self, num1: str, num2: str, min_sum: int, max_sum: int) -> int:
        MOD = 10 ** 9 + 7
        def f(s: string) -> int:
            @cache  # 记忆化搜索
            def f(i: int, sum: int, is_limit: bool) -> int:
                if sum > max_sum:  # 非法
                    return 0
                if i == len(s):
                    return int(sum >= min_sum)
                res = 0
                up = int(s[i]) if is_limit else 9
                for d in range(up + 1):  # 枚举要填入的数字 d
                    res += f(i + 1, sum + d, is_limit and d == up)
                return res % MOD
            return f(0, 0, True)
        ans = f(num2) - f(num1) + (min_sum <= sum(map(int, num1)) <= max_sum)
        return ans % MOD
```