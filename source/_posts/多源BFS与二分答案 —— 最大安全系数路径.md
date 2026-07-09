---
title: 多源BFS与二分答案 —— 最大安全系数路径
date: 2026-07-01 10:00:00
tags: [多源BFS, 二分答案, 连通性]
categories: 算法学习
cover: /img/leetcode.jpg
---

# 多源BFS + 二分答案 —— 最大安全系数路径

> **日期**: 2026-07-1
> **难度**: Medium
> **标签**: 多源BFS、二分答案、连通性
> **题目链接**: [2812. 找出最安全路径](https://leetcode.cn/problems/find-the-safest-path-in-a-grid/)

---

## 题目简述

给定一个 n×n 的 01 矩阵，1 表示小偷，0 表示空。路径的**安全系数**定义为路径上所有点到任一小偷的最小曼哈顿距离。求从 (0,0) 到 (n-1,n-1) 的所有路径中，安全系数的最大值。

## 第一次思路

读完题首先想到两个方向：

1. 需要算每个格子的安全距离
2. 然后要找一个最大安全系数。

但是有三个地方卡住了：

**多源BFS —— 多个小偷怎么一次算完？**
第一反应是对每个小偷单独 BFS，然后再对每个格子取 min。这显然太慢——小偷数量可能很多，每次 BFS 都是 O(n²)，总复杂度不行。正确做法应该是是**多源 BFS**：把所有小偷一次性全部入队，BFS 同时从所有源点向外扩散，第一个到达某格子的就是最近的小偷。这和「铺瓷砖」一个原理，同时从所有源头铺，每个格子被铺到的时间就是最短距离。

**"消除"不安全的格子 —— 要新建数组吗？**
提示说「消除所有满足 `d[x][y] < v` 的单元格」，我第一反应是每次二分都复制一份 grid 并把不安全的格子涂成障碍。这显然浪费内存。实际上根本不需要修改任何数组——check BFS 扩展邻居时直接判断 `d[nr][nc] >= v`，不满足就不入队，等效于"消除"。

**路径可以向上向左走吗？**
题目说可以移动到「任一相邻单元格」，所以是四方向，必须允许上下左右——为了绕开不安全区域，可能需要向上或向左迂回。

## 最终方案

两阶段：

1. **多源 BFS** 计算 `d[x][y]`：把所有小偷位置入队，距离 0，一次 BFS 得到每个格子到最近小偷的距离。
2. **二分答案 + check BFS**：二分安全系数 v，check 函数从 (0,0) 四方向 BFS，只走 `d >= v` 的格子，能到 (n-1,n-1) 则返回 true。

### 关键细节

- 多源 BFS 用 `queue`，出队时 `auto [r, c] = q.front(); q.pop()` —— **不能用引用**，pop 会导致悬挂指针
- check BFS 必须加 `!visited[nr][nc]` 判断，否则已访问格子重复入队

## 完整代码

```cpp
class Solution {
public:
    int maximumSafenessFactor(vector<vector<int>>& grid) {
        constexpr int dirs[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
        int n = grid.size();

        // 多源 BFS 计算每个格子到最近小偷的距离
        vector<vector<int>> d(n, vector<int>(n, -1));
        queue<pair<int, int>> q;
        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                if (grid[i][j] == 1) {
                    d[i][j] = 0;
                    q.push({i, j});
                }
        while (!q.empty()) {
            auto [r, c] = q.front(); q.pop();
            for (auto [dr, dc] : dirs) {
                int nr = r + dr, nc = c + dc;
                if (nr >= 0 && nr < n && nc >= 0 && nc < n && d[nr][nc] == -1) {
                    d[nr][nc] = d[r][c] + 1;
                    q.push({nr, nc});
                }
            }
        }

        // check(v): 是否存在一条路径，路径上所有点的 d >= v
        auto canReach = [&](int v) -> bool {
            if (d[0][0] < v) return false;
            vector visited(n, vector<bool>(n, false));
            queue<pair<int, int>> q;
            q.push({0, 0});
            visited[0][0] = true;
            while (!q.empty()) {
                auto [r, c] = q.front(); q.pop();
                for (auto [dr, dc] : dirs) {
                    int nr = r + dr, nc = c + dc;
                    if (nr >= 0 && nr < n && nc >= 0 && nc < n
                        && !visited[nr][nc] && d[nr][nc] >= v) {
                        if (nr == n - 1 && nc == n - 1) return true;
                        visited[nr][nc] = true;
                        q.push({nr, nc});
                    }
                }
            }
            return false;
        };

        // 二分查找最大安全系数
        int lv = 0, rv = 2 * n;
        while (lv < rv) {
            int v = (lv + rv + 1) / 2;
            if (canReach(v)) lv = v;
            else rv = v - 1;
        }
        return lv;
    }
};
```

## 复杂度分析

| 复杂度 | 分析 |
|--------|------|
| 时间复杂度 | O(n² log n)。多源 BFS O(n²)；二分 O(log n) 次，每次 check BFS O(n²) |
| 空间复杂度 | O(n²)，距离数组 d 和 check 中的 visited 数组 |

## 心得总结

- 多源 BFS 本质是「同时铺瓷砖」，所有源点一起入队 = 取每个格子的最小值，不需要手动 min
- "消除"不安全的格子不等于真的删掉数据——在搜索条件里加一个判断即可
- C++ 中 `auto& [r, c] = q.front(); q.pop()` 是经典坑：pop 销毁元素后引用悬空，必须改成拷贝 `auto [r, c]`
- 二分答案的右边界要想清楚：本题曼哈顿距离最大接近 2n，但是经过我的测试写n好像也没问题，可能是测试数据的问题。
