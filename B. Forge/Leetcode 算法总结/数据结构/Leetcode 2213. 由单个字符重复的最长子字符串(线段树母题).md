给你一个下标从 **0** 开始的字符串 `s` 。另给你一个下标从 **0** 开始、长度为 `k` 的字符串 `queryCharacters` ，一个下标从 `0` 开始、长度也是 `k` 的整数 **下标** 数组 `queryIndices` ，这两个都用来描述 `k` 个查询。

第 `i` 个查询会将 `s` 中位于下标 `queryIndices[i]` 的字符更新为 `queryCharacters[i]` 。

返回一个长度为 `k` 的数组 `lengths` ，其中 `lengths[i]` 是在执行第 `i` 个查询 **之后** `s` 中仅由 **单个字符重复** 组成的 **最长子字符串** 的 **长度** _。_

**示例 1：**

```
输入：s = "babacc", queryCharacters = "bcb", queryIndices = [1,3,3]
输出：[3,3,4]
解释：
- 第 1 次查询更新后 s = "bbbacc" 。由单个字符重复组成的最长子字符串是 "bbb" ，长度为 3 。
- 第 2 次查询更新后 s = "bbbccc" 。由单个字符重复组成的最长子字符串是 "bbb" 或 "ccc"，长度为 3 。
- 第 3 次查询更新后 s = "bbbbcc" 。由单个字符重复组成的最长子字符串是 "bbbb" ，长度为 4 。
因此，返回 [3,3,4] 。
```

```c++
class Solution {
public:
    string s;
    vector<int> pre, suf, max;
    void build(int l, int r, int t) {
        if(l == r) {
            pre[t] = suf[t] = max[t] = 1;
            return;
        }
        int m = (l + r) / 2;
        build(l, m, t<<1);
        build(m + 1, r, t<<1|1);
        maintain(l, r, t);
    }
    void maintain(int l, int r, int t) {
        pre[t] = pre[t<<1];
        suf[t] = suf[t<<1|1];
        max[t] = std::max(max[t<<1], max[t<<1|1]);
        int m = (l + r) / 2;
        if(s[m] == s[m - 1]) {
            if(pre[t<<1] == m - l + 1) pre[t] += pre[t<<1|1];
            if(pre[t<<1|1] == r - m) suf[t] += suf[t<<1];
            max[t] = std::max(max[t], suf[t<<1] + pre[t<<1|1]);
        }
    }
    void update(int l, int r, int i, int t){
        if(l == r) return;
        int m = (l + r) / 2;
        if(i <= m) update(l, m, i, t<<1);
        else update(m + 1, r, i, t<<1|1);
        maintain(l, r, t);
    }

    vector<int> longestRepeating(string s, string queryCharacters, vector<int>& queryIndices) {
        this->s = s;
        int n = s.length(), m = queryIndices.size();
        pre.resize(n << 2);
        suf.resize(n << 2);
        max.resize(n << 2);
        build(1, n, 1);
        vector<int> ans(m);
        for (int i = 0; i < m; ++i) {
            this->s[queryIndices[i]] = queryCharacters[i];
            update(1, n, queryIndices[i] + 1, 1);
            ans[i] = max[1];
        }
        return ans;
    }
};
```