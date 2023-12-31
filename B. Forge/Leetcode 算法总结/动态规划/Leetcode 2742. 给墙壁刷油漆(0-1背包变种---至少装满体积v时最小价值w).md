#hard 
给你两个长度为 `n` 下标从 **0** 开始的整数数组 `cost` 和 `time` ，分别表示给 `n` 堵不同的墙刷油漆需要的开销和时间。你有两名油漆匠：

- 一位需要 **付费** 的油漆匠，刷第 `i` 堵墙需要花费 `time[i]` 单位的时间，开销为 `cost[i]` 单位的钱。
- 一位 **免费** 的油漆匠，刷 **任意** 一堵墙的时间为 `1` 单位，开销为 `0` 。但是必须在付费油漆匠 **工作** 时，免费油漆匠才会工作。

请你返回刷完 `n` 堵墙最少开销为多少。

**示例 1：**

```txt
**输入：**cost = [1,2,3,2], time = [1,2,3,2]
**输出：**3
**解释：**下标为 0 和 1 的墙由付费油漆匠来刷，需要 3 单位时间。同时，免费油漆匠刷下标为 2 和 3 的墙，需要 2 单位时间，开销为 0 。总开销为 1 + 2 = 3 。
```

**示例 2：**

```txt
**输入：**cost = [2,3,4,2], time = [1,1,1,1]
**输出：**4
**解释：**下标为 0 和 3 的墙由付费油漆匠来刷，需要 2 单位时间。同时，免费油漆匠刷下标为 1 和 2 的墙，需要 2 单位时间，开销为 0 。总开销为 2 + 2 = 4 。
```

**提示：**

- `1 <= cost.length <= 500`
- `cost.length == time.length`
- $1 <= cost[i] <= 10^6$
- `1 <= time[i] <= 500`

### 0-1背包转换

#### 思路

根据题意，付费刷墙个数 + 免费刷墙个数 = n.
同时，付费刷墙时间之和 $\geq$ 免费刷墙个数。

结合两个式子，得到：付费刷墙时间之和$\geq$n-付费刷墙个数。
即，$\lceil 付费刷墙时间 + 1 \rfloor$之和$\geq$n。

把$time[i] + 1$看成物品体积，$cost[i]$看成物品价值，问题变成：

从$n$个物品中选择体积和**至少**为$n$的物品，价值和最小是多少?

这是0-1背包的一种$\lceil 至少装满 \rfloor$的变形。我们可以定义$dfs(i, j)$表示考虑前$i$个物品，**剩余**还需要凑出$j$的体积，此时的最小价值和。
$$
f[i][j] = minf[i-1][j - time[i] - 1] + cost[i], f[i-1][j])
$$
边界条件：如果 $j \leq 0$，那么不需要再选任何物品了， 返回 0；否则，如果 $i < 0$ 则返回无穷大。

```c++
class Solution {
public:
    int paintWalls(vector<int>& cost, vector<int>& time) {
        int n = cost.size(), f[n + 1];
        memset(f, 0x3f, sizeof(f));
        f[0] = 0;
        for(int  i = 0; i < n; ++i) {
            int c = cost[i], t =  time[i] + 1;
            for(int j = n; j; --j) {
                f[j] = min(f[j], f[max(j - t, 0)] + c);
            }
        }
        return f[n];
    }
};
```

