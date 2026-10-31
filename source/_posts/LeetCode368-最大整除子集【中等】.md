---
title: LeetCode368.最大整除子集【中等】
date: 2025-04-06 23:46:32
tags: [数组, 动态规划]
categories: 算法学习
cover: /img/leetcode.webp
swiper_index: 8
---

## 相关标签：

> 数组、动态规划

## 题目简介：

> 给你一个由 无重复 正整数组成的集合 `nums` ，请你找出并返回其中最大的整除子集 `answer` ，子集中每一元素对 `(answer[i], answer[j])` 都应当满足：
>
> - `answer[i] % answer[j] == 0` ，或
> - `answer[j] % answer[i] == 0`
>
> 如果存在多个有效解子集，返回其中任何一个均可。

## 示例：

> 示例 1：
>
> ```java
> 输入：nums = [1,2,3]
> 输出：[1,2]
> 解释：[1,3] 也会被视为正确答案。
> ```

> 示例 2：
>
> ```java
> 输入：nums = [1,2,4,8]
> 输出：[1,2,4,8]
> ```

> 提示：
>
> - `1 <= nums.length <= 1000`
> - `1 <= nums[i] <= 2 * 109`
> - `nums` 中的所有整数 互不相同

## 解题思路：

> 这道题其实一开始我还是想的是先求出所有子集，然后对于每个子集去判断是否能够满足相互整除的要求。但是看了一下 nums 数组的长度好吧这个做法肯定会超内存限制。
>
> 然后我想到提前剪枝先去判断 nums 中是否有满足相互整除的一对元素，如果没有就说明最大的整除子集的大小就是 1 了，所以只需要从 nums 中任意取出一个元素返回即可，如果有这样一对元素那就说明最大的整除子集大小大于 1，这样就可以不去存取大小为 1 的子集可以节约一点内存。
>
> 但是一顿分析过后发现本质上空间复杂度还是 2 的 n 次方，所以先存储子集的做法肯定是行不通了。

> 看了一下本题的标签有一个动态规划，不是这怎么和动态规划扯上关系的？
>
> dp\[i\]能表示什么？我们想要找出满足要求的最大子集，但是我们又不能直接存储子集，所以我们可以用 dp\[i\]来表示以 nums\[i\]结尾的最大整除子集的大小。
>
> 使用两层循环，外层遍历 nums 内层从 nums\[0\]遍历到当前元素 nums\[i - 1\]在遍历的过程中去判断当前元素是否能够被 nums\[i\]整除如果能够整除就证明它满足要求可以被加入到子集中去这个时候就可以使用状态转移方程转移 dp\[i\]的状态。不过本题要求双向整除都可以，所以为了简化判断条件我们可以先对 nums 进行一次升序排序，这样就只需要去判断大的整除小的了。

> 下面是具体实现步骤：

```java
for (int i = 1; i < n; i++) {
    for (int j = 0; j < i; j++) {
        if (nums[i] % nums[j] == 0 && dp[j] + 1 > dp[i]) {
            // 跟新dp
            dp[i] = dp[j] + 1;
        }
    }
}
```

> 但是现在还有一个问题要解决：
>
> dp 只能解决找到最大的子集的大小，但是不能记录这个子集中有哪些元素。
>
> 怎么解决呢？
>
> 其实只需要再引入一个路径数组 pre\[\]在每次 dp 跟新的时候同时记录当前元素的下标即可，不过需要注意的是这个路径数组记录的是前缀路径对于最后一步是没有记录的，所以这里我们还要再定义一个变量 maxIndex 来记录当前 dp\[\]的最后一个元素。同时再使用一个变量 maxSize 来记录最长子集的大小。

## 题解：

```java
class Solution {

    public List<Integer> largestDivisibleSubset(int[] nums) {
        int n = nums.length;
        // 把数组排序一下，这样就只用讨论从后面对于前面取模
        Arrays.sort(nums);
        // dp[i] 表示以nums[i]结尾的最大整除子集的大小
        int[] dp = new int[n];
        // pre用来存储转移路径为了最后构造结果集合
        int[] pre = new int[n];
        Arrays.fill(dp, 1);
        Arrays.fill(pre, -1);
        int maxSize = 1;
        int maxIndex = 0;

        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[i] % nums[j] == 0 && dp[j] + 1 > dp[i]) {
                    // 跟新dp和前缀索引
                    dp[i] = dp[j] + 1;
                    pre[i] = j;
                }
            }
            if (dp[i] > maxSize) {
                // 跟新最大子集信息
                maxSize = dp[i];
                maxIndex = i;
            }
        }

        // 构造结果集合
        List<Integer> res = new ArrayList<>();
        while(maxIndex != -1) {
            res.add(nums[maxIndex]);
            maxIndex = pre[maxIndex];
        }

        return res;

    }

}
```

## 思考：

> 做完这道题我受到了启发，leetcode300 最长递增子序列的变式，要求不仅使用 dp 求出最长子序列的长度，还要将这个子序列给保存下来
>
> 那不就是相当于再用一个 pre 路径数组来记录子序列的下标吗？有了这些下标那最后就可以还原出最长子序列了！！！

> 这里贴一下修改后的代码，如果想看原题的可以点下面的链接跳转

[300. 最长递增子序列 - 力扣（LeetCode）][300. _ - _LeetCode]

```java
class Solution {
    public List<Integer> lengthOfLIS(int[] nums) {
        int n = nums.length;
        // dp[i] 表示值在 [0, i - 1] 范围内的严格递增的数有多少
        int[] dp = new int[n];
        int[] pre = new int[n];
        // 每个元素至少可以单独构成一个子序列
        Arrays.fill(dp, 1);
        Arrays.fill(pre, -1);
        int maxLength = 1;
        int maxIndex = 0;
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i] && dp[j] + 1 > dp[i]) {
                    // 用一个数组去记录前面的情况，这样当前位置就能够不断的去更新最大值
                    dp[i] = dp[j] + 1;
                    pre[i] = j;
                }
            }
            // 更新最大的dp[i]
            if (dp[i] > maxLength) {
                maxLength = dp[i];
                maxIndex = i;
            }
        }
        List<Integer> res = new ArrayList<>();
        while (maxIndex != -1) {
            res.add(nums[maxIndex]);
            maxIndex = pre[maxIndex];
        }
        Collections.sort(res);
        return res;
    }
}
```

[300. _ - _LeetCode]: https://leetcode.cn/problems/longest-increasing-subsequence/?envType=study-plan-v2&envId=top-100-liked
