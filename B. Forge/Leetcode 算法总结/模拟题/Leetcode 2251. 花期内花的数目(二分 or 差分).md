给你一个下标从 **0** 开始的二维整数数组 `flowers` ，其中 `flowers[i] = [starti, endi]` 表示第 `i` 朵花的 **花期** 从 `starti` 到 `endi` （都 **包含**）。同时给你一个下标从 **0** 开始大小为 `n` 的整数数组 `people` ，`people[i]` 是第 `i` 个人来看花的时间。

请你返回一个大小为 `n` 的整数数组 `answer` ，其中 `answer[i]`是第 `i` 个人到达时在花期内花的 **数目** 。

**示例 1：**

![[Pasted image 20230928225558.png]]

```txt
输入：flowers = [[1,6],[3,7],[9,12],[4,13]], people = [2,3,7,11]
输出：[1,2,2,2]
解释：上图展示了每朵花的花期时间，和每个人的到达时间。
对每个人，我们返回他们到达时在花期内花的数目。
```

**示例 2：**

![[Pasted image 20230928225636.png]]

```txt
输入：flowers = [[1,10],[3,3]], people = [3,3,2]
输出：[2,2,1]
解释：上图展示了每朵花的花期时间，和每个人的到达时间。
对每个人，我们返回他们到达时在花期内花的数目。
```

**提示：**

- $1 <= flowers.length <= 5 * 10^4$
- `flowers[i].length == 2`
- $1 <= starti <= endi <= 10^9$
- $1 <= people.length <= 5 * 10^4$
- $1 <= people[i] <= 10^9$ 

```c++
class Solution {
public:
    vector<int> fullBloomFlowers(vector<vector<int>>& flowers, vector<int>& people) {
        map<int, int> diff;
        for(auto &f : flowers) {
            ++diff[f[0]];
            --diff[f[1] + 1];
        }
        int n = people.size();
        vector<int> id(n);
        // 产生连续数字，从初始值开始，init, init + 1, init + 2 ...
        iota(id.begin(), id.end(), 0);
        sort(id.begin(), id.end(), [&](int i, int j) {
            return people[i] < people[j];
        });
        int sum = 0;
        auto it = diff.begin();
        for(int i : id) {
            while(it != diff.end() && it->first <= people[i]) {
                sum += it++->second;
            }
            people[i] = sum;
        }
        return people;
    }
};
```

### 二分
```c++
class Solution {
public:
    vector<int> fullBloomFlowers(vector<vector<int>>& flowers, vector<int>& people) {
        int n = flowers.size();
        vector<int> starts(n), ends(n);
        for(int i = 0; i < n; ++i) {
            starts[i] = flowers[i][0];
            ends[i] = flowers[i][1];
        }
        sort(starts.begin(), starts.end());
        sort(ends.begin(), ends.end());
        for(int &p : people) {
            p = (upper_bound(starts.begin(), starts.end(), p) - starts.begin()) - (lower_bound(ends.begin(), ends.end(), p) - ends.begin());
        }
        return people;
    }
};
```