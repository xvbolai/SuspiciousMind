#### [1015. 可被 K 整除的最小整数](https://leetcode.cn/problems/smallest-integer-divisible-by-k/)

给定正整数 `k` ，你需要找出可以被 `k` 整除的、仅包含数字 **1** 的最 **小** 正整数 `n` 的长度。

返回 `n` 的长度。如果不存在这样的 `n` ，就返回-1。

**注意：** `n` 可能不符合 64 位带符号整数。

**示例 1：**

```
输入：k = 1
输出：1
解释：最小的答案是 n = 1，其长度为 1。
```

**示例 2：**

```
输入：k = 2
输出：-1
解释：不存在可被 2 整除的正整数 n 。
```

**示例 3：**

```
输入：k = 3
输出：3
解释：最小的答案是 n = 111，其长度为 3。
```

![[image-20230528171921027.png]]

![[image-20230528171959411.png]]

![[1683680206-OGDZGf-1015-3.png]]

### 算法一（无优化）

```python
class Solution:
    def smallestRepunitDivByK(self, k: int) -> int:
        seen = set()
        x = 1 % k
        while x and x not in seen:
            seen.add(x)
            x = (x * 10 + 1) % k
        return -1 if x else len(seen) + 1
```

### 算法二+优化

```python
class Solution:
    def smallestRepunitDivByK(self, k: int) -> int:
        if k % 2 == 0 or k % 5 == 0:
            return -1
        x = 1 % k
        for i in count(1):  # 一定有解
            if x == 0:
                return i
            x = (x * 10 + 1) % k
```

### 算法三

附：[欧拉定理]([欧拉定理 & 费马小定理 - OI Wiki (oi-wiki.org)](https://oi-wiki.org/math/number-theory/fermat/#欧拉定理))

```python
# 计算欧拉函数（n 以内的与 n 互质的数的个数）
def phi(n: int) -> int:
    res = n
    i = 2
    while i * i <= n:
        if n % i == 0:
            res = res // i * (i - 1)
            while n % i == 0:
                n //= i
        i += 1
    if n > 1:
        res = res // n * (n - 1)
    return res

class Solution:
    def smallestRepunitDivByK(self, k: int) -> int:
        if k % 2 == 0 or k % 5 == 0:
            return -1
        m = phi(k * 9)
        # 从小到大枚举不超过 sqrt(m) 的因子
        i = 1
        while i * i <= m:
            if m % i == 0 and pow(10, i, k * 9) == 1:
                return i
            i += 1
        # 从小到大枚举不低于 sqrt(m) 的因子
        i -= 1
        while True:
            if m % i == 0 and pow(10, m // i, k * 9) == 1:
                return m // i
            i -= 1
```

```c++
class Solution {
    // 计算欧拉函数（n 以内的与 n 互质的数的个数）
    int phi(int n) {
        int res = n;
        for (int i = 2; i * i <= n; i++) {
            if (n % i == 0) {
                res = res / i * (i - 1);
                while (n % i == 0) n /= i;
            }
        }
        if (n > 1)
            res = res / n * (n - 1);
        return res;
    }

    // 快速幂，返回 pow(x, n) % mod
    long long pow(long long x, int n, long long mod) {
        long long res = 1;
        for (; n; n /= 2) {
            if (n % 2) res = res * x % mod;
            x = x * x % mod;
        }
        return res;
    }

public:
    int smallestRepunitDivByK(int k) {
        if (k % 2 == 0 || k % 5 == 0)
            return -1;
        int m = phi(k * 9);
        // 从小到大枚举不超过 sqrt(m) 的因子
        int i = 1;
        for (; i * i <= m; i++)
            if (m % i == 0 && pow(10, i, k * 9) == 1)
                return i;
        // 从小到大枚举不低于 sqrt(m) 的因子
        for (i--; ; i--)
            if (m % i == 0 && pow(10, m / i, k * 9) == 1)
                return m / i;
    }
};
```

