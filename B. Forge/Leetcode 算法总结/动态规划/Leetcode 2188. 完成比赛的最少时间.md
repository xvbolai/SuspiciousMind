给你一个下标从 **0** 开始的二维整数数组 `tires` ，其中 `tires[i] = [fi, ri]` 表示第 `i` 种轮胎如果连续使用，第 `x` 圈需要耗时 $f_{i} * r_i^{(x-1)}$ 秒。
+ 比方说，如果 `fi = 3` 且 `ri = 2` ，且一直使用这种类型的同一条轮胎，那么该轮胎完成第 `1` 圈赛道耗时 `3` 秒，完成第 `2` 圈耗时 `3 * 2 = 6` 秒，完- 完成第 `3` 圈耗时 `3 * 22 = 12` 秒，依次类推。

同时给你一个整数 `changeTime` 和一个整数 `numLaps` 。

比赛总共包含 `numLaps` 圈，你可以选择 **任意** 一种轮胎开始比赛。每一种轮胎都有**无数条** 。每一圈后，你可以选择耗费 `changeTime` 秒 **换成** 任意一种轮胎（也可以换成当前种类的新轮胎）。

请你返回完成比赛需要耗费的 **最少** 时间。

**示例 1：**

```txt
输入：tires = [[2,3],[3,4]], changeTime = 5, numLaps = 4
输出：21
解释：
第 1 圈：使用轮胎 0 ，耗时 2 秒。
第 2 圈：继续使用轮胎 0 ，耗时 2 * 3 = 6 秒。
第 3 圈：耗费 5 秒换一条新的轮胎 0 ，然后耗时 2 秒完成这一圈。
第 4 圈：继续使用轮胎 0 ，耗时 2 * 3 = 6 秒。
总耗时 = 2 + 6 + 5 + 2 + 6 = 21 秒。
完成比赛的最少时间为 21 秒。
```

**示例 2：**

```txt
输入：tires = [[1,10],[2,2],[3,4]], changeTime = 6, numLaps = 5
输出：25
解释：
第 1 圈：使用轮胎 1 ，耗时 2 秒。
第 2 圈：继续使用轮胎 1 ，耗时 2 * 2 = 4 秒。
第 3 圈：耗时 6 秒换一条新的轮胎 1 ，然后耗时 2 秒完成这一圈。
第 4 圈：继续使用轮胎 1 ，耗时 2 * 2 = 4 秒。
第 5 圈：耗时 6 秒换成轮胎 0 ，然后耗时 1 秒完成这一圈。
总耗时 = 2 + 4 + 6 + 2 + 4 + 6 + 1 = 25 秒。
完成比赛的最少时间为 25 秒。
```

提示：

+ $1 <= tires.length <= 10^5$
+ $tires[i].length == 2$
+ $1 <= f_i, changeTime <= 10_5$
+ $2 <= r_i <= 10^5$
+ $1 <= numLaps <= 1000$

#### 上界分析

连续使用同一个轮胎 $i$ 跑 $x$ 圈，第 $x$ 圈的耗时不应超过$changeTime + f_i$ ，即
$$
f_{i}*r_{i}^{x-1} \leq changeTime + f_{i}
$$
考虑 $x$ 至多能是多少。由于 $f_i$​ 越小 $x$ 的上界越大，以及 $r_i$​ 越小 $x$ 的上界越大，那么$f_i​=1,r_i​=2$，则有
$$
2^{x-1} \leq changeTime + 1
$$
解得
$$
x \leq \log(changeTime + 1) + 1
$$
由于 $x$ 是个整数，因此 $x$ 的上界为$x=\lfloor \log(changeTime + 1)  + 1\rfloor$
根据题目的数据范围，代码实现时可将上界视为 17。

#### 算法

首先预处理出连续使用同一个轮胎跑 $x$ 圈的最小耗时，记作 $minSec[x]$，这可以通过遍历每个轮胎计算出来。

然后定义$f[i]$表示跑$i$圈的最小耗时。为方便计算，初始值$f[0]=-changeTime$。
考虑最后一个轮胎连续跑了$j$圈，我们可以从$f[i - j]$转移过来，因此有转移方程:
$$
f[i] = changeTime + \min_{j = 1}^{17, i}f[i - j] + minSet[j]
$$
最后答案为$f[numLaps]$。

```c++
class Solution {
public:
    int minimumFinishTime(vector<vector<int>>& tires, int changeTime, int numLaps) {
        vector<int> minSet(18, INT_MAX / 2);
        for(auto &tire : tires) {
            long time = tire[0];
            for(int x = 1, sum = 0; time <= changeTime + tire[0]; ++x) {
                sum += time;
                minSet[x] = min(minSet[x], sum);
                time *= tire[1];
            }
        }
        vector<int> dp(numLaps + 1, INT_MAX);
        dp[0] = -changeTime;
        for(int i = 1; i <= numLaps; ++i) {
            for(int j = 1; j <= min(17, i); ++j) {
                dp[i] = min(dp[i - j] + minSet[j], dp[i]);
            }
            dp[i] += changeTime;
        }
        return dp[numLaps];
    }
};
```