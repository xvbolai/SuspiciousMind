#### [209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)

给定一个含有 `n` 个正整数的数组和一个正整数 `target` **。**

找出该数组中满足其和 `≥ target` 的长度最小的 **连续子数组** `[numsl, numsl+1, ..., numsr-1, numsr]` ，并返回其长度**。**如果不存在符合条件的子数组，返回 `0` 。

**示例 1：**

```
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
```

**示例 2：**

```
输入：target = 4, nums = [1,4,4]
输出：1
```

**示例 3：**

```
输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0
```

```c++
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int ans = nums.size() + 1, left = 0, sum = 0;
        // for(int right = 0; right < nums.size(); ++right) {
        //     sum += nums[right];
        //     while(sum >= target) {
        //         ans = min(right - left + 1, ans);
        //         sum -= nums[left++];
        //     }
        // }
        for(int right = 0; right < nums.size(); ++right) {
            sum += nums[right];
            while(sum - nums[left] >= target) sum -= nums[left++];
            if(sum >= target) ans = min(right - left + 1, ans);
        }
        return ans <= nums.size() ? ans : 0;
    }
};
```



##### 相关题目

[209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/solution/biao-ti-xia-biao-zong-suan-cuo-qing-kan-k81nh/)

[713. 乘积小于 K 的子数组](https://leetcode.cn/problems/subarray-product-less-than-k/solution/xia-biao-zong-suan-cuo-qing-kan-zhe-by-e-jebq/) 

[3. 无重复字符的最长子串]( https://leetcode.cn/problems/longest-substring-without-repeating-characters/solution/xia-biao-zong-suan-cuo-qing-kan-zhe-by-e-iaks/ )

课后作业： 

[1004. 最大连续 1 的个数 III](https://leetcode.cn/problems/max-consecutive-ones-iii/) 

[1234. 替换子串得到平衡字符串](https://leetcode.cn/problems/replace-the-substring-for-balanced-string/ )

[1658. 将 x 减到 0 的最小操作数](https://leetcode.cn/problems/minimum-operations-to-reduce-x-to-zero/)

#### [713. 乘积小于 K 的子数组](https://leetcode.cn/problems/subarray-product-less-than-k/)

给你一个整数数组 `nums` 和一个整数 `k` ，请你返回子数组内所有元素的乘积严格小于 `k` 的连续子数组的数目。

**示例 1：**

```
输入：nums = [10,5,2,6], k = 100
输出：8
解释：8 个乘积小于 100 的子数组分别为：[10]、[5]、[2],、[6]、[10,5]、[5,2]、[2,6]、[5,2,6]。
需要注意的是 [10,5,2] 并不是乘积小于 100 的子数组。
```

**示例 2：**

```
输入：nums = [1,2,3], k = 0
输出：0
```

```c++
class Solution {
public:
    int numSubarrayProductLessThanK(vector<int>& nums, int k) {
        if(k <= 1) return 0;
        int left = 0, ans = 0, m = 1, n = nums.size();
        for(int right = 0; right < n; ++right) {
            m *= nums[right];
            while(m >= k) m /= nums[left++];
            ans += right - left + 1;
        }
        return ans;
    }
};
```

#### [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长子串** 的长度。

**示例 1:**

```
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

**示例 2:**

```
输入: s = "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

**示例 3:**

```
输入: s = "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_map<char, int> cnt;
        int n = s.length(), left = 0, ans = 0;
        for(int right = 0; right < n; ++right) {
            cnt[s[right]]++;
            while(cnt[s[right]] > 1) --cnt[s[left++]];
            ans = max(ans, right - left + 1);
        }
        return ans;
    }
};
```

#### [1004. 最大连续1的个数 III](https://leetcode.cn/problems/max-consecutive-ones-iii/)

给定一个二进制数组 `nums` 和一个整数 `k`，如果可以翻转最多 `k` 个 `0` ，则返回 *数组中连续 `1` 的最大个数* 。

**示例 1：**

```
输入：nums = [1,1,1,0,0,0,1,1,1,1,0], K = 2
输出：6
解释：[1,1,1,0,0,1,1,1,1,1,1]
粗体数字从 0 翻转到 1，最长的子数组长度为 6。
```

**示例 2：**

```
输入：nums = [0,0,1,1,0,0,1,1,1,0,1,1,0,0,0,1,1,1,1], K = 3
输出：10
解释：[0,0,1,1,1,1,1,1,1,1,1,1,0,0,0,1,1,1,1]
粗体数字从 0 翻转到 1，最长的子数组长度为 10。
```

```c++
class Solution {
public:
    int longestOnes(vector<int>& nums, int k) {
        int zero = 0, left = 0, ans = 0;
        for(int right = 0; right < nums.size(); ++right) {
            zero+= nums[right] == 0 ? 1 : 0;
            while(zero > k) {
                zero -= (nums[left++] == 0 ? 1 : 0);
            }
            ans = max(ans, right - left + 1);
        }
        return ans;
    }
};
```

#### [1234. 替换子串得到平衡字符串](https://leetcode.cn/problems/replace-the-substring-for-balanced-string/)

有一个只含有 `'Q', 'W', 'E', 'R'` 四种字符，且长度为 `n` 的字符串。

假如在该字符串中，这四个字符都恰好出现 `n/4` 次，那么它就是一个「平衡字符串」。

给你一个这样的字符串 `s`，请通过「替换一个子串」的方式，使原字符串 `s` 变成一个「平衡字符串」。

你可以用和「待替换子串」长度相同的 **任何** 其他字符串来完成替换。

请返回待替换子串的最小可能长度。

如果原字符串自身就是一个平衡字符串，则返回 `0`。

**示例 1：**

```shell
输入：s = "QWER"
输出：0
解释：s 已经是平衡的了。
```

**示例 2：**

```shell
输入：s = "QQWE"
输出：1
解释：我们需要把一个 'Q' 替换成 'R'，这样得到的 "RQWE" (或 "QRWE") 是平衡的。
```

**示例 3：**

```shell
输入：s = "QQQW"
输出：2
解释：我们可以把前面的 "QQ" 替换成 "ER"。 
```

**示例 4：**

```
输入：s = "QQQQ"
输出：3
解释：我们可以替换后 3 个 'Q'，使 s = "QWER"。
```

```c++
class Solution {
public:
    int balancedString(string s) {
        int n = s.length(), m = n / 4;
        unordered_map<char, int> cnt;
        for(auto &v : s) ++cnt[v];
        if(cnt['Q'] == m && cnt['W'] == m && cnt['E'] == m && cnt['R'] == m) return 0;
        int ans = n, left = 0;
        for(int right = 0; right < n; ++right) {
            --cnt[s[right]];
            while(cnt['Q'] <= m && cnt['W'] <= m && cnt['E'] <= m && cnt['R'] <= m) {
                ans = min(ans, right - left + 1);
                ++cnt[s[left++]];
            }
        }
        return ans;
    }
};
```

#### [1658. 将 x 减到 0 的最小操作数](https://leetcode.cn/problems/minimum-operations-to-reduce-x-to-zero/)

给你一个整数数组 `nums` 和一个整数 `x` 。每一次操作时，你应当移除数组 `nums` 最左边或最右边的元素，然后从 `x` 中减去该元素的值。请注意，需要 **修改** 数组以供接下来的操作使用。

如果可以将 `x` **恰好** 减到 `0` ，返回 **最小操作数** ；否则，返回 `-1` 。

**示例 1：**

```shell
输入：nums = [1,1,4,2,3], x = 5
输出：2
解释：最佳解决方案是移除后两个元素，将 x 减到 0。
```

**示例 2：**

```shell
输入：nums = [5,6,7,8,9], x = 4
输出：-1
```

**示例 3：**

```shell
输入：nums = [3,2,20,1,1,3], x = 10
输出：5
解释：最佳解决方案是移除后三个元素和前两个元素（总共 5 次操作），将 x 减到 0。
```

把问题转换成从 **nums** 中移除一个**最长**的子数组，使得剩余元素的和为 *x*。

换句话说，要从 **nums** 中找最长的子数组，其元素和等于 *s*−*x*，这里 *s* 为 **nums** 所有元素之和。



```c++
class Solution {
public:
    int minOperations(vector<int>& nums, int x) {
        x = accumulate(nums.begin(), nums.end(), 0) - x;
        if(x < 0) return -1;
        int ans = -1, left = 0, n = nums.size(), s = 0;
        for(int right = 0; right < n; ++right) {
            s += nums[right];
            while(s > x) {
                s -= nums[left++];
            }
            if(s == x)  ans = max(ans, right - left + 1);
        }
        return ans < 0 ? -1 : n - ans; 
    }
};
```

#### [2516. 每种字符至少取 K 个](https://leetcode.cn/problems/take-k-of-each-character-from-left-and-right/)

给你一个由字符 `'a'`、`'b'`、`'c'` 组成的字符串 `s` 和一个非负整数 `k` 。每分钟，你可以选择取走 `s` **最左侧** 还是 **最右侧** 的那个字符。你必须取走每种字符 **至少** `k` 个，返回需要的 **最少** 分钟数；如果无法取到，则返回 `-1` 。

**示例 1：**

```
输入：s = "aabaaaacaabc", k = 2
输出：8
解释：
从 s 的左侧取三个字符，现在共取到两个字符 'a' 、一个字符 'b' 。
从 s 的右侧取五个字符，现在共取到四个字符 'a' 、两个字符 'b' 和两个字符 'c' 。
共需要 3 + 5 = 8 分钟。
可以证明需要的最少分钟数是 8 。
```

**示例 2：**
```
输入：s = "a", k = 1
输出：-1
解释：无法取到一个字符 'b' 或者 'c'，所以返回 -1 。
```

在两端必须取走`a`、`b`和`c`至少k个，换句话说就是剩下的元素个数不大于`s`中`a`、`b`和`c`元素个数减去k的个数。对应可以想到使用滑动窗口，即同向双指针。

```c++
class Solution {
public:
    int takeCharacters(string s, int k) {
        unordered_map<char, int> cnt;
        for(auto &c : s) ++cnt[c];
        if(cnt['a'] < k || cnt['b'] < k || cnt['c'] < k) return -1;
        int n = s.length(), left = 0, ans = 0;
        for(int right = 0; right < n; ++right) {
            --cnt[s[right]];
            while(cnt['a'] < k || cnt['b'] < k || cnt['c'] < k) {
                ++cnt[s[left++]];
            }
            ans = max(right - left + 1, ans);
        }
        return n - ans;
    }   
};
```

====