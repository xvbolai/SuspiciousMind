
你总共需要上 `numCourses` 门课，课程编号依次为 `0` 到 `numCourses-1` 。你会得到一个数组 `prerequisite` ，其中 `prerequisites[i] = [ai, bi]` 表示如果你想选 `bi` 课程，你 **必须** 先选 `ai` 课程。

- 有的课会有直接的先修课程，比如如果想上课程 `1` ，你必须先上课程 `0` ，那么会以 `[0,1]` 数对的形式给出先修课程数对。
先决条件也可以是 **间接** 的。如果课程 `a` 是课程 `b` 的先决条件，课程 `b` 是课程 `c` 的先决条件，那么课程 `a` 就是课程 `c` 的先决条件。

你也得到一个数组 `queries` ，其中 `queries[j] = [uj, vj]`。对于第 `j` 个查询，您应该回答课程 `uj` 是否是课程 `vj` 的先决条件。

返回一个布尔数组 `answer` ，其中 `answer[j]` 是第 `j` 个查询的答案。

**示例 1：**

![[Pasted image 20230912105553.png]]

**输入：numCourses = 2, prerequisites = \[\[1,0\]\], queries = \[\[0,1\],\[1,0\]\]
输出：\[false,true\]
解释：课程 0 不是课程 1 的先修课程，但课程 1 是课程 0 的先修课程。

```c++
class Solution {
public:
    vector<bool> checkIfPrerequisite(int n, vector<vector<int>>& prerequisites, vector<vector<int>>& queries) {
        bitset<101> d[n];
        for(auto &v : prerequisites) {
            d[v[0]][v[1]] = 1;
        }
        for(int i = 0; i < n; ++i) {
            for(int j = 0; j < n; ++j) {
                if(d[j][i]) {
                    d[j] |= d[i];
                }
            }
        }
        vector<bool> ans;
        for(auto &v : queries) {
            ans.push_back(d[v[0]][v[1]]);
        }
        return ans;
    }
};
```
### 记忆化搜索

```c++
class Solution {
public:
    vector<bool> checkIfPrerequisite(int n, vector<vector<int>>& prerequisites, vector<vector<int>>& queries) {
        vector<vector<int>> e(n);
        for(auto &v : prerequisites) {
            e[v[0]].push_back(v[1]);
        }
        vector<vector<int>> mem(n, vector<int>(n, -1));
        function<bool(int, int)> dfs = [&](int u, int des) -> bool {
            auto &t = mem[u][des];
            if(t != -1) return t;
            if(u == des) return true;
            if(e[u].size() == 0) return false;
            for(auto v : e[u]) if(dfs(v, des)) {
                t = 1;
                return true;
            }
            t = 0;
            return false;
        };
        vector<bool> ans;
        for(auto &v : queries) {
            ans.push_back(dfs(v[0], v[1]));
        }
        return ans;
    }
};
```