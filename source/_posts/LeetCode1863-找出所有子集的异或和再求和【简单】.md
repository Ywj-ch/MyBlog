---
title: LeetCode1863-找出所有子集的异或和再求和【简单】
date: 2025-04-05 20:02:12
tags: [位运算, 数组, 回溯]
categories: 算法学习
cover: /img/leetcode.webp
swiper_index: 7
---

## 相关标签：

> 位运算、数组、回溯

## 题目简介：

> 一个数组的 异或总和 定义为数组中所有元素按位 `XOR` 的结果；如果数组为 空 ，则异或总和为 `0` 。
>
> - 例如，数组 `[2,5,6]` 的 异或总和 为 `2 XOR 5 XOR 6 = 1` 。
>
> 给你一个数组 `nums` ，请你求出 `nums` 中每个 子集 的 异或总和 ，计算并返回这些值相加之 和 。
>
> 注意：在本题中，元素 相同 的不同子集应 多次 计数。
>
> 数组 `a` 是数组 `b` 的一个 子集 的前提条件是：从 `b` 删除几个（也可能不删除）元素能够得到 `a` 。

## 示例：

> 示例 1：
>
> ```java
> 输入：nums = [1,3]
> 输出：6
> 解释：[1,3] 共有 4 个子集：
> - 空子集的异或总和是 0 。
> - [1] 的异或总和为 1 。
> - [3] 的异或总和为 3 。
> - [1,3] 的异或总和为 1 XOR 3 = 2 。
> 0 + 1 + 3 + 2 = 6
> ```

> 示例 2：
>
> ```java
> 输入：nums = [5,1,6]
> 输出：28
> 解释：[5,1,6] 共有 8 个子集：
> - 空子集的异或总和是 0 。
> - [5] 的异或总和为 5 。
> - [1] 的异或总和为 1 。
> - [6] 的异或总和为 6 。
> - [5,1] 的异或总和为 5 XOR 1 = 4 。
> - [5,6] 的异或总和为 5 XOR 6 = 3 。
> - [1,6] 的异或总和为 1 XOR 6 = 7 。
> - [5,1,6] 的异或总和为 5 XOR 1 XOR 6 = 2 。
> 0 + 5 + 1 + 6 + 4 + 3 + 7 + 2 = 28
> ```

> 示例 3：
>
> ```java
> 输入：nums = [3,4,5,6,7,8]
> 输出：480
> 解释：每个子集的全部异或总和值之和为 480 。
> ```

## 解题思路：

> 本题我首先想到的就是首先求出数组的所有子集，根据题目的要求这个子集包含空集和全集，所以求取子集我第一个想到的方法就是使用回溯算法，拿到每一个子集后再分别遍历去求异或的值最后再加起来就可以得到答案了

> 这里值得一说的就是回溯算法，其实不管是求取子集还是求取全排列，回溯算法的核心思路就是在循环遍历的时候先将当前值加到临时集合里面去，然后再次递归调用同时将循环向后移动一位，最后就是删除掉临时集合的最后一个元素，这三步是比较固定的，唯一不同的点就是什么时候将临时集合加到目标集合里面去，这就要根据题目的要求来判断了。

求子集：

```java
class Solution {
    // 我这里习惯于将这两个集合设置为全局变量，这样就不用在函数中传递了
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> temp = new ArrayList<>();
    public List<List<Integer>> subsets(int[] nums) {
        // 回溯的两个参数非别为nums数组和开始遍历的下标
        backtrace(0, nums);
        return res;
    }

    private void backtrace(int start, int[] nums) {
        // 由于子集没有条件所以直接加入
        res.add(new ArrayList<>(temp));
        for (int i = start; i < nums.length; i++) {
            // 第一步
            temp.add(nums[i]);
            // 第二步
            backtrace(i + 1, nums);
            // 第三步
            temp.removeLast();
        }
    }
}
```

求全排列：

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> temp = new ArrayList<>();
    public List<List<Integer>> permute(int[] nums) {
        HashSet<Integer> set = new HashSet<>();
        backtrack(nums,set);
        return res;
    }

    private void backtrack(int[] nums, List<Integer> temp, HashSet<Integer> set, List<List<Integer>> res) {
        // 根据本题条件要求长度跟nums一样时才加入
        if (temp.size() == nums.length) {
            res.add(new ArrayList<>(temp));
            return;
        }
        for (int num : nums) {
            if (set.contains(num)) {
                continue;
            }
            temp.add(num);
            set.add(num);
            backtrack(nums, temp, set, res);
            set.remove(num);
            temp.removeLast();
        }
    }
}
```

## 题解：

> 经过上面这么长的铺垫，相信这道题已经没有什么难点了

```java
class Solution {
    List<List<Integer>> list = new ArrayList<>();
    List<Integer> temp = new ArrayList<>();
    public int subsetXORSum(int[] nums) {
        // 先求出所有的子集
        backtrace(nums, list, temp, 0);
        // 再计算每一个子集的异或并求和
        int res = 0;
        for (int i = 0; i < list.size(); i++) {
            List<Integer> arr = list.get(i);
            int flag = 0;
            for (int j = 0; j < arr.size(); j++) {
                flag ^= arr.get(j);
            }
            res += flag;
        }
        return res;
    }

    // 使用回溯算法可以很自然的求出nums的所有子集
    public void backtrace(int[] nums, List<List<Integer>> list, List<Integer> temp, int begin) {
        list.add(new ArrayList<>(temp));
        for (int i = begin; i < nums.length; i++) {
            temp.add(nums[i]);
            backtrace(nums, list, temp, i + 1);
            temp.removeLast();
        }
    }

}
```

## 其他解法：

> 我们发现其实只要将所有的元素进行一边或运算（|），然后再将结果左移数组的长度减一个单位最后得到的结果即为最终答案

```java
class Solution {
    public int subsetXORSum(int[] nums) {
        int or = 0;
        for (int x : nums) {
            or |= x;
        }
        return or << (nums.length - 1);
    }
}
```

详细的推导过程可以见：

[1863. 找出所有子集的异或总和再求和 - 力扣（LeetCode）][1863. _ - _LeetCode]

[1863. _ - _LeetCode]: https://leetcode.cn/problems/sum-of-all-subset-xor-totals/solutions/3614974/on-shu-xue-zuo-fa-pythonjavaccgojsrust-b-9txy/?envType=daily-question&envId=2025-04-05
