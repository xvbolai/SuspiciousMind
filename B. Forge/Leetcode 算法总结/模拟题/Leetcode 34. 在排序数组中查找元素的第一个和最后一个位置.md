给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 `target`，返回 `[-1, -1]`。

你必须设计并实现时间复杂度为 `O(log n)` 的算法解决此问题。

**示例 1：**

```
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]
```

**示例 2：**

```
输入：nums = [5,7,7,8,8,10], target = 6
输出：[-1,-1]
```

**示例 3：**

```
输入：nums = [], target = 0
输出：[-1,-1]
```

**提示：**

- `0 <= nums.length <= 105`
- `-109 <= nums[i] <= 109`
- `nums` 是一个非递减数组
- `-109 <= target <= 109`



```c++
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        auto it_l = lower_bound(nums.begin(), nums.end(), target);
        if(it_l == nums.end() || *it_l != target) return {-1, -1};
        int r = upper_bound(nums.begin(), nums.end(), target) - nums.begin() - 1;
        int l = it_l - nums.begin();
        return {l, r};
    }
};
```



>**lower_bound函数用法**
>
>在C++中，lower_bound函数用于在`有序序列`中查找指定的元素，并返回`第一个大于或等于该元素`的迭代器。lower_bound函数采用两个迭代器参数，第一个参数表示序列的起始位置，第二个参数表示序列的结束位置。lower_bound函数还接受一个可选的第三个参数，表示要查找的元素的值。**该函数的第三个参数作为比较函数的第二个参数。**



>**upper_bound函数用法**
>
>在C++中，upper_bound函数用于在`有序序列`中查找指定的元素，并返回`第一个大于该元素`的迭代器。upper_bound函数和lower_bound函数非常相似，只是返回的迭代器指向的是第一个大于给定值的元素，而不是大于或等于该元素的迭代器。**该函数的第三个参数作为比较函数的第一个参数。**