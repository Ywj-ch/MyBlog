---
title: DFS变通 —— 路径检查到环检测
date: 2026-04-27 10:00:00
tags: [DFS, BFS, 图论]
categories: 算法学习
cover: /img/leetcode.jpg
---

# 算法笔记：DFS 模板的变通——从路径检查到环检测

> **日期**: 2026-04-27  
> **主题**: DFS、图论、模板变通  
> **难度**: Medium  
> **标签**: DFS、BFS、有向图、无向图找环、表驱动

---

## 一、写在前面

最近连续做了两道 DFS 题，一道是检查网格中的有效路径（1391），一道是探测相同值形成的环（1559）。两道题都用到了 DFS，但和之前做过的"岛屿问题"、"填海造陆"那种标准遍历模板不太一样——不是拿着 `void dfs(x, y)` 直接搜就行，而是要根据题意在基础模板上做不少变通。

做完之后我发现，自己正在经历一个从"背模板"到"理解原理然后变通"的阶段。这种感受很难得，趁热记录下来。

---

## 二、LeetCode 1391. 检查网格中是否存在有效路径

> **题目链接**: https://leetcode.cn/problems/check-if-there-is-a-valid-path-in-a-grid/  
> **难度**: Medium  
> **标签**: DFS、表驱动、BFS

### 题目描述

给定一个 `m × n` 的网格，每个格子代表一条街道，有 6 种类型：

- `1`：左 ↔ 右
- `2`：上 ↔ 下
- `3`：左 ↔ 下
- `4`：右 ↔ 下
- `5`：左 ↔ 上
- `6`：右 ↔ 上

从 `(0, 0)` 出发，只能沿着街道的方向移动，问能否走到 `(m-1, n-1)`。

### 核心难点：每种格子的移动方向不同

普通的 DFS 是四个方向随便走，但这里每种格子只有两个方向能走，而且还需要和相邻格子的方向"对接"——你能走过去，邻居得能接住你。

第一反应很自然是想写六个 `if` 去判断，但那样代码会又臭又长。

### 解法：用方向映射表替代 if

这里学到了一个很重要的技巧：**用数据表代替控制流**。定义四方向的编号：

```
0: 上, 1: 下, 2: 左, 3: 右
```

然后每种街道类型直接存它能走的方向编号：

```cpp
// 下标 0 弃用，1~6 对应六种街道
vector<vector<int>> possibleDirs = {
    {0, 0},       // 占位
    {2, 3},       // 1: 左, 右
    {0, 1},       // 2: 上, 下
    {2, 1},       // 3: 左, 下
    {3, 1},       // 4: 右, 下
    {0, 2},       // 5: 上, 左
    {0, 3}        // 6: 上, 右
};
```

方向偏移量也用一个数组：

```cpp
vector<vector<int>> dirs = {{-1,0}, {1,0}, {0,-1}, {0,1}};
```

反方向数组——用来检查邻居能否接住你：

```cpp
vector<int> opposite = {1, 0, 3, 2}; // 0对1, 2对3
```

这样 DFS 的核心逻辑就变成：

```cpp
bool dfs(grid, r, c, visit) {
    visit[r][c] = true;
    for (int d : possibleDirs[grid[r][c]]) {      // 当前格子能走的方向
        int nr = r + dirs[d][0], nc = c + dirs[d][1];
        if (越界 || 已访问) continue;
        int opp = opposite[d];                    // 邻居需要从这个方向接住
        if (邻居的 possibleDirs 包含 opp) {        // 能对接
            if (是终点) return true;
            if (dfs(nr, nc)) return true;
        }
    }
    return false;
}
```

### 踩坑记录

**坑一：终点判断的次序**

我一开始这样写的：

```cpp
if (nr == m-1 && nc == n-1) return true;  // 先判断终点
int opp = opposite[d];
if (contains(...)) { ... }                 // 再检查连通性
```

这会导致即使终点格子不连通也错误返回 `true`。正确做法是**先检查连通性，再判断终点**：

```cpp
int opp = opposite[d];
if (contains(possibleDirs[grid[nr][nc]], opp)) {
    if (nr == m-1 && nc == n-1) return true;  // 连通且是终点
    if (dfs(nr, nc, visit)) return true;
}
```

**坑二：手写 `contains` 的性能陷阱**

我写了一个辅助函数来判断邻居方向表中是否包含 `opp`：

```cpp
bool contains(vector<int> nextD, int opp) {  // 按值传递！
    for (int next : nextD) { ... }
}
```

注意参数是**按值传递的**，每次调用都会拷贝整个 `vector<int>`。在百万级调用下，这就是灾难。

后来用 `std::ranges::find` 替代（C++20）：

```cpp
if (std::ranges::find(possibleDirs[grid[nr][nc]], opp)
    != possibleDirs[grid[nr][nc]].end())
```

