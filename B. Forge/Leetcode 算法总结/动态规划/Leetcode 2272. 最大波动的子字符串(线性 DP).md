字符串的 **波动** 定义为子字符串中出现次数 **最多** 的字符次数与出现次数 **最少** 的字符次数之差。

给你一个字符串 `s` ，它只包含小写英文字母。请你返回 `s` 里所有 **子字符串的** **最大波动** 值。

**子字符串** 是一个字符串的一段连续字符序列。

**示例 1：**

```
输入：s = "aababbb"
输出：3
解释：
所有可能的波动值和它们对应的子字符串如以下所示：
- 波动值为 0 的子字符串："a" ，"aa" ，"ab" ，"abab" ，"aababb" ，"ba" ，"b" ，"bb" 和 "bbb" 。
- 波动值为 1 的子字符串："aab" ，"aba" ，"abb" ，"aabab" ，"ababb" ，"aababbb" 和 "bab" 。
- 波动值为 2 的子字符串："aaba" ，"ababbb" ，"abbb" 和 "babb" 。
- 波动值为 3 的子字符串 "babbb" 。
所以，最大可能波动值为 3 。
```
**示例 2：**
```
**输入：**s = "abcde"
**输出：**0
**解释：**
s 中没有字母出现超过 1 次，所以 s 中每个子字符串的波动值都是 0 。
```

**提示：**

- $1 <= s.length <= 10^4$
- `s`  只包含小写英文字母。

#### 提示 1

根据题意，最大波动值只由 $s$ 中的两**种**字符决定，至于是哪两种我们还不知道，我们可以枚举这两种字符的所有可能值。

由于 $s$ 只包含小写字母，我们可以从 26 个小写字母中选出 2 个不同的字母，并假设这两个字母为答案子串中出现次数最多的和最少的。这一共需要枚举$A_{26}^{2}=26\times25=650$种不同的字母组合。

#### 提示 2

假设出现次数最多的字符为 $a$，出现次数最少的字符为 $b$。由于题目求的是这两个字符出现次数的差，我们可以把 $a$ 视作 $1$，$b$ 视作 $−1$，其余字符视作 $0$,则本题转换成了一个类似 [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/) 的问题。

#### 提示 3

接上文。注意 $a$ 和 $b$ 必须都出现在子串中，不能把只有 $a$ 的子串作为答案。

我们可以用变量 $diff$ 维护 $a$ 和 $b$ 的出现次数之差，初始值为 $0$。

同时用另一个变量 $diffWithB$ 维护**包含了** $b$ 的 $a$ 和 $b$ 的出现次数之差,初始为 $−∞$ , 因为还没有遇到 $b$。

遍历字符串 $s$：
- 当遇到 $a$ 时，diff 和 diffWithB 均加一。
- 当遇到 $b$ 时，diff 减一，diffWithB 记录此时的 diff 值。若 diff 为负则将其置为 0。

统计所有 diffWithB 的最大值，即为答案。若 $s$ 只有一种字符则答案为 0。

```c++
class Solution {
public:
    int largestVariance(string s) {
        int ans = 0;
        // for(char a = 'a'; a <= 'z'; ++a) {
        //     for(char b = 'a'; b <= 'z'; ++b) {
        //         if(a == b) continue;
        //         int diff = 0, diff_with_b = -s.length();
        //         for(char ch : s) {
        //             if(ch == a) {
        //                 ++diff;
        //                 ++diff_with_b;
        //             } else if(ch == b) {
        //                 diff_with_b = --diff;
        //                 diff = max(0, diff);
        //             }
        //             ans = max(ans, diff_with_b);
        //         }
        //     }
        // }
        int diff[26][26] = {}, diff_with_b[26][26];
        memset(diff_with_b, 0x80, sizeof(diff_with_b));
        for(char ch : s) {
            ch -= 'a';
            for(int i = 0; i < 26; ++i) {
                if(i == ch) continue;
                ++diff[ch][i];
                ++diff_with_b[ch][i];
                diff_with_b[i][ch] = --diff[i][ch];
                diff[i][ch] = max(diff[i][ch], 0);
                ans = max({ans, diff_with_b[ch][i], diff_with_b[i][ch]});
            }
        }
        return ans;
    }
};
```