#### [1439. 有序矩阵中的第 k 个最小数组和](https://leetcode.cn/problems/find-the-kth-smallest-sum-of-a-matrix-with-sorted-rows/)

给你一个 `m * n` 的矩阵 `mat`，以及一个整数 `k` ，矩阵中的每一行都以非递减的顺序排列。

你可以从每一行中选出 1 个元素形成一个数组。返回所有可能数组中的第 k 个 **最小** 数组和。

 **示例 1：**

```
输入：mat = [[1,3,11],[2,4,6]], k = 5
输出：7
解释：从每一行中选出一个元素，前 k 个和最小的数组分别是：
[1,2], [1,4], [3,2], [3,4], [1,6]。其中第 5 个的和是 7 。  
```

**示例 2：**

```
输入：mat = [[1,3,11],[2,4,6]], k = 9
输出：17
```

**示例 3：**

```
输入：mat = [[1,10,10],[1,4,5],[2,3,6]], k = 7
输出：9
解释：从每一行中选出一个元素，前 k 个和最小的数组分别是：
[1,1,2], [1,1,3], [1,4,2], [1,4,3], [1,1,6], [1,5,2], [1,5,3]。其中第 7 个的和是 9 。 
```

**示例 4：**

```
输入：mat = [[1,1,10],[2,2,9]], k = 7
输出：12
```

##### 算法一：暴力

```c++
class Solution {
public:
    int kthSmallest(vector<vector<int>>& mat, int k) {
        vector<int> num1 = {0};
        for(auto &row : mat) {
            vector<int> num2;
            for(int x : num1) {
                for(int y : row) {
                    num2.push_back(x + y);
                }
            }
            sort(num2.begin(), num2.end());
            if(num2.size() > k) num2.resize(k);
            num1 = std::move(num2);
        }
        return num1.back();
    }
};
```

##### 算法二：最小堆

**前置问题：**

>给定两个以 **升序排列** 的整数数组 `nums1` 和 `nums2` , 以及一个整数 `k` 。定义一对值 `(u,v)`，其中第一个元素来自 `nums1`，第二个元素来自 `nums2` 。请找到和最小的 `k` 个数对 `(u1,v1)`, ` (u2,v2)` ...  `(uk,vk)` 。

**思路一：**直接暴力统计数组`nums1和nums2`所有元素对值和，然后再快排。时间复杂度为`m*nlog(m*n)`，其中`m、n`分别为数组元素个数。

**思路二：**可以利用数组性质，由于两个数组是升序排序，`(nums1[0], nums2[0])`是最小的数对，计入答案。并且次小的只能是`(nums1[0], nums2[1])或(nums1[1], nums2[0])`，因为其它没有计入的答案的数对和不会比这两个更小。

`(nums1[0], nums2[1])或(nums1[1], nums2[0])`这两个数对的大小还好比较，但是如果要求第**k**小，就要涉及到更多的数对，那样就更加复杂了。如何按从小到大的顺序**快速地**求出这些数对呢？

为了更高效地比大小，我们可以借助**最小堆**来优化。

设堆顶中保存下标对`(i, j)`，即可能成为下一个数对的nums1和nums2的下标。堆顶是最小的`nums1[i] + nums2[j]`。

初始化把(0, 0)入堆。

每次出堆时，可能成为下一个数对的是`(i + 1, j)和(i, j + 1)`，这两入堆。

显然这会导致一个堆中出现重复元素，例如当`(1, 0)`出堆时，会把`(1, 1)`和`(2, 0)`入堆；当`(0, 1)`出堆时，也会把`(1, 1)`入堆。为避免重复入堆，可以用一个哈希表来标记哪些数值对在堆中。

**能否不用哈希表呢？**

换个角度，如果`(i, j)`入堆，那么**出堆**的下表对可能是什么？

根据上边讨论，出堆下标只能是`(i - 1, j)和(i, j - 1)`。因此我们只需要保证`(i - 1, j)和(i, j - 1)`的**其中一个**入堆即可。不妨假定`(i - 1, j)`出堆时，把`(i, j)`入堆，而对于`(i, j - 1)`则不入堆`(i, j)`.

换句话说，在`(i, j)`出堆时，把`(i + 1, j)`入堆，而`(i, j + 1)`则什么也不做。

但是按照这个思路，导致我们只能遍历到的下标只有`(0, 0)、(1, 0)、(2, 0)...`这些下标对。

所以初始化时不仅要把`(0, 0)`入堆，`(0, 1)、(0, 2)...`也要入堆。