运行时间从 1590ms 直接降到 24ms。这里其实最大的因素不是标准库的优化，而是**消除了 vector 拷贝**。

### 完整代码

```cpp
class Solution {
public:
    vector<vector<int>> possibleDirs = {{0,0},{2,3},{0,1},{2,1},{3,1},{0,2},{0,3}};
    vector<vector<int>> dirs = {{-1,0},{1,0},{0,-1},{0,1}};
    vector<int> opposite = {1, 0, 3, 2};

    bool dfs(vector<vector<int>>& grid, int r, int c, vector<vector<bool>>& visit) {
        int m = grid.size(), n = grid[0].size();
        visit[r][c] = true;
        for (int d : possibleDirs[grid[r][c]]) {
            int nr = r + dirs[d][0], nc = c + dirs[d][1];
            if (nr < 0 || nr >= m || nc < 0 || nc >= n || visit[nr][nc]) continue;
            int opp = opposite[d];
            if (std::ranges::find(possibleDirs[grid[nr][nc]], opp)
                != possibleDirs[grid[nr][nc]].end()) {
                if (nr == m - 1 && nc == n - 1) return true;
                if (dfs(grid, nr, nc, visit)) return true;
            }
        }
        return false;
    }

    bool hasValidPath(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();
        if (m == 1 && n == 1) return true;
        vector<vector<bool>> visit(m, vector<bool>(n, false));
        return dfs(grid, 0, 0, visit);
    }
};
```

### 延伸：硬编码写法为什么更快？

力扣上有种写法把每种格子类型和每个来向都硬编码成 `if-else`，代码又长又丑，但跑得飞快。

这是因为硬编码没有循环、没有查表、没有间接内存访问——CPU 分支预测几乎百发百中。而表驱动写法的 `for (int d : possibleDirs[...])` 和 `std::ranges::find` 对 CPU 来说是不确定的，分支预测容易失败，指令流水线停顿。

但这只能在力扣上刷榜用。工程中表驱动才是正道——加一种新类型只需改一行数据，而不是改几十处 `if`。

---

## 三、LeetCode 1559. 二维网格图中探测环

> **题目链接**: https://leetcode.cn/problems/detect-cycles-in-2d-grid/  
> **难度**: Medium  
> **标签**: DFS、环检测、无向图

### 题目描述

给定一个二维字符网格，判断是否存在**相同值**形成的环。环要求长度 ≥ 4，且不能走回头路（`A → B → A` 不算环）。

### 核心难点：怎么判断环？

直觉是 DFS，但和"遍历所有格子"的 DFS 不同，这里需要判断"是否存在环"——一种"短路搜索"：一旦找到就立刻停止，层层返回。

### 解法：传递父节点，碰到已访问的非父节点就是环

关键技巧是**在 DFS 参数中携带父节点坐标** `(pr, pc)`：

```cpp
bool dfs(grid, visit, r, c, pr, pc) {
    visit[r][c] = true;
    for (四个方向) {
        nr = r + dr, nc = c + dc;
        if (越界) continue;
        if (值不同) continue;
        if (nr == pr && nc == pc) continue;  // 不走回头路
        if (visit[nr][nc]) return true;       // 遇到已访问的非父节点 → 环！
        if (dfs(grid, visit, nr, nc, r, c)) return true;
    }
    return false;
}
```

**为什么不需要额外的路径记录数组？**

无向图 DFS 中，`visit` 数组已经记录了"所有访问过的节点"。当你遇到一个已访问节点时：

- 如果它是父节点 → 这是回头路，跳过
- 如果它不是父节点 → 说明存在另一条路径到达了这个节点 → 形成环

`(pr, pc)` 参数本身就提供了"当前路径"的上下文，不需要像有向图那样额外维护 `inStack` 数组。

**为什么环长度自动 ≥ 4？**

网格是二分图，不存在奇数环（三角形）。能被检测到的非父节点已访问邻居，必然构成至少 4 条边的四边形环。两个格子的来回被父节点检查排除，三个格子无法在网格中形成三角。

### 完整代码

```cpp
class Solution {
public:
    vector<vector<int>> dir = {{-1,0},{1,0},{0,-1},{0,1}};

    bool dfs(vector<vector<char>>& grid, vector<vector<bool>>& visit,
             int r, int c, int pr, int pc) {
        int m = grid.size(), n = grid[0].size();
        visit[r][c] = true;
        for (int i = 0; i < 4; i++) {
            int nr = r + dir[i][0], nc = c + dir[i][1];
            if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
            if (grid[nr][nc] != grid[r][c]) continue;
            if (nr == pr && nc == pc) continue;    // 不走回头路
            if (visit[nr][nc]) return true;         // 找到环
            if (dfs(grid, visit, nr, nc, r, c)) return true;
        }
        return false;
    }

    bool containsCycle(vector<vector<char>>& grid) {
        int m = grid.size(), n = grid[0].size();
        vector<vector<bool>> visit(m, vector<bool>(n, false));
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (!visit[r][c]) {
                    if (dfs(grid, visit, r, c, -1, -1)) return true;
                }
            }
        }
        return false;
    }
};
```

