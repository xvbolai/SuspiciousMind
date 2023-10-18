
给你一个正整数数组 `nums` 。

如果数组 `nums` 的子集中的元素乘积是一个 **无平方因子数** ，则认为该子集是一个 **无平方** 子集。

**无平方因子数** 是无法被除 `1` 之外任何平方数整除的数字。

返回数组 `nums` 中 **无平方** 且 **非空** 的子集数目。因为答案可能很大，返回对 `109 + 7` 取余的结果。

`nums` 的 **非空子集** 是可以由删除 `nums` 中一些元素（可以不删除，但不能全部删除）得到的一个数组。如果构成两个子集时选择删除的下标不同，则认为这两个子集不同。

**示例 1：**

```txt
**输入：**nums = [3,4,4,5]
**输出：**3
**解释：**示例中有 3 个无平方子集：
- 由第 0 个元素 [3] 组成的子集。其元素的乘积是 3 ，这是一个无平方因子数。
- 由第 3 个元素 [5] 组成的子集。其元素的乘积是 5 ，这是一个无平方因子数。
- 由第 0 个和第 3 个元素 [3,5] 组成的子集。其元素的乘积是 15 ，这是一个无平方因子数。
可以证明给定数组中不存在超过 3 个无平方子集。
```

**示例 2：**

```txt
**输入：**nums = [1]
**输出：**1
**解释：**示例中有 1 个无平方子集：
- 由第 0 个元素 [1] 组成的子集。其元素的乘积是 1 ，这是一个无平方因子数。
可以证明给定数组中不存在超过 1 个无平方子集。
```

**提示：**

- `1 <= nums.length <= 1000`
- `1 <= nums[i] <= 30`

### 方法一：转换成 0-1 背包方案数
把无平方因子数的数字记作 SF（square-free number）。

对于每个 $[2,30]$ 内的 SF，通过预处理得到每个 SF 的质因子集合，用二进制表示。二进制从低到高第 $i$ 个比特为 1 表示第 $i$ 个质数在集合中，为 0 表示第 $i$ 个质数不在集合中。

那么把每个是 SF 的 $nums[i]$ 转换成对应的质因子集合，题目就变成「遍历所有由 $30$ 以内的质数组成的集合 $j$（这有 $2^{10}$个），对每个$j$，计算选一些不相交的质因子集合，它们的并集恰好为 $j$ 的方案数」。

### 方法二：子集状压 DP

如果把 $nums$ 的长度增加到 $10^5$ 的话，方法一就太慢了。

毕竟 $nums$ 的值域很小，把**相同数字一并处理**是更优的。

怎么转移呢？选择的数字乘积不能有平方因子，对应的质数集合也就不能有交集。那么枚举 $mask$ 的补集 $other$ 的子集 $j$，这样可以保证 $mask$ 和 $j$ 是没有交集的。站在并集 $j\cup mask$的角度，按照「选或不选」的思想，如果选了 $mask$，那么就要从 $f[j]$ 转移过来了。

具体地，设 $x$ 是 $mask$ 对应的 SF，$cnt[x]$ 为 $x$ 在 $nums$ 中的出现次数，那么需要从 $cnt[x]$ 个 $mask$ 中选一个，与 $j$ 并成 $j\cup mask$，对应的转移为
$$
f[j\cup mask] += f[j] * cnt[x]
$$

```c++
class Solution {
    static constexpr int NSQ[] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29};
    static constexpr int MOD = 1e9 + 7, MX = 30, N_P = 10, M = 1 << N_P;
public:
    int squareFreeSubsets(vector<int>& nums) {
        int sf2mask[MX + 1]{};
        for(int i = 2; i <= MX; ++i) {
            for(int j = 0; j < N_P; ++j) {
                int p = NSQ[j];
                if(i % p == 0) {
                    if(i % (p * p) == 0) {
                        sf2mask[i] = -1;
                        break;
                    }
                    sf2mask[i] |= 1 << j;
                }
            }
        }
        long f[M]{1};
        // 0-1背包
        // for(int x : nums) {
        //     if(int mask = sf2mask[x]; mask >= 0) {
        //         for(int j = M - 1; j >= mask; --j) {
        //             if((j | mask) == j) 
        //                 f[j] = (f[j] + f[j^mask]) % MOD;
        //         }
        //     }
        // }
        // return (accumulate(f, f + M, 0L) - 1) % MOD;
        // 子集状态DP
        int cnt[MX + 1]{}, pow2 = 1;
        for(int x : nums) {
            if(x == 1) pow2 = pow2 * 2 % MOD;
            else ++cnt[x];
        }
        for(int x = 2; x <= MX; ++x) {
            int mask = sf2mask[x], c = cnt[x];
            if(mask > 0 && c) {
                int other = (M - 1) ^ mask, j = other;
                do {
                    f[j | mask] = (f[j|mask] + f[j] * cnt[x]) % MOD;
                    j = (j - 1) & other;
                } while(j != other);
            }
        }
        return (accumulate(f, f + M, 0L) % MOD * pow2 - 1 + MOD) % MOD;
    }
};
```