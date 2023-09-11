给定 *n* 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1 。

求在该柱状图中，能够勾勒出来的矩形的最大面积。

**示例 1:**

![[histogram.jpg]]

```
输入：heights = [2,1,5,6,2,3]
输出：10
解释：最大的矩形为图中红色区域，面积为 10
```

**示例 2：**

![[histogram-1.jpg|100x200]]

```
输入： heights = [2,4]
输出： 4
```

#### 方法一：单调栈

**思路**

我们可以遍历每根柱子，以当前柱子 i 的高度作为矩形的高，那么矩形的宽度边界即为向左找到第一个高度小于当前柱体 i 的柱体，向右找到第一个高度小于当前柱体 i 的柱体。

对于每个柱子我们都如上计算一遍以当前柱子作为高的矩形面积，最终比较出最大的矩形面积即可。

```c++
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int n = heights.size();
        vector<int> left(n), right(n, n);
        stack<int> s;
        for(int i = 0; i < n; ++i) {
            while(!s.empty() && heights[s.top()] >= heights[i]) {
                right[s.top()] = i;
                s.pop();
            }
            left[i] = (s.empty()) ? -1 : s.top();
            s.push(i);
        }
        int res = 0;
        for(int i = 0; i < n; ++i) {
            res = max(res, (right[i] - left[i] - 1) * heights[i]);
        }
        return res;
    }
};
```

