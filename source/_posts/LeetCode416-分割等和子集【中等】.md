---
title: LeetCode416.分割等和子集【中等】
date: 2025-04-07 19:24:31
tags: [数组, 动态规划]
categories: 算法学习
cover: /img/leetcode.webp
swiper_index: 9
---

## 相关标签：

> 数组、动态规划

## 题目简介：

> 给你一个 只包含正整数 的 非空 数组 `nums` 。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

## 示例：

> 示例 1：
>
> ```java
> 输入：nums = [1,5,11,5]
> 输出：true
> 解释：数组可以分割成 [1, 5, 5] 和 [11] 。
> ```

> 示例 2：
>
> ```java
> 输入：nums = [1,2,3,5]
> 输出：false
> 解释：数组不能分割成两个元素和相等的子集。
> ```

> 提示：
>
> - `1 <= nums.length <= 200`
> - `1 <= nums[i] <= 100`

## 解题思路：

> 题目要求我们分割等和子集，那首先想到的就是排除和为奇数的情况，当和为偶数时才有讨论的必要。我们现在的目标就变成了判断能否从数组中找出一些元素凑成总和的一半。
>
> 诶，这个描述是不是有一点耳熟，我换一个说法你就明白了，给你一个背包，让你判断能否将背包装满。所以这道题本质上就是背包问题，而且是 01 背包问题（不允许重复）。
>
> 唯一需要注意的一点就是这里的 dp 数组需要定义成 boolean 类型同时状态转移的条件也要做适宜的改动。

## 题解：

```java
class Solution {
    public boolean canPartition(int[] nums) {
        int n = nums.length;
        int sum = 0;
        for (int i = 0; i < n; i++) {
            sum += nums[i];
        }
        // 如果为奇数那就肯定不可能分成两个等和子集
        if (sum % 2 != 0) {
            return false;
        }
        // 现在就是看数组中的元素能否凑够half了
        int half = sum / 2;
        // dp[i] 表示能否凑出i
        boolean[] dp = new boolean[half + 1];
        // 这里要先初始化一下
        // 初不初始化要看你的具体问题，比如我这里0肯定满足要求所以要初始化
        // 而像经典的背包问题就是不需要初始化的，应为当背包为空时肯定对应价值也为0
        dp[0] = true;
        for (int i = 0; i < n; i++) {
            for (int j = half; j >= nums[i]; j--) {
                // 从后往前遍历保证当前的值是上一次遍历的结果，而不是当前遍历的结果由此来保证唯一性
                if (dp[j - nums[i]]) {
                    dp[j] = true;
                }
            }
        }
        return dp[half];
    }
}
```

## 思考：

> 这里再由此引申一下背包问题
>
> 背包问题通常分为 4 种
>
> - 01 背包问题
> - 完全背包问题
> - 多重背包问题
> - 分组背包问题

> 这里只简单提一下前面两种，如果需要详细学习可以参考下面的文章：
>
> [背包问题详解（01 背包，完全背包，多重背包，分组背包）-CSDN 博客][01_-CSDN]

> 背包问题：
>
> 给定容量为 W 的背包和 N 个物品，每个物品有重量 W\[i\]和价值 V\[i\]

> 01 背包问题
>
> 内层循环逆序遍历（保证每个物品只选一次）

```java
int[] dp = new int[W + 1];
for (int i = 0; i < N; i++) {
    for (int j = W; j >= w[i]; j--) {
        dp[j] = Math.max(dp[j], dp[j - w[i]] + v[i]);
    }
}
```

> 如何理解反向遍历就可以保证唯一性：
>
> 通过从大到小遍历，我们可以保证在更新 dp\[i\] 时，dp\[i - num\] 的值是上一次遍历的结果，而不是当前遍历的结果。

> 完全背包问题
>
> 内层循环正序遍历（允许重复选择）

```java
int[] dp = new int[W + 1];
for (int i = 0; i < N; i++) {
    for (int j = w[i]; j <= W; j++) {
        dp[j] = Math.max(dp[j], dp[j - w[i]] + v[i]);
    }
}
```

[01_-CSDN]: https://blog.csdn.net/2301_79558858/article/details/137546255?ops_request_misc=%257B%2522request%255Fid%2522%253A%25220fc10c1a82166811625662ee4adfe67b%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=0fc10c1a82166811625662ee4adfe67b&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-137546255-null-null.142^v102^pc_search_result_base3&utm_term=%E8%83%8C%E5%8C%85%E9%97%AE%E9%A2%98&spm=1018.2226.3001.4187
