---
title: 二分答案模板 —— 求最大值 vs 求最小值
date: 2026-05-12 12:02:23
tags: [二分法]
categories: 算法学习
cover: /img/leetcode.jpg
---

# 二分答案模板 —— 求最大值 vs 求最小值

> **核心思想**：在答案的可能范围 `[L, R]` 内，通过二分猜测 + check 函数验证，快速找到最优解。
>
> 区别于标准二分在**数据**里查目标，二分答案是在**答案**里猜最优。

---

## 一、两种模板速览

### 模板 A：求最大值（最大的可行值）

```cpp
while (left < right) {
    int mid = (left + right + 1) / 2;
    if (check(mid)) {
        left = mid;
    } else {
        right = mid - 1;
    }
}
return left;
```

**使用场景**：求最大跳跃距离、最大载重、最大速度……

### 模板 B：求最小值（最小的可行值）

```cpp
while (left < right) {
    int mid = (left + right) / 2;
    if (check(mid)) {
        right = mid;
    } else {
        left = mid + 1;
    }
}
return left;
```

**使用场景**：求最少初始能量、最小化最大值、最早完成时间……

---

## 二、核心区别对比

| | 求最大值 | 求最小值 |
|--|---------|---------|
| mid 公式 | `(l+r+1)/2` ↑ 向上取整 | `(l+r)/2` ↓ 向下取整 |
| check 成立时 | `l = mid`（保留，向右） | `r = mid`（保留，向左） |
| check 不成立时 | `r = mid - 1` | `l = mid + 1` |
| 返回 | `left` | `left` |
| 边界长度 2 时 | `[3,4]` → mid=4 不卡死 | `[3,4]` → mid=3 不卡死 |

### 为什么不能混用？

以 `left=3, right=4` 的边界情况为例：

**求最大值时如果错误使用向下取整：**
```
mid = (3+4)/2 = 3
check(3)=true → left=mid=3  ← 区间没变！死循环 ❌
```

**求最小值时如果错误使用向上取整：**
```
mid = (3+4+1)/2 = 4
check(4)=true → right=mid=4  ← 区间没变！死循环 ❌
```

> **一句话记法**：
> - 求最大值 → `l = mid` → 怕死循环 → mid **向上**取整
> - 求最小值 → `r = mid` → 怕死循环 → mid **向下**取整

---

## 三、例题详解

### 例题 1：求最大值 —— LeetCode 1855. 下标对中的最大距离

**题目**：两个非递增数组 `nums1` 和 `nums2`，找 `i <= j` 且 `nums1[i] <= nums2[j]` 的最大 `j - i`。

**二分思路**：猜一个距离 `d`，检查是否存在这样的下标对。

```cpp
bool check(vector<int>& nums1, vector<int>& nums2, int dist) {
    for (int i = 0; i < nums1.size(); i++) {
        int j = i + dist;
        if (j < nums2.size() && nums1[i] <= nums2[j]) {
            return true;
        }
    }
    return false;
}

int maxDistance(vector<int>& nums1, vector<int>& nums2) {
    int left = 0, right = nums2.size() - 1;
    while (left < right) {                         // 求最大值模板
        int mid = (left + right + 1) / 2;          // ↑ 向上取整
        if (check(nums1, nums2, mid)) {
            left = mid;                            // 可行，保留
        } else {
            right = mid - 1;
        }
    }
    return left;
}
```

| 步骤 | 说明 |
|------|------|
| 二分范围 | `[0, nums2.size()-1]` |
| check | 遍历 i，看是否存在 `nums1[i] <= nums2[i+dist]` |
| 单调性 | 距离 d 可行 → 所有小于 d 的距离也可行 |

> 本题更优解是**双指针**（O(n)），利用了数组非递增的性质，详见原题笔记。

### 例题 2：求最小值 —— LeetCode 1665. 完成所有任务的最少初始能量

**题目**：`tasks[i] = [actual_i, minimum_i]`，完成消耗 actual，开始前需要达到 minimum。求能完成所有任务的最少初始能量。

**关键点**：
1. **排序策略**：按 `minimum - actual` 从大到小排序（差值越大的越要先做）
2. **二分下界**：`sum(actual)`（能量至少够总消耗）
3. **二分上界**：`sum(minimum)`（能量足够开启每个任务）

