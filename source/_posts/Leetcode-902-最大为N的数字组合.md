---
title: Leetcode-902-最大为N的数字组合
date: 2022-10-18 15:03:31
tags: Leetcode
categories: 算法
---

### 题目
给定一个按**非递减顺序**排列的数字数组`digits`。你可以用任意次数`digits[i]`来写的数字。例如，如果`digits`=`['1','3','5']`，我们可以写数字，如`'13'`,`'551'`, 和`'1351315'`。

返回 可以生成的小于或等于给定整数`n`的正整数的个数 。

<!--more-->

#### 示例 1：
~~~bash
输入：digits = ["1","3","5","7"], n = 100
输出：20
解释：
可写出的 20 个数字是：
1, 3, 5, 7, 11, 13, 15, 17, 31, 33, 35, 37, 51, 53, 55, 57, 71, 73, 75, 77.
~~~

#### 示例 2：
~~~bash
输入：digits = ["1","4","9"], n = 1000000000
输出：29523
解释：
我们可以写 3 个一位数字，9 个两位数字，27 个三位数字，
81 个四位数字，243 个五位数字，729 个六位数字，
2187 个七位数字，6561 个八位数字和 19683 个九位数字。
总共，可以使用D中的数字写出 29523 个整数。
~~~

#### 示例 2：
~~~bash
输入：digits = ["7"], n = 8
输出：1
~~~

- 提示
~~~bash
1 <= digits.length <= 9
digits[i].length == 1
digits[i] 是从 '1' 到 '9' 的数
digits 中的所有值都 不同 
digits 按 非递减顺序 排列
1 <= n <= 109
~~~

### 解题思路
根据题目意思，我们将问题拆分为两个部分
- 当构成的数位数小于目标数数位的时候，个数为digits.Length^Length（构成数数位），此时构成的数显然都比目标数要小
    - 例如示例2中，构成三位数字的个数为`3^3=27`,构成四位数字的个数为`3^4=81`

- 当构成的数位数等于目标数数位的时候，我们从前向后遍历目标数，并于digits中的数进行比较
    - 若digits中的数比遍历的数字小，使用临时变量`temp`存储数量
        - 此时使用这些数为开头的数都比目标数小，个数为temp*(digits.Length^(位数-1))
    - 若digits中的数和遍历的数字相等时，继续向后遍历，此时注意，若digits中的数都比遍历数字大的时候，停止遍历
    - 例如  digits = ["5"，"4" , "3", "2"],  n = 4167 时，比4小的数有两个，则有2 * 4^3=2 * 64=128,比1小的数没有，此时停止遍历。
    - 遍历到最末尾的时候，若还存在和遍历数字相等的数，需要将结果+1

### 代码
~~~C#
public int AtMostNGivenDigitSet(string[] digits, int n) 
{
    int result = 0;
    int tempAdd = 0;
    string target = n.ToString();
    int length = 1;
    int[] digitsTemp = new int[digits.Length];
    for (int i = 0; i < digits.Length; i++)
    {
        digitsTemp[i] = int.Parse(digits[i]);
    }
    //当构成的数位数小于目标数数位的时候
    while (target.Length > length) 
    {
        result += (int)Math.Pow(digits.Length, length);
        length++;
    }
    //当构成的数位数等于目标数数位的时候
    for (int i = 0; i < target.Length; i++)
    {
        int temp = 0;
        tempAdd = 0;
        int targetNum = int.Parse(target[i].ToString());
        for (int j = 0; j < digitsTemp.Length; j++)
        {
            //若digits中的数比遍历的数字小，使用临时变量`temp`存储数量
            if (targetNum > digitsTemp[j]) 
            {
                temp++;
            }
            //若digits中的数和遍历的数字相等时，继续遍历
            if (targetNum == digitsTemp[j]) 
            {
                tempAdd++;
            }
        }
        result += temp*(int)Math.Pow(digits.Length, length - i - 1);
        //若digits中的数都比遍历数字大的时候，停止遍历
        if (tempAdd == 0) 
        {
            break;
        }                
    }
    //遍历到最末尾的时候，若还存在和遍历数字相等的数，需要将结果，即加上tempAdd,此时tempAdd为1
    return result+tempAdd;
}
~~~
    