这个地方代码实现时，为方便比较大小，实际入堆的是三元组`(sums1[i] + sum[j], i, j)`.

```c++
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        vector<vector<int>> ans;
        priority_queue<tuple<int, int, int>>  pq;
        int m = nums1.size(), n = nums2.size();
        for(int j = 0; j < n; ++j) pq.emplace(-nums1[0] - nums2[j], 0, j);
        while(!pq.empty() && ans.size() < k) {
            auto [_, i, j] = pq.top();
            pq.pop();
            ans.push_back({nums1[i], nums2[j]});
            if(i + 1 < m) pq.emplace(-nums1[i + 1] - nums2[j], i + 1, j);
        }
        return ans;
    }
};
```

也可以在循环的过程中去把 `(0, j)` 入堆。由于一开始堆的大小不大，出堆入堆更快，整体效率更高。

```c++
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        vector<vector<int>> ans;
        priority_queue<tuple<int, int, int>>  pq;
        int m = nums1.size(), n = nums2.size();
        // for(int j = 0; j < n; ++j) pq.emplace(-nums1[0] - nums2[j], 0, j);
        pq.emplace(-nums1[0] - nums2[0], 0, 0);
        while(!pq.empty() && ans.size() < k) {
            auto [_, i, j] = pq.top();
            pq.pop();
            ans.push_back({nums1[i], nums2[j]});
            if(i == 0 && j + 1 < n) pq.emplace(-nums1[0] - nums2[j + 1], 0, j + 1);
            if(i + 1 < m) pq.emplace(-nums1[i + 1] - nums2[j], i + 1, j);
        }
        return ans;
    }
};
```

##### 复杂度分析

+ 时间复杂度：**klog(min(m, k))**.
+ 空间复杂度: **min(n, k)**.

##### 相似题目（第 k 小/大）

