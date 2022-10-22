---
title: Leetcode-76-最小覆盖子串
date: 2022-10-21 11:45:43
tags: Leetcode
categories: 算法
---

### 题目
给你一个字符串`s`、一个字符串`t`。返回`s`中涵盖`t`所有字符的最小子串。如果`s`中不存在涵盖`t`所有字符的子串，则返回空字符串`""`。

<!--more-->

注意：

对`t`中重复字符，我们寻找的子字符串中该字符数量必须不少于`t`中该字符数量。
如果`s`中存在这样的子串，我们保证它是唯一的答案。

#### 示例 1：
~~~bash
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
~~~

#### 示例 2：
~~~bash
输入：s = "a", t = "a"
输出："a"
~~~

#### 示例 3:
~~~bash
输入: s = "a", t = "aa"
输出: ""
解释: t 中两个字符 'a' 均应包含在 s 的子串中，
因此没有符合条件的子字符串，返回空字符串。
~~~

- 提示
1 <= s.length, t.length <= 105
s 和 t 由英文字母组成


### 解题思路
此题采用滑动窗口思想
- 定义start,end=0，为我们初始的滑动窗口
- 开始循环遍历整个数组元素，判断当前end指针是否超过整个数组的长度，是退出循环，否则执行第3步
- 然后end指针开始向右移动一个长度，并更新窗口内的区间数据
- 当窗口区间的数据满足我们的要求时，右指针end就保持不变，左指针start开始移动，直到移动到一个不满足要求的区间时，start不再移动位置
- 执行第2步

此题的重点在于如何判断数据满足条件，即`s`的字串涵盖`t`所有字符
此处采用的是定义一个长度为128的数组用于存放字符出现的次数，此处也可优化，根据提示由英文字母组成，可简化数组长度
    - 先将目标字符串的字符初始到数组中
    - 遍历s，即滑动窗口并更新s数组中的元素
    - 当t中元素不为0时，判断s中元素是否大于t中元素值

### 代码
~~~C#
public string MinWindow(string s, string t) 
{
    int[] sArray = new int[128];
    int[] tArray = new int[128];
    string result = string.Empty;
    int start = 0;
    int end = 0;
    for (int i = 0; i < t.Length; i++)
    {
        tArray[t[i]] += 1;
    }
    while (end < s.Length)
    {
        //移动左指针
        sArray[s[end++]] += 1;
        while (EqualsArray(sArray, tArray))
        {
            //更新result值
            if (result == string.Empty)
            {
                result = s.Substring(start, end - start);
            }
            else
            {
                result = end - start < result.Length ? s.Substring(start, end - start) : result;
            }
            //移动右指针
            sArray[s[start++]] -= 1;
        }
    }
    return result;
}

//此函数用于判断此时s的字串是否包含t中所有元素
public bool EqualsArray(int[] sArray, int[] tArray) 
{
    for (int i = 0; i < tArray.Length; i++)
    {
        if (tArray[i] != 0 && tArray[i] > sArray[i])
        {
            return false;
        }
    }
    return true;
}

~~~