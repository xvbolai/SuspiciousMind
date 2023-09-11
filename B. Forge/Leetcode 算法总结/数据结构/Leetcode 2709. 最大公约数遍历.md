#### [2709. 最大公约数遍历](https://leetcode.cn/problems/greatest-common-divisor-traversal/)
#数据结构 #困难

给你一个下标从 **0** 开始的整数数组 `nums` ，你可以在一些下标之间遍历。对于两个下标 `i` 和 `j`（`i != j`），当且仅当 `gcd(nums[i], nums[j]) > 1` 时，我们可以在两个下标之间通行，其中 `gcd` 是两个数的 **最大公约数** 。你需要判断 `nums` 数组中 **任意** 两个满足 `i < j` 的下标 `i` 和 `j` ，是否存在若干次通行可以从 `i` 遍历到 `j` 。如果任意满足条件的下标对都可以遍历，那么返回 `true` ，否则返回 `false` 。

**示例 1：**

```
输入：nums = [2,3,6]
输出：true
解释：这个例子中，总共有 3 个下标对：(0, 1) ，(0, 2) 和 (1, 2) 。
从下标 0 到下标 1 ，我们可以遍历 0 -> 2 -> 1 ，我们可以从下标 0 到 2 是因为 gcd(nums[0], nums[2]) = gcd(2, 6) = 2 > 1 ，从下标 2 到 1 是因为 gcd(nums[2], nums[1]) = gcd(6, 3) = 3 > 1 。
从下标 0 到下标 2 ，我们可以直接遍历，因为 gcd(nums[0], nums[2]) = gcd(2, 6) = 2 > 1 。同理，我们也可以从下标 1 到 2 因为 gcd(nums[1], nums[2]) = gcd(3, 6) = 3 > 1 。
```

**示例 2：**

```
输入：nums = [3,9,5]
输出：false
解释：我们没法从下标 0 到 2 ，所以返回 false 。
```

把 `nums` 中的每个位置看成一个点，把所有质数也都看成一个点。如果 `nums[i]` 被质数 `p` 整除，那么从位置点 `i` 向质数点 `p` 连一条边。因为每个数至多只能被 `log`⁡ 个质数整除，因此连边的总数是 $\mathcal{O}(n\log{A})$ 的。

```c++
#define MAXX ((int) 1e5)
bool inited = false;
vector<int> fac[MAXX + 10];

// 全局预处理每个数的质因数
void init() {
    if (inited) return;
    inited = true;

    for (int i = 2; i <= MAXX; i++) if (fac[i].empty()) for (int j = i; j <= MAXX; j += i) fac[j].push_back(i);
}

class Solution {
public:

    bool canTraverseAllPairs(vector<int>& nums) {
        init();

        int n = nums.size();
        int mx = 0;
        for (int x : nums) mx = max(mx, x);

        // 初始化并查集
        int root[n + mx + 1];
        for (int i = 0; i <= n + mx; i++) root[i] = i;

        // 查询并查集的根
        function<int(int)> findroot = [&](int x) {
            return root[x] == x ?  x : root[x] = findroot(root[x]);
        };

        // 对每个 nums[i]，向它们的质因数连边
        for (int i = 0; i < n; i++) for (int p : fac[nums[i]]) {
            int x = findroot(i), y = findroot(n + p);
            if (x == y) continue;
            root[x] = y;
        }

        // 检查是否所有位置点都在同一连通块内
        unordered_set<int> st;
        for (int i = 0; i < n; i++) st.insert(findroot(i));
        return st.size() == 1;
    }
};
```