- [373. 查找和最小的 K 对数字](https://leetcode.cn/problems/find-k-pairs-with-smallest-sums/)
- [378. 有序矩阵中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-sorted-matrix/)
- [719. 找出第 K 小的数对距离](https://leetcode.cn/problems/find-k-th-smallest-pair-distance/)
- [786. 第 K 个最小的素数分数](https://leetcode.cn/problems/k-th-smallest-prime-fraction/)
- [2040. 两个有序数组的第 K 小乘积](https://leetcode.cn/problems/kth-smallest-product-of-two-sorted-arrays/)
- [2386. 找出数组的第 K 大和](https://leetcode.cn/problems/find-the-k-sum-of-an-array/)

通过**前置问题**我们可以知道两个数组如何求解，对于一个矩阵求第**k**小的和，从第一、而行开始求出k个最小值，然后用这个k个最小值与第三行再求出k个最小和，以此类推，最后求出整个矩阵第**k**小的和。

```c++
class Solution {
public:

    vector<int> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        vector<int> ans;
        priority_queue<tuple<int, int, int>>  pq;
        int m = nums1.size(), n = nums2.size();
        // for(int j = 0; j < n; ++j) pq.emplace(-nums1[0] - nums2[j], 0, j);
        pq.emplace(-nums1[0] - nums2[0], 0, 0);
        while(!pq.empty() && ans.size() < k) {
            auto [_, i, j] = pq.top();
            pq.pop();
            ans.push_back(nums1[i]+nums2[j]);
            if(i == 0 && j + 1 < n) pq.emplace(-nums1[0] - nums2[j + 1], 0, j + 1);
            if(i + 1 < m) pq.emplace(-nums1[i + 1] - nums2[j], i + 1, j);
        }
        return ans;
    }

    int kthSmallest(vector<vector<int>>& mat, int k) {
        vector<int> num1 = {0};
        for(auto &row : mat) {
            // vector<int> num2;
            // for(int x : num1) {
            //     for(int y : row) {
            //         num2.push_back(x + y);
            //     }
            // }
            // sort(num2.begin(), num2.end());
            // if(num2.size() > k) num2.resize(k);
            // num1 = std::move(num2);
            num1 = kSmallestPairs(num1, row, k);  
        }
        return num1.back();
    }
};
```

##### 算法三：二分答案

设有`f(s)`个不超过`s`的数组和。由于随着`s`的增加，`f(s)`不会减小，有单调性，可以用二分。

+ 如果`f(s)>k`，说明答案至少为`s`。
+ 如果`f(s) < k`，说明答案至少为`s+1`。
+ 二分结束后，答案为`s`，那么必然有`f(s- 1) < k且f(s) >= k`。注意这和第k小是等价的。

如何计算`f(s)?`

直接用回溯来计算。统计不超过`s`的数组的个数，一旦个数超过`k`就停止回溯，不再继续递归。

```c++
class Solution {
    bool dfs(vector<vector<int>> &mat, int i, int s, int &k) {
        if(i < 0) return --k == 0;
        for(int x : mat[i]) {
            if(x - mat[i][0] > s) break;
            if(dfs(mat, i - 1, s - (x - mat[i][0]), k)) {
                return true;
            }
        }
        return false;
    }

public:
    int kthSmallest(vector<vector<int>> &mat, int k) {
        int left = 0, s1 = 0, right = 0, m = mat.size(), n = mat[0].size(), left_k;
        for(auto &v : mat) {
            left += v[0];
            right += v[n - 1];
        }
        s1 = left;
        while(left < right) {
            int mid = left + (right - left) / 2;
            left_k = k;
            if(dfs(mat, m - 1, mid - s1, left_k)) { // f(mid) >= k
                right = mid;
            } else left = mid + 1; // f(mid) < k
        }
        return left;
    }
};
```



##### 二分答案相似题（按照难度分排序）

- [875. 爱吃香蕉的珂珂](https://leetcode.cn/problems/koko-eating-bananas/)
- [1283. 使结果不超过阈值的最小除数](https://leetcode.cn/problems/find-the-smallest-divisor-given-a-threshold/)
- [2187. 完成旅途的最少时间](https://leetcode.cn/problems/minimum-time-to-complete-trips/)
- [2226. 每个小孩最多能分到多少糖果](https://leetcode.cn/problems/maximum-candies-allocated-to-k-children/)
- [1870. 准时到达的列车最小时速](https://leetcode.cn/problems/minimum-speed-to-arrive-on-time/)
- [1011. 在 D 天内送达包裹的能力](https://leetcode.cn/problems/capacity-to-ship-packages-within-d-days/)
- [2064. 分配给商店的最多商品的最小值](https://leetcode.cn/problems/minimized-maximum-of-products-distributed-to-any-store/)
- [1760. 袋子里最少数目的球](https://leetcode.cn/problems/minimum-limit-of-balls-in-a-bag/)
- [1482. 制作 m 束花所需的最少天数](https://leetcode.cn/problems/minimum-number-of-days-to-make-m-bouquets/)
- [1642. 可以到达的最远建筑](https://leetcode.cn/problems/furthest-building-you-can-reach/)
- [1898. 可移除字符的最大数目](https://leetcode.cn/problems/maximum-number-of-removable-characters/)
- [778. 水位上升的泳池中游泳](https://leetcode.cn/problems/swim-in-rising-water/)
- [2258. 逃离火灾](https://leetcode.cn/problems/escape-the-spreading-fire/)



#### [373. 查找和最小的 K 对数字](https://leetcode.cn/problems/find-k-pairs-with-smallest-sums/)

给定两个以 **升序排列** 的整数数组 `nums1` 和 `nums2` , 以及一个整数 `k` 。

定义一对值 `(u,v)`，其中第一个元素来自 `nums1`，第二个元素来自 `nums2` 。

请找到和最小的 `k` 个数对 `(u1,v1)`, ` (u2,v2)` ...  `(uk,vk)` 。

**示例 1:**

```
输入: nums1 = [1,7,11], nums2 = [2,4,6], k = 3
输出: [1,2],[1,4],[1,6]
解释: 返回序列中的前 3 对数：
     [1,2],[1,4],[1,6],[7,2],[7,4],[11,2],[7,6],[11,4],[11,6]
```

**示例 2:**

```
输入: nums1 = [1,1,2], nums2 = [1,2,3], k = 2
输出: [1,1],[1,1]
解释: 返回序列中的前 2 对数：
     [1,1],[1,1],[1,2],[2,1],[1,2],[2,2],[1,3],[1,3],[2,3]
```

**示例 3:**

```
输入: nums1 = [1,2], nums2 = [3], k = 3 
输出: [1,3],[2,3]
解释: 也可能序列中所有的数对都被返回:[1,3],[2,3]
```

```c++
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        vector<vector<int>> ans;
        priority_queue<tuple<int, int, int>>  pq;
        int m = nums1.size(), n = nums2.size();
        // for(int j = 0; j < n; ++j) pq.emplace(-nums1[0] - nums2[j], 0, j);
        pq.emplace(-nums1[0] - nums2[0], 0, 0);
        while(!pq.empty() && ans.size() < k) {
            auto [_, i, j] = pq.top();
            pq.pop();
            ans.push_back({nums1[i], nums2[j]});
            if(i == 0 && j + 1 < n) pq.emplace(-nums1[0] - nums2[j + 1], 0, j + 1);
            if(i + 1 < m) pq.emplace(-nums1[i + 1] - nums2[j], i + 1, j);
        }
        return ans;
    }
};
```

#### [378. 有序矩阵中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-sorted-matrix/)

给你一个 `n x n` 矩阵 `matrix` ，其中每行和每列元素均按升序排序，找到矩阵中第 `k` 小的元素。

请注意，它是 **排序后** 的第 `k` 小元素，而不是第 `k` 个 **不同** 的元素。

你必须找到一个内存复杂度优于 `O(n2)` 的解决方案。

**示例 1：**

```
输入：matrix = [[1,5,9],[10,11,13],[12,13,15]], k = 8
输出：13
解释：矩阵中的元素为 [1,5,9,10,11,12,13,13,15]，第 8 小元素是 13
```

**示例 2：**

```
输入：matrix = [[-5]], k = 1
输出：-5
```

**思路一：快排**

```c++
class Solution {
public:
    int kthSmallest(vector<vector<int>>& matrix, int k) {
        vector<int> t;
        for(auto &u : matrix) {
            for(auto &v : u) {
                t.push_back(v);
            }
        }
        sort(t.begin(), t.end());
        return t[k - 1];
    }
};
```

**复杂度分析**

+ 时间复杂度：`n^2log(n)`
+ 空间复杂度：`n^2`

**思路二：归并排序**

由题目的性质可知，这个矩阵的每一行均为有序数组。问题即转换为从这`n`个有序数组中找到第`k`大的数，可以利用归并排序做法，归并到第k个数即停止。

常规的归并排序是两个数组的归并，对于`n`归并利用一个小根堆维护，一优化时间复杂度。

```c++
class Solution {
public:
    int kthSmallest(vector<vector<int>>& matrix, int k) {
        int m = matrix.size(), n = matrix[0].size();
        priority_queue<tuple<int, int, int>> pq;
        for(int i = 0; i < m; ++i) pq.emplace(-matrix[i][0], i, 0);
        while(k > 1) {
            --k;
            auto [_, i, j] = pq.top();
            pq.pop();
            if(j + 1 < m) pq.emplace(-matrix[i][j + 1], i, j + 1);
        }
        auto [val, _, __] = pq.top();
        return -val;
    }
};
```

**复杂度分析**

+ 时间复杂度：`O(klog(n))`
+ 空间复杂度：`O(n)`

**思路三：二分**

由题目给出的性质可知，这个矩阵内的元素是从左上到右下递增的（假设矩阵左上角为 `matrix[0][0]`）

```c++
class Solution {
public:

    bool check(vector<vector<int>>& matrix, int mid, int k) {
        int ans = 0;
        int m = matrix.size(), n = matrix[0].size(), i = m - 1, j = 0;
        while(i >= 0 && j < n) {
            if(matrix[i][j] <= mid) {
                ans += 1 + i;
                ++j;
            } else --i;
        }
        return ans >= k;
    }
    int kthSmallest(vector<vector<int>>& matrix, int k) {
        int m = matrix.size(), n = matrix[0].size();
        int left = matrix[0][0], right = matrix[m - 1][n - 1];
        while(left < right) {
            int mid = left + (right - left) / 2;
            if(check(matrix, mid, k)) {
                right = mid; 
            } else left = mid + 1;
        }
        return left;
    }
};
```

**复杂度分析：**

+ 时间复杂度：`O(nlog(right - left))`.
+ 空间复杂度：`O(1)`

#### [719. 找出第 K 小的数对距离](https://leetcode.cn/problems/find-k-th-smallest-pair-distance/)

数对 `(a,b)` 由整数 `a` 和 `b` 组成，其数对距离定义为 `a` 和 `b` 的绝对差值。

给你一个整数数组 `nums` 和一个整数 `k` ，数对由 `nums[i]` 和 `nums[j]` 组成且满足 `0 <= i < j <nums.length` 。返回 **所有数对距离中** 第 `k` 小的数对距离。

**示例 1：**

```
输入：nums = [1,3,1], k = 1
输出：0
解释：数对和对应的距离如下：
(1,3) -> 2
(1,1) -> 0
(3,1) -> 2
距离第 1 小的数对是 (1,1) ，距离为 0。
```

**示例 2：**

```
输入：nums = [1,1,1], k = 2
输出：0
```

**示例 3：**

```
输入：nums = [1,6,1], k = 3
输出：5
```



找第k小数，立马可以想到可以二分，那么二分啥呢？**数对距离**。

```c++
class Solution {
public:

    bool check(vector<int>& nums, int mid, int k) {
        int ans = 0, left = 0;
        for(int right = 0; right < nums.size(); ++right) {
            while(nums[right] - nums[left] > mid) ++left;
            ans += (right - left);
        }
        return ans >= k;
    }
    int smallestDistancePair(vector<int>& nums, int k) {
        sort(nums.begin(), nums.end());
        int n = nums.size(), left = 0, right = nums[n - 1];
        while(left < right) {
            int mid = left + (right - left) / 2;
            if(check(nums, mid, k)) {
                right = mid;
            } else {
                left = mid + 1;
            }
        }
        return left;
    }
};
```

#### [2386. 找出数组的第 K 大和](https://leetcode.cn/problems/find-the-k-sum-of-an-array/)

给你一个整数数组 `nums` 和一个 **正** 整数 `k` 。你可以选择数组的任一 **子序列** 并且对其全部元素求和。

数组的 **第 k 大和** 定义为：可以获得的第 `k` 个 **最大** 子序列和（子序列和允许出现重复）

返回数组的 **第 k 大和** 。

子序列是一个可以由其他数组删除某些或不删除元素排生而来的数组，且派生过程不改变剩余元素的顺序。

**注意：空子序列的和视作 `0` 。** 

**示例 1：**

```c++
输入：nums = [2,4,-2], k = 5
输出：2
解释：所有可能获得的子序列和列出如下，按递减顺序排列：
- 6、4、4、2、2、0、0、-2
数组的第 5 大和是 2 。
```

**示例 2：**

```
输入：nums = [1,-2,3,4,-10,12], k = 16
输出：10
解释：数组的第 16 大和是 10 。
```

**思路一：二分结果**

+ 最大数必然是数组中所有正数之和`sum`，次小的必然是减去正数或者加上负数，因此可以统一变成减去绝对值。
+ 因此题目可以变成找出第`k-1`小的**子序列**绝对值和`ans`。
+ 结果便是`sum -ans`


```c++
class Solution {
public:
    bool check(vector<int>& nums, long mid, int k) {
        int ans = 0;
        function<void(int, long)> dfs = [&](int i, long s) {
            if(i == nums.size() || s + nums[i] > mid || ans >= k) return;
            ++ans;
            dfs(i + 1, s + nums[i]);
            dfs(i + 1, s);
        };
        dfs(0, 0L);
        return ans >= k;
    }

    long long kSum(vector<int>& nums, int k) {
        long sum = 0;
        for(int &x : nums) if(x >= 0) sum += x;
        else x = -x;
        sort(nums.begin(), nums.end());
        --k;
        long left = 0, right = accumulate(nums.begin(), nums.end(), 0L);
        while(left < right) {
            long mid = left + (right - left) / 2;
            if(check(nums, mid, k)) right = mid;
            else left = mid + 1;
        }
        return sum - left;
    }
};
```

**思路二：最小堆**

```c++
class Solution {
public:
    long long kSum(vector<int>& nums, int k) {
        long sum = 0;
        for(int &x : nums) if(x >= 0) sum += x;
        else x = -x;
        sort(nums.begin(), nums.end());
        priority_queue<pair<long, int>> pq;
        pq.emplace(sum, 0);
        while(--k) {
            auto [sum, i] = pq.top();
            pq.pop();
            if(i < nums.size()) {
                pq.emplace(sum - nums[i], i + 1);
                if(i) pq.emplace(sum + nums[i - 1] - nums[i], i + 1);
            }
        }
        return pq.top().first;
    }
};
```

#### [786. 第 K 个最小的素数分数](https://leetcode.cn/problems/k-th-smallest-prime-fraction/)

给你一个按递增顺序排序的数组 `arr` 和一个整数 `k` 。数组 `arr` 由 `1` 和若干 **素数** 组成，且其中所有整数互不相同。

对于每对满足 `0 <= i < j < arr.length` 的 `i` 和 `j` ，可以得到分数 `arr[i] / arr[j]` 。

那么第 `k` 个最小的分数是多少呢? 以长度为 `2` 的整数数组返回你的答案, 这里 `answer[0] ==arr[i]` 且 `answer[1] == arr[j]` 。

**示例 1：**

```
输入：arr = [1,2,3,5], k = 3
输出：[2,5]
解释：已构造好的分数,排序后如下所示: 
1/5, 1/3, 2/5, 1/2, 3/5, 2/3
很明显第三个最小的分数是 2/5
```

**示例 2：**

```
输入：arr = [1,7], k = 1
输出：[1,7]
```



```c++
class Solution {
public:
    vector<int> kthSmallestPrimeFraction(vector<int>& arr, int k) {
        int n = arr.size();
        double left = 0.0, right = 1.0;
        while(true) {
            int x = 0, y = 1;
            int i = 0, cnt = 0;
            double mid = (left + right) / 2;
            for(int j = 1; j < n; ++j) {
                while((double)arr[i] / arr[j] < mid) {
                    if(arr[i] * y > arr[j] * x) {
                        x = arr[i];
                        y = arr[j];
                    }     
                    ++i;        
                }
                cnt += i;
            }
            if(cnt == k) return {x, y};
            else if(cnt < k) left = mid;
            else right = mid;
        }
    }
};
```

#### [2040. 两个有序数组的第 K 小乘积](https://leetcode.cn/problems/kth-smallest-product-of-two-sorted-arrays/)

给你两个 **从小到大排好序** 且下标从 **0** 开始的整数数组 `nums1` 和 `nums2` 以及一个整数 `k` ，请你返回第 `k` （从 **1** 开始编号）小的 `nums1[i] * nums2[j]` 的乘积，其中 `0 <= i < nums1.length` 且 `0 <=j < nums2.length` 。

**示例 1：**

```
输入：nums1 = [2,5], nums2 = [3,4], k = 2
输出：8
解释：第 2 小的乘积计算如下：
- nums1[0] * nums2[0] = 2 * 3 = 6
- nums1[0] * nums2[1] = 2 * 4 = 8
第 2 小的乘积为 8 。
```

**示例 2：**

```
输入：nums1 = [-4,-2,0,3], nums2 = [2,4], k = 6
输出：0
解释：第 6 小的乘积计算如下：
- nums1[0] * nums2[1] = (-4) * 4 = -16
- nums1[0] * nums2[0] = (-4) * 2 = -8
- nums1[1] * nums2[1] = (-2) * 4 = -8
- nums1[1] * nums2[0] = (-2) * 2 = -4
- nums1[2] * nums2[0] = 0 * 2 = 0
- nums1[2] * nums2[1] = 0 * 4 = 0
第 6 小的乘积为 0 。
```

**示例 3：**

```
输入：nums1 = [-2,-1,0,1,2], nums2 = [-3,-1,2,4,5], k = 3
输出：-6
解释：第 3 小的乘积计算如下：
- nums1[0] * nums2[4] = (-2) * 5 = -10
- nums1[0] * nums2[3] = (-2) * 4 = -8
- nums1[4] * nums2[0] = 2 * (-3) = -6
第 3 小的乘积为 -6 。
```



```c++
class Solution {
public:
    long long kthSmallestProduct(vector<int>& nums1, vector<int>& nums2, long long k) {
        vector<int> pos1, pos2, neg1, neg2;
        for(int x : nums1) (x < 0) ? neg1.push_back(x) : pos1.push_back(x);
        for(int x : nums2) (x < 0) ? neg2.push_back(x) : pos2.push_back(x);
        long long left = -1e10, right = 1e10;
        while(left < right) {
            long long mid = (right + left) >> 1; // 采用移位运算，不然出现超时。
            long long  ans = 0;
            for(int i = 0, j = (int)pos2.size() - 1; i < pos1.size(); ++i) {
                while(j >= 0 && 1ll * pos1[i] * pos2[j] > mid) --j;
                ans += j + 1;
            }
            for(int i = 0, j = 0; i < neg1.size(); ++i) {
                while(j < pos2.size() && 1ll * neg1[i] * pos2[j] > mid) ++j;
                ans += (int)pos2.size() - j;
            }
            for(int i = 0, j = 0; i < pos1.size(); ++i) {
                while(j < neg2.size() && 1ll * pos1[i] * neg2[j] <= mid) ++j;
                ans += j;
            }
            for(int i = 0, j = neg2.size() - 1; i < neg1.size(); ++i) {
                while(j >= 0 && 1ll * neg1[i] * neg2[j] <= mid) --j;
                ans += (int)neg2.size() - 1 - j;
            }
            if(ans >= k) right = mid;
            else left = mid + 1;
        }
        return right;
    }
};
```



