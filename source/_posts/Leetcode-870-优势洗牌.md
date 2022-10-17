---
title: Leetcode-870.优势洗牌
date: 2022-10-10 20:57:36
tags: Leetcode
categories: 算法
---
### 题目
给定两个大小相等的数组 nums1 和 nums2，nums1 相对于 nums2 的优势可以用满足 nums1[i] > nums2[i] 的索引 i 的数目来描述。

返回 nums1 的任意排列，使其相对于 nums2 的优势最大化。

<!--more-->

#### 示例 1：
~~~bash
输入：nums1 = [2,7,11,15], nums2 = [1,10,4,11]
输出：[2,11,7,15]
~~~
#### 示例 2：
~~~bash
输入：nums1 = [12,24,8,32], nums2 = [13,25,32,11]
输出：[24,32,8,12]
~~~

- 提示：
~~~bash
1 <= nums1.length <= 105
nums2.length == nums1.length
0 <= nums1[i], nums2[i] <= 109
~~~
### 解题思路
依据题目，要想获取优势最大化，使用田忌赛马策略，首先将数组nums1进行排序，然后获取nums2排序后的下标数组
然后使用双指针的策略，遍历数组，将nums1[i]与nums2[下标]进行比较，若nums1[i]较大，则赋值到下标sort_Nums2_Index[start]处，并移动start，若nums1[i]较小，则赋值到下标sort_Nums2_Index[end]处，并移动end。采用田忌赛马策略，若我的最小比你的最小大，则对上你的最小，若我的最小比你的最小小，则对上你的最大。

### 代码
~~~C#
public static int[] AdvantageCount(int[] nums1, int[] nums2) 
{
    //将数组nums1进行排序
    Array.Sort(nums1);
    int[] result = new int[nums1.Length];
    int[] sort_Nums2_Index = new int[nums1.Length];
    for (int i = 0; i < nums1.Length; i++)
    {
        sort_Nums2_Index[i] = i;
    }
    //获取nums2排序后的下标数组
    Array.Sort(sort_Nums2_Index, (i, j) => nums2[i] - nums2[j]);
    int start = 0;
    int end = nums1.Length - 1;
    for (int i = 0; i < nums1.Length; i++)
    {
        if (nums1[i] > nums2[sort_Nums2_Index[start]])
        {
            result[sort_Nums2_Index[start]] = nums1[i];
            start++;
        }
        else 
        {
            result[sort_Nums2_Index[end]] = nums1[i];
            end--;
        }
    }
    return result;
}
~~~