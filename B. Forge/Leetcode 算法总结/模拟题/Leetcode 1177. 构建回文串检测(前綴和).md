#### [1177. 构建回文串检测](https://leetcode.cn/problems/can-make-palindrome-from-substring/)
给你一个字符串 `s`，请你对 `s` 的子串进行检测。

每次检测，待检子串都可以表示为 `queries[i] = [left, right, k]`。我们可以 **重新排列** 子串 `s[left], ..., s[right]`，并从中选择 **最多** `k` 项替换成任何小写英文字母。

如果在上述检测过程中，子串可以变成回文形式的字符串，那么检测结果为 `true`，否则结果为 `false`。

返回答案数组 `answer[]`，其中 `answer[i]` 是第 `i` 个待检子串 `queries[i]` 的检测结果。

注意：在替换时，子串中的每个字母都必须作为 独立的 项进行计数，也就是说，如果 `s[left..right] = "aaa"` 且 `k = 2`，我们只能替换其中的两个字母。（另外，任何检测都不会修改原始字符串 `s`，可以认为每次检测都是独立的）

**示例：**

```txt
输入：s = "abcda", queries = [[3,3,0],[1,2,0],[0,3,1],[0,3,2],[0,4,1]]
输出：[true,false,false,true,true]
解释：
queries[0] : 子串 = "d"，回文。
queries[1] : 子串 = "bc"，不是回文。
queries[2] : 子串 = "abcd"，只替换 1 个字符是变不成回文串的。
queries[3] : 子串 = "abcd"，可以变成回文的 "abba"。 也可以变成 "baab"，先重新排序变成 "bacd"，然后把 "cd" 替换为 "ab"。
queries[4] : 子串 = "abcda"，可以变成回文的 "abcba"。
```



```c++
class Solution {
public:
    vector<bool> canMakePaliQueries(string s, vector<vector<int>>& queries) {
        int n = s.length(), qn = queries.size();
        vector<int> sum(n + 1);
        sum[0] = 0;
        for(int i = 0; i < n; ++i) {
            int bit = 1 << (s[i] - 'a');
            sum[i + 1] = (sum[i] ^ bit);
        }
        vector<bool> ans(qn);
        for(int i = 0; i < qn; ++i) {
            auto t = queries[i];
            int m = __builtin_popcount(sum[t[1] + 1] ^ sum[t[0]]);
            ans[i] = m / 2 <= t[2];
        }
        return ans;
    }
};
```


