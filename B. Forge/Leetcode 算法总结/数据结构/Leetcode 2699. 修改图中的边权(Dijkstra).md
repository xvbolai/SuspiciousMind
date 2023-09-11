#### Leetcode 2699. 修改图中的边权(Dijkstra)
#数据结构 #dijkstra 

给你一个 `n` 个节点的 **无向带权连通** 图，节点编号为 `0` 到 `n - 1` ，再给你一个整数数组 `edges` ，其中 `edges[i] = [ai, bi, wi]` 表示节点 `ai` 和 `bi` 之间有一条边权为 `wi` 的边。
部分边的边权为 `-1`（`wi = -1`），其他边的边权都为 **正** 数（`wi > 0`）。
你需要将所有边权为 `-1` 的边都修改为范围 `[1, 2 * 109]` 中的 **正整数** ，使得从节点 `source` 到节点 `destination` 的 **最短距离** 为整数 `target` 。如果有 **多种** 修改方案可以使 `source` 和 `destination` 之间的最短距离等于 `target` ，你可以返回任意一种方案。
如果存在使 `source` 到 `destination` 最短距离为 `target` 的方案，请你按任意顺序返回包含所有边的数组（包括未修改边权的边）。如果不存在这样的方案，请你返回一个 **空数组** 。
**注意：你不能修改一开始边权为正数的边。
示例 1：
![[Pasted image 20230530211835.png|200]]
```
输入：n = 5, edges = [[4,1,-1],[2,0,-1],[0,3,-1],[4,3,-1]], source = 0, destination = 1, target = 5
输出：[[4,1,1],[2,0,1],[0,3,3],[4,3,1]]
解释：上图展示了一个满足题意的修改方案，从 0 到 1 的最短距离为 5 。
```
示例 2：**

![[Pasted image 20230530211922.png|200]]
```txt
输入：n = 3, edges = [[0,1,-1],[0,2,5]], source = 0, destination = 2, target = 6
输出：[]
解释：上图是一开始的图。没有办法通过修改边权为 -1 的边，使得 0 到 2 的最短距离等于 6 ，所以返回一个空数组。
```



```c++
class Solution {
public:
    const int INF = 0x3f3f3f3f;
    int dijkstra(int n, const vector<vector<int>>& edges, int source, int dest) {
        int dis[n];
        fill(dis, dis + n, INF);
        vector<pair<int, int>> E[n];
        for(auto &e : edges) {
            if(e[2] == -1) continue;
            E[e[0]].emplace_back(e[1], e[2]);
            E[e[1]].emplace_back(e[0], e[2]);
        } 
        dis[source] = 0;
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
        pq.emplace(0, source);
        while(!pq.empty()) {
            auto [d, u] = pq.top();
            pq.pop();
            if(dis[u] != d) continue;
            if(u == dest) break;
            for(auto &[v, d] : E[u]) {
                if(dis[u] + d < dis[v]) {
                    dis[v] = dis[u] + d;
                    pq.emplace(dis[v], v);
                }
            }
        }
        return dis[dest];
    }

    vector<vector<int>> modifiedGraphEdges(int n, vector<vector<int>>& edges, int source, int destination, int target) {
        int d = dijkstra(n, edges, source, destination);
        if(d < target) return {};
        bool found = false;
        if(d == target) found = true;
        for(auto &e : edges) {
            if(e[2] != -1) continue;
            if(found) {
                e[2] = INF;
                continue;
            }
            e[2] = 1;
            d = dijkstra(n, edges, source, destination);
            if(d <= target) {
                found = true;
                e[2] += target - d;
            }
        }
        return found ? edges : vector<vector<int>>();
    }
};
```



```c++
class Solution {
public:
    vector<vector<int>> modifiedGraphEdges(int n, vector<vector<int>> &edges, int source, int destination, int target) {
        vector<pair<int, int>> g[n];
        for (int i = 0; i < edges.size(); i++) {
            int x = edges[i][0], y = edges[i][1];
            g[x].emplace_back(y, i);
            g[y].emplace_back(x, i); // 建图，额外记录边的编号
        }

        int dis[n][2], delta, vis[n];
        memset(dis, 0x3f, sizeof(dis));
        dis[source][0] = dis[source][1] = 0;
        auto dijkstra = [&](int k) { // 这里 k 表示第一次/第二次
            memset(vis, 0, sizeof(vis));
            for (;;) {
                // 找到当前最短路，去更新它的邻居的最短路
                // 根据数学归纳法，dis[x][k] 一定是最短路长度
                int x = -1;
                for (int i = 0; i < n; ++i)
                    if (!vis[i] && (x < 0 || dis[i][k] < dis[x][k]))
                        x = i;
                if (x == destination) // 起点 source 到终点 destination 的最短路已确定
                    return;
                vis[x] = true; // 标记，在后续的循环中无需反复更新 x 到其余点的最短路长度
                for (auto [y, eid]: g[x]) {
                    int wt = edges[eid][2];
                    if (wt == -1)
                        wt = 1; // -1 改成 1
                    if (k == 1 && edges[eid][2] == -1) {
                        // 第二次 Dijkstra，改成 w
                        int w = delta + dis[y][0] - dis[x][1];
                        if (w > wt)
                            edges[eid][2] = wt = w; // 直接在 edges 上修改
                    }
                    // 更新最短路
                    dis[y][k] = min(dis[y][k], dis[x][k] + wt);
                }
            }
        };

        dijkstra(0);
        delta = target - dis[destination][0];
        if (delta < 0) // -1 全改为 1 时，最短路比 target 还大
            return {};

        dijkstra(1);
        if (dis[destination][1] < target) // 最短路无法再变大，无法达到 target
            return {};

        for (auto &e: edges)
            if (e[2] == -1) // 剩余没修改的边全部改成 1
                e[2] = 1;
        return edges;
    }
};