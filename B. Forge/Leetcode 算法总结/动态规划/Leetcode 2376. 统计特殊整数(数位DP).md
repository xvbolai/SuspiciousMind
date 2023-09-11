#### [2376. 统计特殊整数](https://leetcode.cn/problems/count-special-integers/)
如果一个正整数每一个数位都是 **互不相同** 的，我们称它是 **特殊整数** 。
给你一个 **正** 整数 `n` ，请你返回区间 `[1, n]` 之间特殊整数的数目。

**示例 1：**

```
输入：n = 20
输出：19
解释：1 到 20 之间所有整数除了 11 以外都是特殊整数。所以总共有 19 个特殊整数。
```
**示例 2：**
```
输入：n = 5
输出：5
解释：1 到 5 所有整数都是特殊整数。
```

**示例 3：**
```
输入：n = 135
输出：110
解释：从 1 到 135 总共有 110 个整数是特殊整数。
不特殊的部分数字为：22 ，114 和 131 。
```

### 前置知识：位运算与集合论

![[Pasted image 20230608161342.png]]
![[Pasted image 20230608161407.png]]
![[Pasted image 20230608161532.png]]
![[Pasted image 20230608161553.png]]
```c++
class Solution {
public:
    int countSpecialNumbers(int n) {
        
        string s = to_string(n);
        n = s.length();
        int mem[n][1<<10];
        memset(mem, -1, sizeof(mem));
        function<int(int, int, bool, bool)> dfs = [&](int i, int mask, bool is_limit, bool is_num) -> int {
            if(i == n) {
                return is_num;
            }
            if(!is_limit && is_num && mem[i][mask] != -1) return mem[i][mask];
            int res = 0;
            if(!is_num) {
                res = dfs(i + 1, mask, false, false);
            }
            int up = is_limit ? s[i] - '0' : 9;
            for(int d = 1 - int(is_num); d <= up; ++d) {
                if(((mask >> d) & 1) == 0) {
                    res += dfs(i + 1, mask | (1 << d), is_limit && d == up, true);
                }
            }
            if(!is_limit && is_num) mem[i][mask] = res;
            return res;
        };

        return dfs(0, 0, true, false);

    }
};
```
![[Pasted image 20230608161750.png]]
### 强化训练（数位 DP）

- [233. 数字 1 的个数](https://leetcode.cn/problems/number-of-digit-one/)（[题解](https://leetcode.cn/problems/number-of-digit-one/solution/by-endlesscheng-h9ua/)）
-  [面试题 17.06. 2出现的次数](https://leetcode.cn/problems/number-of-2s-in-range-lcci/)（[题解](https://leetcode.cn/problems/number-of-2s-in-range-lcci/solution/by-endlesscheng-x4mf/)）
- [600. 不含连续1的非负整数](https://leetcode.cn/problems/non-negative-integers-without-consecutive-ones/)（[题解](https://leetcode.cn/problems/non-negative-integers-without-consecutive-ones/solution/by-endlesscheng-1egu/)）
- [902. 最大为 N 的数字组合](https://leetcode.cn/problems/numbers-at-most-n-given-digit-set/) √
- [1067. 范围内的数字计数](https://leetcode.cn/problems/digit-count-in-range/)
-  [1397. 找到所有好字符串](https://leetcode.cn/problems/find-all-good-strings/)（有难度，需要结合一个经典字符串算法）