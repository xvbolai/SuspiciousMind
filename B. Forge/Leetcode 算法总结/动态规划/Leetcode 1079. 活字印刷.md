![[image-20230519172140811.png]]

![[image-20230519172347024.png]]

![[image-20230519172432875.png]]

```c++
const int MX = 8;
int c[MX][MX];

int init = []() {
    for (int i = 0; i < MX; i++) {
        c[i][0] = c[i][i] = 1;
        for (int j = 1; j < i; j++)
            c[i][j] = c[i - 1][j - 1] + c[i - 1][j]; // 预处理组合数
    }
    return 0;
}();

class Solution {
public:
    int numTilePossibilities(string tiles) {
        unordered_map<char, int> counts;
        for (char c: tiles) counts[c]++; // 统计每个字母的出现次数
        int n = tiles.length(), m = counts.size(), f[m + 1][n + 1];
        memset(f, 0, sizeof(f));
        f[0][0] = 1; // 构造空序列的方案数
        int i = 1;
        for (auto &[_, cnt]: counts) { // 枚举第 i 种字母
            for (int j = 0; j <= n; j++) // 枚举序列长度 j
                for (int k = 0; k <= j && k <= cnt; k++) // 枚举第 i 种字母选了 k 个
                    f[i][j] += f[i - 1][j - k] * c[j][k];
            i++;
        }
        return accumulate(f[m] + 1, f[m] + n + 1, 0);
    }
};
```
![[image-20230519172831187.png]]


```c++
const int MAXN = 8;
int c[MAXN][MAXN];

auto init = []() {
    for(int i = 0; i < MAXN; ++i) {
        c[i][0] = c[i][i] = 1;
        for(int j = 1; j < i; ++j) {
            c[i][j] = c[i - 1][j - 1] + c[i - 1][j];
        }
    }
};

class Solution {
public:
    int numTilePossibilities(string tiles) {
        unordered_map<char, int> m;
        for(auto &cc : tiles)    ++m[cc];
        vector<int> dp(tiles.size() + 1, 0);
        dp[0] = 1;
        int n = 0;
        init();
        for(auto &[_, cnt] : m) {
            n += cnt;
            for(int j = n; j; --j) {
                for(int k = 1; k <= j && k <= cnt; ++k) {
                    dp[j] += dp[j - k] * c[j][k];
                }
            }
        }
        return accumulate(dp.begin(), dp.end(), 0) - 1;
    }
};
```

