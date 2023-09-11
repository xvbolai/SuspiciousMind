
给你一个下标从 **0** 开始的 **二进制** 字符串 `floor` ，它表示地板上砖块的颜色。
+  `floor[i] = '0'` 表示地板上第 `i` 块砖块的颜色是 **黑色** 。
+  `floor[i] = '1'` 表示地板上第 `i` 块砖块的颜色是 **白色** 。

同时给你 `numCarpets` 和 `carpetLen` 。你有 `numCarpets` 条 **黑色** 的地毯，每一条 **黑色** 的地毯长度都为 `carpetLen` 块砖块。请你使用这些地毯去覆盖砖块，使得未被覆盖的剩余 **白色** 砖块的数目 **最小** 。地毯相互之间可以覆盖。

请你返回没被覆盖的白色砖块的 **最少** 数目。

**示例 1：**

![[Pasted image 20230729211803.png|center]]
```
输入：floor = "10110101", numCarpets = 2, carpetLen = 2
输出：2
解释：
上图展示了剩余 2 块白色砖块的方案。
没有其他方案可以使未被覆盖的白色砖块少于 2 块。
```

**示例 2：**

![[Pasted image 20230729211901.png|center]]
```
输入：floor = "11111", numCarpets = 2, carpetLen = 3
输出：0
解释：
上图展示了所有白色砖块都被覆盖的一种方案。
注意，地毯相互之间可以覆盖。
```

#### 提示 1

思考方向？

看到题目给的数据范围，先想想能否用 DP 做出来。（DP 可以认为是一种更高级的暴力）

#### 提示 2

如何定义 DP 的状态？

一般来说，题目给了什么就用什么定义：地板长度和地毯个数。而地毯长度更适合去划分状态。

只用地板长度一个维度够吗？

不够，状态定义没有体现出所使用的地毯的个数。因此需要两个维度。

#### 提示 3

状态的值及其转移如何设计？

一般来说，题目求什么就定义什么：定义 $f[i][j]$ 表示用 $i$ 条地毯覆盖前 $j$ 块板砖时，没被覆盖的白色砖块的最少数目。

转移时可以考虑**是否使用**第 $i$ 条地毯，且其**末尾**覆盖第 $j$ 块板砖：(为什么是末尾？因为 $f[i][j]$ 的定义是**前** $j$ 块板砖，覆盖 $j$ 后面的板砖完全是在浪费地毯长度）

+ 不使用: $f[i][j]=f[i][j - 1]+floor[j]=='1'$
+ 使用: $f[i][j]=f[i-1][j-carpetLen]$
取二者最小值，即
$$
f[i][j]=min(f[i][j-1] + floor[j]=='1', f[i-1][j-carpetLen])
$$
注意 $i=0$ 的时候只能不使用，需要单独计算。

最后答案为 $f[numCarpets][floor.length−1]$。

```c++
class Solution {
public:
    int minimumWhiteTiles(string floor, int n, int carpetLen) {
        int m = floor.length();
        if(n * carpetLen >= m) return 0;
        vector<vector<int>> f(n + 1, vector<int>(m));
        f[0][0] = (floor[0] == '1');
        for(int i = 1; i < m; ++i) {
            f[0][i] = f[0][i - 1] + (floor[i] == '1');
        }
        for(int i = 1; i <= n; ++i) {
            for(int j = carpetLen * i; j < m; ++j) {
                f[i][j] = min(f[i][j - 1] + (floor[j] == '1'), f[i - 1][j - carpetLen]);
            }
        }
        return f[n][m - 1];
    }
};
```
