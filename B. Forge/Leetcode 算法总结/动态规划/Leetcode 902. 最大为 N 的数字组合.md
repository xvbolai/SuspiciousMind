#### [902. 最大为 N 的数字组合](https://leetcode.cn/problems/numbers-at-most-n-given-digit-set/)

给定一个按 **非递减顺序** 排列的数字数组 `digits` 。你可以用任意次数 `digits[i]` 来写的数字。例如，如果 `digits = ['1','3','5']`，我们可以写数字，如 `'13'`, `'551'`, 和 `'1351315'`。
返回 _可以生成的小于或等于给定整数 `n` 的正整数的个数_ 。
**示例 1：**

```
输入：digits = ["1","3","5","7"], n = 100
输出：20
解释：
可写出的 20 个数字是：
1, 3, 5, 7, 11, 13, 15, 17, 31, 33, 35, 37, 51, 53, 55, 57, 71, 73, 75, 77.
```

**示例 2：**

```
输入：digits = ["1","4","9"], n = 1000000000
输出：29523
解释：
我们可以写 3 个一位数字，9 个两位数字，27 个三位数字，
81 个四位数字，243 个五位数字，729 个六位数字，
2187 个七位数字，6561 个八位数字和 19683 个九位数字。
总共，可以使用D中的数字写出 29523 个整数。

```


```c++
class Solution {
public:
    int atMostNGivenDigitSet(vector<string>& digits, int n) {
        string s = to_string(n);
        int mem[s.length()];
        memset(mem, -1, sizeof(mem));
        function<int(int, bool, bool)> dfs = [&](int i, bool is_limit, bool is_num) -> int {
            if(i == s.length()) {
                return int(is_num);
            }
            if(!is_limit && is_num && mem[i] != -1) return mem[i];
            int res = 0;
            if(!is_num) {
                res = dfs(i + 1, false, false);
            }
            char up = is_limit ? s[i] : '9';
            for(auto d : digits) {
                if(d[0] > up) break;
                res += dfs(i + 1, is_limit && d[0] == up, true);
            }
            if(!is_limit && is_num) mem[i] = res;
            return res;
        };

        return dfs(0, true, false);
    }
};
```