---
title: BFS和哈希表优化同值跳跃
date: 2026-05-18 10:00:00
tags: [BFS, 哈希表, 图优化]
categories: 算法学习
cover: /img/leetcode.jpg
---

# BFS + 哈希表优化同值跳跃

> **日期**: 2026-05-18 
> 难度**: Hard **
> **标签**: BFS、哈希表、图优化、稀疏图
> 题目链接：[LeetCode 1345. Jump Game IV](https://leetcode.cn/problems/jump-game-iv/)

---

## 题目描述

一个整数数组 `arr`，从下标 0 出发，每次可以：
- 跳到 `i+1`（不越界）
- 跳到 `i-1`（不越界）
- 跳到任意与 `arr[i]` 值相同的下标

求到达最后一个下标 `n-1` 的最少跳跃次数。

---

## 第一次思路：邻接矩阵（卡住了）

第一时间想到的就是标准的 BFS 求无权图最短路径。那就先把图建出来嘛——三个规则都很清晰：

- 左邻居：`i → i-1`
- 右邻居：`i → i+1`
- 同值跳跃：`i → j` if `arr[i] == arr[j]`

于是写了一个两层循环去建邻接矩阵：

```cpp
vector<vector<int>> graph(n, vector<int>(n, -1));
for (int i = 0; i < n; i++) {
    if (i - 1 >= 0) graph[i][i - 1] = 1;
    if (i + 1 < n) graph[i][i + 1] = 1;
    for (int j = 0; j < n; j++) {
        if (j != i && arr[j] == arr[i]) graph[i][j] = 1;
    }
}
```

然后 BFS 层序遍历找最短步数。

**结果：内存爆炸 + 超时。**

### 踩坑 1：邻接矩阵 O(n²) 根本存不下

`n` 最大 `5×10⁴`，邻接矩阵需要 `2.5×10⁹` 个元素，每个 int 4 字节就是 **10 GB**。直接超内存。

### 踩坑 2：构建图的三层循环 O(n²) 时间

构建图本身就是 O(n²)，`2.5×10⁹` 次操作，必超时。

### 踩坑 3：BFS 内层又遍历全图 O(n²)

BFS 里对每个节点又 `for (int k = 0; k < n; k++)` 遍历所有邻居，又是 O(n²)。

### BFS 里一个隐蔽的 bug

后来修 BFS 的时候还发现了一个变量名冲突：

```cpp
for (int j = 0; j < size; j++) {     // j 是层内遍历索引
    ...
    if (j != i && !visited[j] ...)   // 这里的 j 又是"邻居下标"
}
```

`j` 同时用来计数和当下标，完全不是同一个意思。BFS 根本没在遍历邻居，只是在检查恰好等于计数值的位置。修成 `for (int neighbor = 0; ...)` 就好了。

---

## 最终方案：哈希表预处理 + BFS

真正的突破来自意识到这个图是非常稀疏的——每个节点只有最多 3 种类型的邻居，不应该显式建图，而应该在 BFS 过程中**实时计算邻居**。

### 关键洞察

1. 左/右邻居：直接 `i-1` / `i+1`，判界就行，O(1)
2. 同值邻居：用 `unordered_map<int, vector<int>>` 预处理，把每个值对应的所有下标存起来
3. **用完就 erase**：这是最关键的优化——当第一次访问到某个值的节点时，把它的所有同值下标都入队，然后立即从哈希表中删除这个键。这样每个值对应的下标列表只会被遍历一次，总共 O(n)

---

## 完整代码

```cpp
class Solution {
public:
    int minJumps(vector<int>& arr) {
        int n = arr.size();
        if (n == 1) return 0;

        // 1. 预处理：值 → 下标列表
        unordered_map<int, vector<int>> umap;
        for (int i = 0; i < n; i++) {
            umap[arr[i]].push_back(i);
        }

        // 2. BFS
        vector<bool> visited(n, false);
        visited[0] = true;
        deque<int> dq;
        dq.push_back(0);
        int step = 0;

        while (!dq.empty()) {
            int size = dq.size();
            for (int j = 0; j < size; j++) {
                int i = dq.front();
                dq.pop_front();
                if (i == n - 1) return step;

                // 左邻居
                if (i - 1 >= 0 && !visited[i - 1]) {
                    visited[i - 1] = true;
                    dq.push_back(i - 1);
                }
                // 右邻居
                if (i + 1 < n && !visited[i + 1]) {
                    visited[i + 1] = true;
                    dq.push_back(i + 1);
                }
                // 同值跳跃：第一次遇到这个值时，所有同值下标一起入队
                if (umap.count(arr[i])) {
                    for (int next : umap[arr[i]]) {
                        if (!visited[next]) {
                            visited[next] = true;
                            dq.push_back(next);
                        }
                    }
                    umap.erase(arr[i]);  // 关键：用完马上删，避免重复遍历
                }
            }
            step++;
        }
        return -1;
    }
};
```

---

## 复杂度分析

| 复杂度 | 分析 |
|--------|------|
| 时间复杂度 | O(n) — 每个下标入队一次，每个值的列表遍历一次后删除，总操作为 O(n) |
| 空间复杂度 | O(n) — 哈希表 O(n)、队列 O(n)、visited 数组 O(n) |

---

## 心得总结

1. **邻接矩阵是坑，稀疏图用邻接表或动态生成邻居。** 看到 `n ≤ 10⁵` 就别想矩阵了，内存和时间都不允许。

2. **BFS 处理同值跳跃时一定要及时清空列表。** 不然同一个值的所有下标会被反复遍历，退化成 O(n²)。用 `umap.erase()` 彻底删除键，不要只 clear 向量。

3. **变量名不要乱用。** BFS 层序遍历的索引和邻居下标要用不同的变量名，`j` 同时干两件事这种 bug 肉眼很难看出来，但跑起来就是死循环。

4. **图和最短路径的组合套路：** 当图是隐式定义的（规则给出邻居，而不是显式的 edge list），不要在内存里建完整图，在 BFS 里实时算邻居就好。