```cpp
class Solution {
public:
    bool check(vector<vector<int>>& tasks, int energy) {
        for (auto& t : tasks) {
            if (energy < t[1]) return false;  // 能量不够开始
            energy -= t[0];                    // 消耗能量
        }
        return true;
    }

    int minimumEffort(vector<vector<int>>& tasks) {
        // 1. 按差值降序排序（差值大的先做）
        sort(tasks.begin(), tasks.end(), [](auto& a, auto& b) {
            return a[1] - a[0] > b[1] - b[0];
        });

        // 2. 确定二分范围
        int left = 0, right = 0;
        for (auto& t : tasks) {
            left += t[0];
            right += t[1];
        }

        // 3. 求最小值模板
        while (left < right) {
            int mid = (left + right) / 2;        // ↓ 向下取整
            if (check(tasks, mid)) {
                right = mid;                     // 可行，保留，继续往左
            } else {
                left = mid + 1;
            }
        }
        return left;
    }
};
```

| 步骤 | 说明 |
|------|------|
| 排序 | 按 `min - actual` 降序 |
| 二分下界 | `sum(actual)`，总消耗 |
| 二分上界 | `sum(minimum)`，保证能开启每个任务 |
| check | 模拟按顺序执行，一旦能量不够就返回 false |
| 单调性 | 能量 E 可行 → 所有更大的能量也可行 |

### 为什么这样排序？

设 `[a1, m1]` 和 `[a2, m2]` 两个任务，先做 1 后做 2 需要的能量：

```
至少 max(m1, a1 + m2)
先做 2 后做 1：至少 max(m2, a2 + m1)
```

取较小者 → 需要 `m1 - a1 > m2 - a2` 时先做 1。所以按 `min - actual` 降序排列。

---

## 四、什么时候可以用二分答案？

| 特征 | 说明 |
|------|------|
| ✅ **答案有单调性** | "X 可行 → 所有比 X 大/小的也可行" |
| ✅ **容易验证** | 能写个 `check(mid)` 快速判断一个猜测是否可行 |
| ✅ **答案范围已知** | 知道答案在 `[L, R]` 之间 |
| ✅ **数据大但 check 快** | `O(n log Range)` 能过 |

### 和查找型二分的区别

| | 查找型二分 | 二分答案 |
|--|-----------|---------|
| 搜索对象 | **数据**（有序数组） | **答案**（猜测的范围） |
| 循环条件 | `left <= right` | `left < right` |
| 判断依据 | 中间值 vs 目标值 | `check(mid)` 是否可行 |
| 典型应用 | 找第一个 ≥ target | 求最大/最小可行值 |

---

## 五、相关题目

**求最大值：**
- [1855. 下标对中的最大距离](https://leetcode.cn/problems/maximum-distance-between-a-pair-of-values/)
- [875. 爱吃香蕉的珂珂](https://leetcode.cn/problems/koko-eating-bananas/)
- [1011. 在 D 天内送达包裹的能力](https://leetcode.cn/problems/capacity-to-ship-packages-within-d-days/)
- [1482. 制作 m 束花所需的最少天数](https://leetcode.cn/problems/minimum-number-of-days-to-make-m-bouquets/)

**求最小值：**
- [1665. 完成所有任务的最少初始能量](https://leetcode.cn/problems/minimum-initial-energy-to-finish-tasks/)
- [410. 分割数组的最大值](https://leetcode.cn/problems/split-array-largest-sum/)
- [LCR 074. 合并区间的最小次数](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)
- [1283. 使结果不超过阈值的最小除数](https://leetcode.cn/problems/find-the-smallest-divisor-given-a-threshold/)

---

## 六、心得总结

1. **两种模板不能混用** — mid 取整方向和 left/right 更新是对称的，`l = mid` 必须向上取整，`r = mid` 必须向下取整
2. **二分答案 vs 查找型二分** — 二分答案在答案区间上猜（`while <` + `check`），查找型二分在数据里搜（`while <=` + 返回目标）
3. **看 left 的更新方式就知道是哪种**：`l = mid` → 求最大值；`r = mid` → 求最小值
4. **翻车不可怕**，关键在于搞清楚翻车原因，把两种模板刻进脑子里

---

### 速查口诀

```
left = mid  向上取整 → 求最大值
right = mid 向下取整 → 求最小值
check 成立别丢掉，保留 mid 继续缩
check 不成立丢掉 mid，往另一侧缩
```
