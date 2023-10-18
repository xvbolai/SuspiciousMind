#### [2354. 优质数对的数目](https://leetcode.cn/problems/number-of-excellent-pairs/)

#困难 

给你一个下标从 **0** 开始的正整数数组 `nums` 和一个正整数 `k` 。
如果满足下述条件，则数对 `(num1, num2)` 是 **优质数对** ：
- `num1` 和 `num2` **都** 在数组 `nums` 中存在。
- `num1 OR num2` 和 `num1 AND num2` 的二进制表示中值为 **1** 的位数之和大于等于 `k` ，其中 `OR` 是按位 **或** 操作，而 `AND` 是按位 **与** 操作。
返回 **不同** 优质数对的数目。
如果 `a != c` 或者 `b != d` ，则认为 `(a, b)` 和 `(c, d)` 是不同的两个数对。例如，`(1, 2)` 和 `(2, 1)` 不同。注意：**如果 `num1` 在数组中至少出现 一次** ，则满足 `num1 == num2` 的数对 `(num1, num2)` 也可以是优质数对。

**Tips**

a | b 和 a & b 1的个数实则等于a 的1的个数和b的1的个数之和。

根据容斥原理，可以看成一个数组。
$$ \left| A \cup B \right | = \left | A\right | + \left| B \right | - \left | A \cap B \right |  $$


```c++
class Solution {
public:
    long long countExcellentPairs(vector<int>& nums, int k) {
        unordered_map<int, int> cnt;
        for(int x : unordered_set<int>(nums.begin(), nums.end())) {
            ++cnt[__builtin_popcount(x)];
        }
        long ans = 0L;
        for(auto &[cbx, cx] : cnt) {
            for(auto &[cby, cy] : cnt) {
                if(cbx + cby >= k) {
                    ans += (long) cx * cy;
                }
            }
        }
        return ans;
    }
};
```