#中等 

给你两个下标从 **0** 开始的二进制字符串 `s1` 和 `s2` ，两个字符串的长度都是 `n` ，再给你一个正整数 `x` 。

你可以对字符串 `s1` 执行以下操作 **任意次** ：

- 选择两个下标 `i` 和 `j` ，将 `s1[i]` 和 `s1[j]` 都反转，操作的代价为 `x` 。
- 选择满足 `i < n - 1` 的下标 `i` ，反转 `s1[i]` 和 `s1[i + 1]` ，操作的代价为 `1` 。

请你返回使字符串 `s1` 和 `s2` 相等的 **最小** 操作代价之和，如果无法让二者相等，返回 `-1` 。

**注意** ，反转字符的意思是将 `0` 变成 `1` ，或者 `1` 变成 `0` 。

**示例 1：**

```txt
**输入：**s1 = "1100011000", s2 = "0101001010", x = 2
**输出：**4
**解释：**我们可以执行以下操作：
- 选择 i = 3 执行第二个操作。结果字符串是 s1 = "110_**11**_11000" 。
- 选择 i = 4 执行第二个操作。结果字符串是 s1 = "1101_**00**_1000" 。
- 选择 i = 0 和 j = 8 ，执行第一个操作。结果字符串是 s1 = "_**0**_1010010_**1**_0" = s2 。
总代价是 1 + 1 + 2 = 4 。这是最小代价和。
```

**示例 2：**

```txt
**输入：**s1 = "10110", s2 = "00011", x = 4
**输出：**-1
**解释：**无法使两个字符串相等.
```

**提示：**

- `n == s1.length == s2.length`
- `1 <= n, x <= 500`
- `s1` 和 `s2` 只包含字符 `'0'` 和 `'1'` 。

时间复杂度$O(n^2)$.
```c++
class Solution {
public:
    int minOperations(string s1, string s2, int x) {
        if(count(s1.begin(), s1.end(), '1') % 2 != count(s2.begin(), s2.end(), '1') % 2) return -1;
        int n = s1.length();
        int mem[n][n + 1][2];
        memset(mem, -1, sizeof(mem));
        function<int(int, int, bool)> dfs = [&](int i, int j, bool pre_rev) -> int {
            if(i < 0) return j || pre_rev ? INT_MAX / 2 : 0;
            int& res = mem[i][j][pre_rev];
            if(res != -1) return res;
            if((s1[i] == s2[i]) == !pre_rev) return dfs(i - 1, j, false);
            res = min(dfs(i - 1, j + 1, false) + x, dfs(i - 1, j, true) + 1);
            if(j) res = min(res, dfs(i - 1, j - 1, false));
            return res;
        };
        return dfs(n - 1, 0, false);
    }
};
```

时间复杂度$O(n)$.

```c++
class Solution {
public:
    int minOperations(string s1, string s2, int x) {
        if(s1 == s2) return 0;
        vector<int> dif;
        for(int i = 0; i < s1.length(); ++i) if(s1[i] != s2[i]) {
            dif.emplace_back(i);
        }
        if(dif.size() % 2) return -1;
        int f0 = 0, f1 = x;
        for(int i = 1; i < dif.size(); ++i) {
            int new_f = min(f1 + x, f0 + (dif[i] - dif[i - 1]) * 2);
            f0 = f1; 
            f1 = new_f;
        }
        return f1 / 2;
    }
};
```