### DFS vs BFS 实测对比

这道题我也写了 BFS 版本，实测数据很有意思：

| 指标 | DFS | BFS |
|------|-----|-----|
| 运行时间 | 24ms | 170ms |
| 内存占用 | 63MB | 203MB |

DFS 全面优于 BFS，原因：

- **DFS 沿一条路径深入**，环往往在较深处被发现，一找到就能立即短路返回；BFS 层层扩展，在发现环之前可能已经遍历了大量无关节点。
- **BFS 队列开销大**：`queue<tuple<int,int,int,int>>` 中每个元素 4 个 int，大量节点堆积。
- **缓存友好性**：DFS 递归沿路径深入，内存访问连续；BFS 在队列中跳跃访问。

结论：**找环问题用 DFS，最短路问题用 BFS**。

---

## 四、两种 DFS 变种的对比

| 维度 | 1391 有效路径 | 1559 探测环 |
|------|-------------|-----------|
| 搜索目标 | 是否存在一条到达终点的路径 | 是否存在环 |
| 返回值 | `bool`，短路返回 | `bool`，短路返回 |
| 额外参数 | 无（方向由表格控制） | `(pr, pc)` 父节点 |
| 判断条件 | 邻居必须"接住"当前方向 | 遇到已访问非父节点即环 |
| 核心技巧 | 方向映射表 + opposite 数组 | 父节点传递替代路径记录 |

---

## 五、DFS 模板的三种形态

做完这两道题后，我意识到自己之前接触的 DFS 其实有三种不同的形态：

```
┌─────────────────────────────────────────────────┐
│  形态一：遍历型 DFS（void）                       
│  特点：走完所有节点，产生副作用（标记、计数）          
│  例题：岛屿数量、填海造陆                          
│  模板：void dfs(x, y) { visit[x][y]=true; 走邻居; }
└─────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────┐
│  形态二：搜索型 DFS（bool）                       
│  特点：找到一个目标就立即终止，层层短路返回         
│  例题：1391 检查有效路径、1559 探测环              
│  模板：bool dfs(x, y) { if 找到目标 return true; }
└─────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────┐
│  形态三：回溯型 DFS（void + 状态恢复）             
│  特点：枚举所有可能性，每层需要恢复现场            
│  例题：全排列、N 皇后、解数独                      
│  模板：做选择 → dfs() → 撤销选择                  
└─────────────────────────────────────────────────┘
```

从 `void` 到 `bool`，从"走遍"到"找到"，这个转变看起来小，但实际上是 DFS 用途的一次跃迁。之前做的连通块、岛屿类问题都是形态一，惯性让我觉得 DFS 就是返回 `void`。而通过这两道题让我意识到：**DFS 的返回值不是固定的，取决于你用它干什么**。

打个比方：
- 遍历型是"打扫房间"——每个角落都要擦一遍，不用中途喊停
- 搜索型是"找钥匙"——一旦找到立刻举手说"找到了"，然后停止一切搜索
- 回溯型是"试密码"——试一个不行就擦掉重试，直到把正确密码试出来

---

## 六、心得总结

### 表驱动 vs 硬编码

- 表驱动：用数据描述规则，加新规则只改数据不改逻辑。代码短、可维护、利索。
- 硬编码：把所有情况写成 `if-else`，快但不可扩展。力扣刷榜可用，工程中别学。

### 无向图找环的通用框架

```
bool dfs(当前节点, 父节点):
    标记当前已访问
    for 每个邻居:
        if 邻居 == 父节点: continue   // 不走回头路
        if 邻居已访问: return true    // 后向边 = 环
        if dfs(邻居, 当前节点): return true
    return false
```

这个框架适用于大部分无向图环检测题，三个条件缺一不可。

### 什么时候用 BFS？

- 问"最短"、"最少" → BFS
- 问"是否存在"、"找环"、"连通性" → DFS
- 数据量大且深度可能很深 → BFS 或迭代 DFS（防爆栈）

---

## 七、相关题目

- [LeetCode 200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/) - 遍历型 DFS 模板
- [LeetCode 79. 单词搜索](https://leetcode.cn/problems/word-search/) - 回溯型 DFS
- [LeetCode 785. 判断二分图](https://leetcode.cn/problems/is-graph-bipartite/) - DFS 染色
- [LeetCode 802. 找到最终的安全状态](https://leetcode.cn/problems/find-eventual-safe-states/) - 有向图找环
