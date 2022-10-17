---
title: Leetcode-453-最小操作次数使数组元素相等
date: 2022-10-12 19:55:19
tags: Leetcode
categories: 算法
---

### 题目
给你一个长度为 n 的整数数组，每次操作将会使 n - 1 个元素增加 1 。返回让数组所有元素相等的最小操作次数。

<!--more-->

### 示例 1：
~~~bash
输入：nums = [1,2,3]
输出：3
解释：
只需要3次操作（注意每次操作会增加两个元素的值）：
[1,2,3]  =>  [2,3,3]  =>  [3,4,3]  =>  [4,4,4]
~~~

### 示例 2：
~~~bash
输入：nums = [1,1,1]
输出：0
~~~

- 提示
~~~bash
n == nums.length
1 <= nums.length <= 105
-109 <= nums[i] <= 109
答案保证符合 32-bit 整数
~~~


### 解题思路

题目为每次操作将会让 n - 1 个元素增加 1 ，反着理解的话即为每次将 1 个元素减少 1 让数组中元素相等，这样就变为了数组中所有元素变为数组中最小元素需要的次数，先获取最小值，然后差值求和

### 代码
~~~C#
public int MinMoves(int[] nums) 
{
    int minNum = nums[0];
    int result = 0;
    //此处用于获取数组中的最小元素
    for (int i = 0; i < nums.Length; i++)
    {
        minNum = Math.Min(minNum, nums[i]);
    }
    //差值求和
    for (int i = 0; i <nums.Length; i++)
    {
        result += (nums[i]=minNum);
    }
    return result;
}
~~~
