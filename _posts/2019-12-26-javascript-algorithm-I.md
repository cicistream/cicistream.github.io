---
layout: post
title: "经典算法解 · 最大子序合（贪心·分治·动态规划）"
subtitle: "给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。"
date:  2019-12-26
author: "cici"
header-img: "img/post-bg-rwd.jpg"
tags:
  - JavaScript
  - algorithm
---

部分转载：[LeetCode](https://leetcode-cn.com/problems/maximum-subarray/solution/zui-da-zi-xu-he-by-leetcode/)题解

-------

## 题目：最大子序合

给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

示例:
```javascript

输入: [-2,1,-3,4,-1,2,1,-5,4],
输出: 6

```

解释: 连续子数组 `[4,-1,2,1]`的和最大，为6。

### 方法一：贪心算法

- 使用单个数组作为输入来查找最大（或最小）元素（或总和）的问题，贪心算法是可以在线性时间解决的方法之一。
- 每一步都选择最佳方案，到最后就是全局最优的方案。

```javascript

var maxSubArray = function(nums) {
    let ans = nums[0];
    let current = nums[0];
    for(let i = 1; i < nums.length; i++) {
        current = Math.max(nums[i], current+nums[i]);
        ans = Math.max(current, ans);  
    };
    return ans;
};

```

- 时间复杂度：O(*N*)。只遍历一次数组。
- 空间复杂度：O(*1*)。只遍历一次数组。

### 方法二：分治法

这个是使用分治法解决问题的典型的例子，并且可以用与合并排序相似的算法求解。下面是用分治法解决问题的模板：

* 定义基本情况。
* 将问题分解为子问题并递归地解决它们。
* 合并子问题的解以获得原始问题的解。

#### 算法

当最大子数组有 n 个数字时：
若 n==1，返回此元素。
`left_sum` 为最大子数组前 `n/2` 个元素，在索引为` (left + right) / 2 `的元素属于左子数组。
`right_sum` 为最大子数组的右子数组，为最后 `n/2` 的元素。
`cross_sum` 是包含左右子数组且含索引` (left + right) / 2` 的最大值。

![image](/img/algorithm-code-pic.png)

```javascript

var maxSubArray = function(nums) {
    const cross_sum = (nums, left, right, p)=> {
        if (left == right){return nums[left]}
        let curr_sum1 = 0;
        let left_subsum = Number.NEGATIVE_INFINITY;
        let right_subsum = Number.NEGATIVE_INFINITY;
        for(let i = p;i>left-1; i--){
            curr_sum1 += nums[i];
            left_subsum = Math.max(left_subsum, curr_sum1);
        }
        let curr_sum2 = 0
        for(let i = p+1;i<right+1; i++){
        curr_sum2 += nums[i];
        right_subsum = Math.max(right_subsum, curr_sum2);
        }
        return left_subsum + right_subsum;
    }
            
    const helper = (nums, left, right)=>{
        if (left == right) return nums[left];
        let p = Math.floor((left + right) / 2);
        let left_sum = helper(nums, left, p);
        let right_sum = helper(nums, p + 1, right);
        let crossSum = cross_sum(nums, left, right, p);
        return Math.max(left_sum, right_sum, crossSum);
    }
        
    return helper(nums, 0, nums.length - 1);
};

```

- 时间复杂度：O(*NlogN*)。
- 空间复杂度：O(*logN*)，递归时栈使用的空间。

### 动态规划（Kadane 算法）

#### 算法

在整个数组或在固定大小的滑动窗口中找到总和或最大值或最小值的问题可以通过动态规划（DP）在线性时间内解决。

有两种标准 DP 方法适用于数组：
- 常数空间，沿数组移动并在原数组修改。
- 线性空间，首先沿 **left->right**方向移动，然后再沿 **right->left** 方向移动。 合并结果。

我们在这里使用第一种方法，因为可以修改数组跟踪当前位置的最大和。
下一步是在知道当前位置的最大和后更新全局最大和。

```javascript

var maxSubArray = function(nums) {
    let ans = nums[0];
    let sum = 0;
    for(const num of nums) {
        if(sum > 0) {
            sum += num;
        } else {
            sum = num;
        }
        ans = Math.max(ans, sum);
    }
    return ans;
  }

```

- 时间复杂度：O(*N*)。只遍历一次数组。
- 空间复杂度：O(*1*)。只遍历一次数组。