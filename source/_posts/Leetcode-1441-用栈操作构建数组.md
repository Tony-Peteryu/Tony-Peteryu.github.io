---
title: Leetcode-1441-用栈操作构建数组
date: 2022-10-17 15:21:04
tags: Leetcode
categories: 算法
---

### 题目
给你一个数组`target`和一个整数`n`。每次迭代，需要从 `list = { 1 , 2 , 3 ..., n }`中依次读取一个数字。

<!--more-->

请使用下述操作来构建目标数组`target`：

- `"Push"`：从`list`中读取一个新元素， 并将其推入数组中。
- `"Pop"`：删除数组中的最后一个元素。
- 如果目标数组构建完成，就停止读取更多元素。

题目数据保证目标数组严格递增，并且只包含`1`到`n`之间的数字。

请返回构建目标数组所用的操作序列。如果存在多个可行方案，返回任一即可。


### 示例 1：
~~~bash
输入：target = [1,3], n = 3
输出：["Push","Push","Pop","Push"]
解释： 
读取 1 并自动推入数组 -> [1]
读取 2 并自动推入数组，然后删除它 -> [1]
读取 3 并自动推入数组 -> [1,3]
~~~

### 示例 2：
~~~bash
输入：target = [1,2,3], n = 3
输出：["Push","Push","Push"]
~~~

### 示例 3：
~~~bash
输入：target = [1,2], n = 4
输出：["Push","Push"]
解释：只需要读取前 2 个数字就可以停止。
~~~

### 解题思路
题目中给出的数组是严格递增的，因此我们可以遍历`target`，同时维护一个`temp`，从`1`开始

- 若`target`中的数不等于`temp`时
    - 添加[“Push”, “Pop”] 
- 若`target`中的数不等于`temp`时
    - 添加[“Push”]

遍历完成即数组构建完成

### 代码
~~~C#
public IList<string> BuildArray(int[] target, int n)
{
    int temp = 1;
    List<string> result = new List<string>();
    for (int i = 0; i < target.Length; i++)
    {
        while (target[i] != temp) 
        {
            result.Add("Push");
            result.Add("Pop");
            temp++;
        }
        result.Add("Push");
        temp++;
    }
    return result;
}
~~~