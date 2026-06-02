---
title: 图上DP —— 矩阵最大非负积
date: 2026-03-23 10:00:00
tags: [动态规划, 状态机思维]
categories: 算法学习
cover: /img/leetcode.jpg
---

# LeetCode 1807. 矩阵的最大非负积

> **题目链接**: https://leetcode.cn/problems/maximum-non-negative-product-in-a-matrix/  
> **日期**: 2026-03-23  
> **难度**: Medium  
> **标签**: 动态规划、状态机思维

---

## 一、题目描述

给定一个 `m × n` 的矩阵 grid，从左上角 `(0, 0)` 出发，只能向右或向下移动，最终到达右下角 `(m-1, n-1)`。

沿路径访问的单元格中所有整数的乘积即为该路径的**积**。

求所有路径中，**最大非负积**是多少？如果最大积为负数，返回 `-1`。

**注意**：最终答案要对 `10^9 + 7` 取模。

---

## 二、踩坑回顾

### 第一次尝试：DFS 暴力枚举

看到 `m, n ≤ 15` 不大，直觉上直接遍历所有路径应该可行。于是写了这样的 DFS：

```cpp
void dfs(vector<vector<int>>& grid, int r, int c, long long num) {
    num *= grid[r][c];
    if (r + c == grid.size() + grid[0].size() - 2) {
        nums.push_back(num);
        return;
    }
    if (c < grid[0].size() - 1) dfs(grid, r, c + 1, num);
    if (r < grid.size() - 1) dfs(grid, r + 1, c, num);
}
```

自信满满，结果超时。

**复盘**：路径数 = `C(m+n-2, m-1)`，当 `m = n = 15` 时，`C(28, 14) ≈ 4 × 10^8`，铁定超时。

### 第二次尝试：看了提示后想到双 DP

意识到累乘类问题不能简单用贪心，必须 DP。但矩阵里有正数也有负数，普通的 DP 只维护一个最大值不够用——遇到负数，最大值可能变成最小值，最小值可能变成最大值。

所以需要**同时维护两个 DP 表**：

- `max_dp[i][j]`：从 `(0,0)` 到 `(i,j)` 的**最大**路径积
- `min_dp[i][j]`：从 `(0,0)` 到 `(i,j)` 的**最小**路径积

### 第三次尝试：初始化踩坑

一开始把 DP 表全初始化为 1，想当然地类比累加问题：

```cpp
vector<vector<long long>> max_dp(m, vector<long long>(n, 1));
vector<vector<long long>> min_dp(m, vector<long long>(n, 1));
```

**错了**。累乘的初始化必须是 `grid[0][0]`，因为第一条路只能从起点出发，不能"空着"乘以 1。

改正后通过。

---

## 三、解题思路

### 核心洞察

**为什么需要两个 DP？**

考虑一个简单例子：`-1, -2`

- `max(-1, -2) × -3 = -2 × -3 = 6`（最大值）
- `min(-1, -2) × -3 = -1 × -3 = 3`（最小值）

负数的存在让"最大"和"最小"必须同时追踪。

### 状态转移方程

设当前单元格值为 `grid[r][c]`：

**当 `grid[r][c] >= 0` 时**：

```
max_dp[r][c] = max(max_dp[r-1][c], max_dp[r][c-1]) × grid[r][c]
min_dp[r][c] = min(min_dp[r-1][c], min_dp[r][c-1]) × grid[r][c]
```

**当 `grid[r][c] < 0` 时**：
```
max_dp[r][c] = min(min_dp[r-1][c], min_dp[r][c-1]) × grid[r][c]  // 大 × 负 = 小
min_dp[r][c] = max(max_dp[r-1][c], max_dp[r][c-1]) × grid[r][c]  // 小 × 负 = 大
```

### 边界处理

第一行和第一列没有"上方"或"左方"的多种选择，只有唯一路径，所以直接累乘即可。

---

## 四、代码实现

```cpp
class Solution {
public:
    const int MOD = 1e9 + 7;

    int maxProductPath(vector<vector<int>>& grid) {
        int m = grid.size();
        int n = grid[0].size();

        // max_dp[i][j]: 最大路径积
        // min_dp[i][j]: 最小路径积
        vector<vector<long long>> max_dp(m, vector<long long>(n));
        vector<vector<long long>> min_dp(m, vector<long long>(n));

        // 初始化起点
        max_dp[0][0] = min_dp[0][0] = grid[0][0];

        // 处理第一列：只能从上往下走
        for (int r = 1; r < m; r++) {
            max_dp[r][0] = min_dp[r][0] = max_dp[r - 1][0] * grid[r][0];
        }

        // 处理第一行：只能从左往右走
        for (int c = 1; c < n; c++) {
            max_dp[0][c] = min_dp[0][c] = max_dp[0][c - 1] * grid[0][c];
        }

        // 填表：枚举每个单元格
        for (int r = 1; r < m; r++) {
            for (int c = 1; c < n; c++) {
                if (grid[r][c] >= 0) {
                    max_dp[r][c] = max(max_dp[r - 1][c], max_dp[r][c - 1]) * grid[r][c];
                    min_dp[r][c] = min(min_dp[r - 1][c], min_dp[r][c - 1]) * grid[r][c];
                } else {
                    max_dp[r][c] = min(min_dp[r - 1][c], min_dp[r][c - 1]) * grid[r][c];
                    min_dp[r][c] = max(max_dp[r - 1][c], max_dp[r][c - 1]) * grid[r][c];
                }
            }
        }

        long long res = max(max_dp[m - 1][n - 1], min_dp[m - 1][n - 1]);

        if (res < 0) return -1;
        return res % MOD;
    }
};
```

---

## 五、复杂度分析

| 维度 | 分析 |
|------|------|
| **时间** | O(m × n) - 遍历矩阵一次 |
| **空间** | O(m × n) - 两个 DP 表（可优化到 O(n)） |

---

## 六、心得总结

### 思维转折点

1. **DFS 暴力不可行**：`C(28, 14) ≈ 4 × 10^8` 级别的路径数，任何优化都救不了
2. **双 DP 的必要性**：有负数时，最大可能变最小、最小可能变最大，必须同时追踪
3. **初始化不能想当然**：累乘初始化为 1 是常见错误，起点必须单独处理

### 什么时候用双 DP？

| 场景 | 单 DP 够用？ | 原因 |
|------|-------------|------|
| 累加路径最大和 | ✅ | 加法保序 |
| 累乘路径最大积（有负数）| ❌ | 负数会翻转大小关系 |
| 最大子数组和 | ✅ | 贪心可解 |

### 状态设计的通用思路

遇到"路径 + 最值 + 特殊运算"的问题时：

1. 先问自己：运算是否保序？（加法保序，乘法不保序）
2. 不保序 → 考虑多状态追踪
3. 看运算性质：乘法有负数 → 双状态；计数 → 单状态

---

## 七、相关题目

- [LeetCode 64. 最小路径和](https://leetcode.cn/problems/minimum-path-sum/) - 基础路径 DP
- [LeetCode 931. 下降路径最小和](https://leetcode.cn/problems/minimum-falling-path-sum/) - 多方向转移
- [LeetCode 152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/) - 同样是双 DP 思路
