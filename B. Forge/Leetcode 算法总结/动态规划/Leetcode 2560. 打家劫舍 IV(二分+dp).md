#### [2560. 打家劫舍 IV](https://leetcode.cn/problems/house-robber-iv/)

沿街有一排连续的房屋。每间房屋内都藏有一定的现金。现在有一位小偷计划从这些房屋中窃取现金。

由于相邻的房屋装有相互连通的防盗系统，所以小偷 **不会窃取相邻的房屋** 。

小偷的 **窃取能力** 定义为他在窃取过程中能从单间房屋中窃取的 **最大金额** 。

给你一个整数数组 `nums` 表示每间房屋存放的现金金额。形式上，从左起第 `i` 间房屋中放有 `nums[i]` 美元。

另给你一个整数 `k` ，表示窃贼将会窃取的 **最少** 房屋数。小偷总能窃取至少 `k` 间房屋。

返回小偷的 **最小** 窃取能力。

**示例 1：**

```
输入：nums = [2,3,5,9], k = 2
输出：5
解释：
小偷窃取至少 2 间房屋，共有 3 种方式：
- 窃取下标 0 和 2 处的房屋，窃取能力为 max(nums[0], nums[2]) = 5 。
- 窃取下标 0 和 3 处的房屋，窃取能力为 max(nums[0], nums[3]) = 9 。
- 窃取下标 1 和 3 处的房屋，窃取能力为 max(nums[1], nums[3]) = 9 。
因此，返回 min(5, 9, 9) = 5 。
```

**示例 2：**

```
输入：nums = [2,7,9,3,1], k = 2
输出：2
解释：共有 7 种窃取方式。窃取能力最小的情况所对应的方式是窃取下标 0 和 4 处的房屋。返回 max(nums[0], nums[4]) = 2 。
```

看到「最大化最小值」或者「最小化最大值」就要想到**二分答案**，这是一个固定的套路。

为什么？一般来说，二分的值越大，越能/不能满足要求；二分的值越小，越不能/能满足要求，有单调性，可以二分。

类似的题目在先前的周赛中出现过多次，例如：