#### 相似题目（前缀和+异或）
- [1371. 每个元音包含偶数次的最长子字符串](https://leetcode.cn/problems/find-the-longest-substring-containing-vowels-in-even-counts/)
- [1542. 找出最长的超赞子字符串](https://leetcode.cn/problems/find-longest-awesome-substring/)
- [1915. 最美子字符串的数目](https://leetcode.cn/problems/number-of-wonderful-substrings/)，[题解](https://leetcode.cn/problems/number-of-wonderful-substrings/solution/qian-zhui-he-chang-jian-ji-qiao-by-endle-t57t/)

#### 更多前缀和题目：

- [560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)
- [974. 和可被 K 整除的子数组](https://leetcode.cn/problems/subarray-sums-divisible-by-k/)
- [1590. 使数组和能被 P 整除](https://leetcode.cn/problems/make-sum-divisible-by-p/)
- [523. 连续的子数组和](https://leetcode.cn/problems/continuous-subarray-sum/)
- [525. 连续数组](https://leetcode.cn/problems/contiguous-array/)

#### [1371. 每个元音包含偶数次的最长子字符串](https://leetcode.cn/problems/find-the-longest-substring-containing-vowels-in-even-counts/)

给你一个字符串 `s` ，请你返回满足以下条件的最长子字符串的长度：每个元音字母，即 'a'，'e'，'i'，'o'，'u' ，在子字符串中都恰好出现了偶数次。

**示例 1：**

```
输入：s = "eleetminicoworoep"
输出：13
解释：最长子字符串是 "leetminicowor" ，它包含 e，i，o 各 2 个，以及 0 个 a，u。
```

**示例 2：**

```
输入：s = "leetcodeisgreat"
输出：5
解释：最长子字符串是 "leetc" ，其中包含 2 个 e 。
```

```c++
class Solution {
public:
    int findTheLongestSubstring(string s) {
        int index[1 << 6];
        memset(index, -1, sizeof(index));
        int idx = 0, ans = 0;
        index[0] = 0;
        for(int i = 0; i < s.size(); ++i) {
            if(s[i] == 'a') idx ^= 1;
            else if(s[i] == 'e') idx ^= 2;
            else if(s[i] == 'i') idx ^= 4;
            else if(s[i] == 'o') idx ^= 8;
            else if(s[i] == 'u') idx ^= 16;
            if(index[idx] != -1) ans = max(ans, i + 1 - index[idx]);
            else index[idx] = i + 1;
        }
        return ans;
    }
};
```

#### [1542. 找出最长的超赞子字符串](https://leetcode.cn/problems/find-longest-awesome-substring/)

给你一个字符串 `s` 。请返回 `s` 中最长的 **超赞子字符串** 的长度。

「超赞子字符串」需满足满足下述两个条件：
- 该字符串是 `s` 的一个非空子字符串
- 进行任意次数的字符交换后，该字符串可以变成一个回文字符串

**示例 1：**

```
输入：s = "3242415"
输出：5
解释："24241" 是最长的超赞子字符串，交换其中的字符后，可以得到回文 "24142"
```

**示例 2：**

```
输入：s = "12345678"
输出：1
```

**示例 3：**

```
输入：s = "213123"
输出：6
解释："213123" 是最长的超赞子字符串，交换其中的字符后，可以得到回文 "231132"
```

**示例 4：**

```
输入：s = "00"
输出：2
```

```c++
class Solution {
public:
    int longestAwesome(string s) {
        int index[1<<10];
        memset(index, -1, sizeof(index));
        int ans = 0, idx = 0, n = s.length();
        index[idx] = 0;
        for(int i = 0; i < n; ++i) {
            idx ^= (1 << (s[i] - '0'));
            if(index[idx] != -1) {
                ans = max(ans, i + 1 - index[idx]);
            } else {
                index[idx] = i + 1;
            }
            for(int j = 0; j < 10; ++j) {
                if(index[idx ^ (1 << j)] != -1) {
                    ans = max(ans, i + 1 - index[idx ^ (1 << j)]);
                }
            }
        }
        return ans;
    }
};
```

#### [1915. 最美子字符串的数目](https://leetcode.cn/problems/number-of-wonderful-substrings/)

如果某个字符串中 **至多一个** 字母出现 **奇数** 次，则称其为 **最美** 字符串。

- 例如，`"ccjjc"` 和 `"abab"` 都是最美字符串，但 `"ab"` 不是。

给你一个字符串 `word` ，该字符串由前十个小写英文字母组成（`'a'` 到 `'j'`）。请你返回 `word` 中 **最美非空子字符串** 的数目。如果同样的子字符串在 `word` 中出现多次，那么 应当对 **每次出现** 分别计数。

**子字符串** 是字符串中的一个连续字符序列。

**示例 1：**

```
输入：word = "aba"
输出：4
解释：4 个最美子字符串如下所示：
- "aba" -> "a"
- "aba" -> "b"
- "aba" -> "a"
- "aba" -> "aba"
```

**示例 2：**

```
输入：word = "aabb"
输出：9
解释：9 个最美子字符串如下所示：
- "aabb" -> "a"
- "aabb" -> "aa"
- "aabb" -> "aab"
- "aabb" -> "aabb"
- "aabb" -> "a"
- "aabb" -> "abb"
- "aabb" -> "b"
- "aabb" -> "bb"
- "aabb" -> "b"
```

**示例 3：**

```
输入：word = "he"
输出：2
解释：2 个最美子字符串如下所示：
- "he" -> "h"
- "he" -> "e"
```

```c++
class Solution {
public:
    long long wonderfulSubstrings(string word) {
        long long ans = 0;
        int sum[1 << 10] = {0};
        sum[0] = 1;
        int idx = 0;
        for(int i = 0; i < word.size(); ++i) {
            idx ^= (1 << (word[i] - 'a'));
            ans += sum[idx]; // 统计偶数得前缀和
            for(int j = 0; j < 10; ++j) {
                ans += sum[idx ^ (1 << j)]; // 统计奇数前缀和
            }
            ++sum[idx];
        }
        return ans;
    }
};
```