- [2439. 最小化数组中的最大值](https://leetcode.cn/problems/minimize-maximum-of-array/)
- [2513. 最小化两个数组中的最大值](https://leetcode.cn/problems/minimize-the-maximum-of-two-arrays/)
- [2517. 礼盒的最大甜蜜度](https://leetcode.cn/problems/maximum-tastiness-of-candy-basket/)
- [2528. 最大化城市的最小供电站数目](https://leetcode.cn/problems/maximize-the-minimum-powered-city/)

设二分的最大金额为 *mx*，定义 **f[i]** 表示在前 *i* 个房屋中窃取金额不超过 *mx* 的房屋的最大个

分类讨论：

- 不选第 *i* 个房屋：$f[i]=f*[i−1]$；
- 选第 *i* 个房屋，前提是金额不超过 $mx：f[i]=f[i−2]+1$。

```c++
class Solution {
public:
    int minCapability(vector<int>& nums, int k) {
        int left = *min_element(nums.begin(), nums.end()) - 1, right = *max_element(nums.begin(), nums.end());
        while(left < right) {
            int f0 = 0, f1 = 0, mid = left + (right - left) / 2;
            for(int x : nums) {
                if(x > mid) f0 = f1;
                else {
                    int t = f1;
                    f1 = max(f1, f0 + 1);
                    f0 = t;
                }
            }
            // (f1 >= k ? right : left) = mid;
            if(f1 >= k) right = mid;
            else left = mid + 1;
        }
        return right;
    }
};
```

#### [2439. 最小化数组中的最大值](https://leetcode.cn/problems/minimize-maximum-of-array/)

```c++
class Solution {
public:
    bool check(vector<int> &nums, int k) {
        int  n = nums.size();
        long long mid = 0;
        for(int i = n - 1; i > 0; --i) {
            mid = (nums[i] + mid > k ? nums[i] + mid - k : 0);
        }
        return nums[0] + mid <= k;
    }
    int minimizeArrayValue(vector<int>& nums) {
        int left = 0, right = *max_element(nums.begin(), nums.end());
        while(left < right) {
            int mid = left + (right - left) / 2;
            if(check(nums, mid)) {
                right = mid;
            } else {
                left = mid + 1;
            }
        }
        return right;
    }
};
```

#### [2513. 最小化两个数组中的最大值](https://leetcode.cn/problems/minimize-the-maximum-of-two-arrays/)

给你两个数组 `arr1` 和 `arr2` ，它们一开始都是空的。你需要往它们中添加正整数，使它们满足以下条件：

- `arr1` 包含 `uniqueCnt1` 个 **互不相同** 的正整数，每个整数都 **不能** 被 `divisor1` **整除** 。
- `arr2` 包含 `uniqueCnt2` 个 **互不相同** 的正整数，每个整数都 **不能** 被 `divisor2` **整除** 。
- `arr1` 和 `arr2` 中的元素 **互不相同** 。

给你 `divisor1` ，`divisor2` ，`uniqueCnt1` 和 `uniqueCnt2` ，请你返回两个数组中 **最大元素** 的 **最小值** 。

![[image-20230528135617696.png]]

```c++
class Solution {
public:
    int minimizeSet(int divisor1, int divisor2, int uniqueCnt1, int uniqueCnt2) {
        int i = 0;
        long n = uniqueCnt1 + uniqueCnt2;
        long l = 0, r = n * 2 - 1;
        long lcm = std::lcm((long)divisor1,(long)divisor2);
        while(l < r){
            long x = l + (r - l) / 2;
            int val1 = max(uniqueCnt1 - (int)(x / divisor2 - x / lcm),0);
            int val2 = max(uniqueCnt2 - (int)(x / divisor1 - x / lcm),0);
            int common = x - x / divisor1 - x / divisor2 + x / lcm;
            if(common >= val1 + val2){
                r = x;
            }else{
                l = x + 1;
            }
        } 
        return l;
    }
};
```

##### 鸽巢原理

##### [878. 第 N 个神奇数字](https://leetcode.cn/problems/nth-magical-number/)

一个正整数如果能被 `a` 或 `b` 整除，那么它是神奇的。

给定三个整数 `n` , `a` , `b` ，返回第 `n` 个神奇的数字。因为答案可能很大，所以返回答案 **对** `109 + 7` **取模** 后的值。

**示例 1：**

```
输入：n = 1, a = 2, b = 3
输出：2
```

**示例 2：**

```
输入：n = 4, a = 2, b = 3
输出：6
```

![[1669032532-GjXsyF-878-2.png]]

```c++
class Solution {
public:
    int nthMagicalNumber(int n, int a, int b) {
        long lcm = std::lcm(a, b), left = 0, right = static_cast<long>(min(a, b)) * n;
        while(left < right) {
            long mid = left + (right - left) / 2;
            if(mid / a + mid / b - mid / lcm >= n) right = mid;
            else left = mid + 1;
        }
        return left % long(1e9 + 7);
    }
};
```

#### [1201. 丑数 III](https://leetcode.cn/problems/ugly-number-iii/)

给你四个整数：`n` 、`a` 、`b` 、`c` ，请你设计一个算法来找出第 `n` 个丑数。

丑数是可以被 `a` **或** `b` **或** `c` 整除的 **正整数** 。

**示例 1：**

```
输入：n = 3, a = 2, b = 3, c = 5
输出：4
解释：丑数序列为 2, 3, 4, 5, 6, 8, 9, 10... 其中第 3 个是 4。
```

**示例 2：**

```
输入：n = 4, a = 2, b = 3, c = 4
输出：6
解释：丑数序列为 2, 3, 4, 6, 8, 9, 10, 12... 其中第 4 个是 6。
```

**示例 3：**

```
输入：n = 5, a = 2, b = 11, c = 13
输出：10
解释：丑数序列为 2, 4, 6, 8, 10, 11, 12, 13... 其中第 5 个是 10。
```

**示例 4：**

```
输入：n = 1000000000, a = 2, b = 217983653, c = 336916467
输出：1999999984
```

```c++
class Solution {
public:
    int nthUglyNumber(int n, int a, int b, int c) {
        long lcm_ab = std::lcm((long)a, (long)b), lcm_bc = std::lcm((long)b, (long)c), lcm_ac = std::lcm((long)a, (long)c), lcm_abc = std::lcm(lcm_ab, lcm_ac);
        long left = 1, right = (long)n * min(a, min(b, c));
        while(left < right) {
            long mid = left + (right - left)  / 2;
            if(mid / a + mid / b + mid / c - mid / lcm_ab - mid / lcm_ac - mid / lcm_bc + mid / lcm_abc >= n) {
                right = mid;
            } else left = mid + 1;
        }
        return right;
    }
};
```

#### [2517. 礼盒的最大甜蜜度](https://leetcode.cn/problems/maximum-tastiness-of-candy-basket/)

给你一个正整数数组 `price` ，其中 `price[i]` 表示第 `i` 类糖果的价格，另给你一个正整数 `k` 。

商店组合 `k` 类 **不同** 糖果打包成礼盒出售。礼盒的 **甜蜜度** 是礼盒中任意两种糖果 **价格** 绝对差的最小值。

返回礼盒的 **最大** 甜蜜度。

**示例 1：**

```
输入：price = [13,5,1,8,21,2], k = 3
输出：8
解释：选出价格分别为 [13,5,21] 的三类糖果。
礼盒的甜蜜度为 min(|13 - 5|, |13 - 21|, |5 - 21|) = min(8, 8, 16) = 8 。
可以证明能够取得的最大甜蜜度就是 8 。
```

**示例 2：**

```
输入：price = [1,3,1], k = 2
输出：2
解释：选出价格分别为 [1,3] 的两类糖果。 
礼盒的甜蜜度为 min(|1 - 3|) = min(2) = 2 。
可以证明能够取得的最大甜蜜度就是 2 。
```

**示例 3：**

```
输入：price = [7,7,7,7], k = 2
输出：0
解释：从现有的糖果中任选两类糖果，甜蜜度都会是 0 。
```

```c++
class Solution {
public:

    bool check(vector<int>& price, int mid, int k) {
        int cnt = 1, pre = price[0];
        for(int i = 1; i < price.size(); ++i) {
            if(price[i] >= pre + mid) {
                pre = price[i];
                ++cnt;
            }
        }
        return cnt >= k;
    }
    int maximumTastiness(vector<int>& price, int k) {
        sort(price.begin(), price.end());
        int n = price.size(), left = 0, right = (price[n - 1] - price[0]) / (k - 1) + 1;
        while(left < right) {
            int mid = left + (right - left + 1) / 2;
            if(check(price, mid, k)) {
                left = mid;
            } else {
                right = mid - 1;
            }
        }
        return left;
    }
};
```

#### [2528. 最大化城市的最小供电站数目](https://leetcode.cn/problems/maximize-the-minimum-powered-city/)

给你一个下标从 **0** 开始长度为 `n` 的整数数组 `stations` ，其中 `stations[i]` 表示第 `i` 座城市的供电站数目。

每个供电站可以在一定 **范围** 内给所有城市提供电力。换句话说，如果给定的范围是 `r` ，在城市 `i` 处的供电站可以给所有满足 `|i - j| <= r` 且 `0 <= i, j <= n - 1` 的城市 `j` 供电。

- `|x|` 表示 `x` 的 **绝对值** 。比方说，`|7 - 5| = 2` ，`|3 - 10| = 7` 。

一座城市的 **电量** 是所有能给它供电的供电站数目。

政府批准了可以额外建造 `k` 座供电站，你需要决定这些供电站分别应该建在哪里，这些供电站与已经存在的供电站有相同的供电范围。

给你两个整数 `r` 和 `k` ，如果以最优策略建造额外的发电站，返回所有城市中，最小供电站数目的最大值是多少。

这 `k` 座供电站可以建在多个城市。

**示例 1：**

```
输入：stations = [1,2,4,5,0], r = 1, k = 2
输出：5
解释：
最优方案之一是把 2 座供电站都建在城市 1 。
每座城市的供电站数目分别为 [1,4,4,5,0] 。
- 城市 0 的供电站数目为 1 + 4 = 5 。
- 城市 1 的供电站数目为 1 + 4 + 4 = 9 。
- 城市 2 的供电站数目为 4 + 4 + 5 = 13 。
- 城市 3 的供电站数目为 5 + 4 = 9 。
- 城市 4 的供电站数目为 5 + 0 = 5 。
供电站数目最少是 5 。
无法得到更优解，所以我们返回 5 。
```

**示例 2：**

```
输入：stations = [4,4,4,4], r = 0, k = 3
输出：4
解释：
无论如何安排，总有一座城市的供电站数目是 4 ，所以最优解是 4 。
```



```c++
class Solution {
public:

    long long maxPower(vector<int>& stations, int r, int k) {
        int n = stations.size();
        long sum[n + 1], power[n], diff[n];
        sum[0] = 0;
        for(int i = 0; i < n; ++i) {
            sum[i + 1] = sum[i] + stations[i];
        }
        for(int i = 0; i < n; ++i) {
            power[i] = sum[min(i + r + 1, n)] - sum[max(0, i - r)];
        }
        auto check = [&](long min_power) -> bool {
            memset(diff, 0, sizeof(diff));
            long sum_d = 0, need = 0;
            for(int i = 0; i < n; ++i) {
                sum_d += diff[i];
                long m = min_power - power[i] -sum_d;
                if(m > 0) {
                    need += m;
                    if(need > k) return false;
                    sum_d += m;
                    if(i + r * 2 + 1 < n) diff[i + r * 2 + 1] = -m;
                }
            }
            return true;
        };
        long left = *min_element(power, power + n), right = left + k;
        while(left < right) {
            long mid = left + (right - left + 1)/ 2;
            if(check(mid)) {
                left = mid;
            } else {
                right = mid - 1;
            }
        }
        return left;
    }
};
```

