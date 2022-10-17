---
title: Linq
date: 2022-10-12 20:06:17
tags: Linq
categories: C#
---

### LINQ介绍
#### 什么是LINQ
LINQ是英文Language Integrated Query的缩写，中文译为语言集成查询

LINQ(语言集成查询)是C#、VB.NET以及F#中的统一查询语法，用以检索来自不同数据源和格式的数据。它集成在c#、VB以及F#语言中，从而消除了编程语言和数据库之间的不匹配，并为不同类型的数据源提供了一个单一的查询接口。

<!--more-->

#### LINQ的优势
- 熟悉的语言：C#，VB.NET，F#开发者无需学习额外的语法
- 写更少的代码：与更传统的方法相比，LINQ减少了需要编写的代码量
- 可读性更高：LINQ使代码更具可读性，因此其他开发人员可以轻松地理解和维护它
- 标准化方式查询多个数据源：可以使用同样的LINQ语法查询多个数据源
- 编译时查询的安全性：LINQ查询支持在编译时提供对象的类型检查
- 智能感知的支持：LINQ为泛型集合提供了智能感知
- 数据重塑：LINQ查询可以检索并返回不同类型的数据

### LINQ标准查询操作符
#### LINQ操作符之Where
Where操作符(Linq扩展方法)基于给定的条件表达式过滤集合，并返回一个新的集合。
~~~C#
### 查询订单中价格大于2000 且为线上的订单
var list = order.Where(e => e.Price > 2000).Where(e => e.Source == "线上").ToList();
~~~
`tips:`需要注意的是LINQ查询具有`延迟执行`的特性
- 它是指查询操作并不是在查询运算符定义的时候执行，真正使用集合中的数据时才执行，例如遍历数据集合时调用MoveNext方法会触发查询操作
- 当返回值为IEnumerable<TSource>、IEnumerable<IGrouping<TKey, TSource>>和IOrderedEnumerable<TSource>的时候，Linq为延迟执行。实际上上述三个返回值类型都实现了IEnumerable<T>(公开枚举数)这个接口，yield return的返回值就是IEnumerable<T>，所以，当Linq查询操作符的返回值为上述三个类型时，查询为延迟查询，其他为立即执行。
- 如何让查询立即执行呢，可以使用LINQ内置所提供的转换操作的定义：ToArray(转换为数组)、ToDictionary(转换为字典)、ToList(转换为集合)
- 在使用的时候要注意这个特性，避免重复查询出现

#### LINQ操作符之OfType
OfType操作符用于根据指定的类型过滤IEnumerable中的元素，它返回一个包含指定类型的IEnumerable<T>的子集合。
~~~C#
### 从中筛选出类型为int类型的元素
var lists = new List<object> { "asdas", "asdadf", 1, 2, 10.5, 15.45 };
var result = lists.OfType<string>().ToList();
~~~

#### LINQ操作符之OrderBy和OrderByDescending
在LINQ的查询语法中，OrderBy按升序或降序对集合的值进行排序。默认情况下，它按升序对集合进行排序，因为ascending关键字在这里是可选的。使用descending关键字对集合进行降序排序。

在LINQ的方法语法中，OrderBy则只能按升序对集合的值进行排序，而OrderByDescending则按降序排序，并且OrderByDescending只适用于方法语法中。
~~~C#
### 依据订单的价格升序排列
var result = list.OrderBy(e => e.Price).ToList();
~~~

#### LINQ操作符之ThenBy和ThenByDescending
OrderBy()方法根据指定的字段按升序对集合进行排序。在OrderBy之后使用ThenBy()方法对另一个字段上的集合按升序排序。LINQ首先根据OrderBy方法指定的主字段对集合进行排序，然后根据ThenBy方法指定的次字段对结果集合进行升序排序。
~~~C#
### 先依据的价格升序排列，再根据订单创建时间降序排列
var result = list.OrderBy(e=>e.Price).ThenByDescending(e=>e.CreatedAt).ToList();
~~~

#### LINQ操作符之 GroupBy
分组操作符根据给定的键创建一组元素。这个组包含在一个特殊类型的集合中，该集合实现了一个IGrouping<TKey,TSource>接口，其中TKey是一个键值，在这个键值上形成了组，而TSource是与分组键值匹配的元素集合。
~~~C#
### 依据订单来源进行分组排序
var result = list.GroupBy(e=>e.Source);
foreach (var item in result)
{
    Console.WriteLine(item.Key);
    foreach (var value in item)
    {
        Console.WriteLine(JsonConvert.SerializeObject(value));
    }
}
~~~
同时可以依据多字段进行分组排序
~~~C#
### 依据订单及用户分组排序
var result = list.GroupBy(e=>new TypeA(e.Source,e.Customer)).ToList();
~~~

#### LINQ操作符之ToLookup
ToLookup操作符是一个扩展方法，它用于从源集合中提取一组键/值对。在这里，结果集合中的每个元素都是一个通用的Lookup对象，该对象包含与该键匹配的键和子项。
LINQ方法语法中，ToLookup与GroupBy相同，唯一的区别是GroupBy的执行是延迟的，而ToLookup的执行是立即的。

~~~C#
var result = list.ToLookup(e=>e.Source)
~~~

#### LINQ操作符之Join
内部连接生成一个结果集，其中第一个集合的每个元素对于第二个集合中的每个匹配元素都出现一次。如果第一个集合中的元素在第二个集合中没有任何匹配的元素，那么它就不会出现在结果集中。
内连接仅用于从两个数据源返回匹配的元素，而从结果集中删除不匹配的元素，如图所示
![Linq内连接](https://img-blog.csdnimg.cn/c7059d53e12a48e0a4751e41915267b2.png)
~~~C#
var result = customers.                // 1.外部数据源
    Join(
    addresses,                         // 2.内部数据源
    c => c.AddressId,                  // 3.外部键选择器
    e => e.Id,                         // 4.内部键选择器
    (customers, addresses) => new      // 5.期望返回的结果集选择器
    {
        name = customers.Name,
        province = addresses.Province,
        city = addresses.City,
        district = addresses.District,
        street = addresses.Street
    }
    ).ToList();
~~~
外部数据源中的外部键起到驱动作用，从内部中寻找匹配的数据，并按照结果集选择器中的去返回

#### LINQ操作符之GroupJoin
GroupJoin基本上是用来生成分组数据结构的。来自第一个数据源的每个项都与来自第二个数据源的一组相关项配对
~~~C#
 var customers = FakeData.Customers;
 var addresses = FakeData.Addresses;
 var result = addresses.                // 1.外部数据源
     GroupJoin(
     customers,                         // 2.内部数据源
     c => c.Id,                         // 3.外部键选择器
     e => e.AddressId,                  // 4.内部键选择器
     (addresses, customers) => new      // 5.期望返回的结果集选择器
     {
         addresses,
         customers
     }
     ).ToList();
~~~

#### LINQ操作符之Select
投影是用于从数据源中选择数据的一种机制。你可以选择与数据源相同形式的数据(即原始数据处于其原始状态)。还可以通过对数据执行一些操作来创建新的数据形式。
LINQ中的Select操作符也允许我们指定我们想要检索的属性，你是想检索所有的属性，还是一些你需要在Select操作符中指定的属性。标准的LINQ选择操作符也允许我们执行一些计算。
~~~C#
var result = orders.Select(e => e.Customer).ToList();
~~~

#### LINQ操作符之SelectMany
LINQ的SelectMany操作符是将序列的每个元素投影到IEnumerable<T>并将结果序列合并为一个序列。这意味着SelectMany操作符组合来自一系列结果的记录，然后将其转换为一个结果。
~~~C#
var result = petOwners
        .SelectMany(petOwner => petOwner.Pets, (petOwner, petName) => new { petOwner.Name, petName }).Where(e=>e.petName.Contains("S"));
~~~

#### LINQ操作符之All
C#中LINQ的All操作符用于检查数据源的所有元素是否满足给定的条件。如果所有元素都满足条件，则返回true，否则返回false。
~~~C#
### 判断数组中是否所有元素都大于20
var numbers = new[] { 1, 3, 8, 16, 20, 30, 56, 80 };
var areAllNumbersGreaterThan10 = numbers.All(x => x > 20);
~~~

#### LINQ操作符之Any
C#中LINQ的Any操作符用于检查数据源中是否至少有一个元素满足给定的条件。如果任何元素满足给定条件，则返回true，否则返回false。它也用于检查一个集合是否包含一些数据。这意味着它还检查集合的长度。如果它包含任何数据，则返回true，否则返回false。
~~~C#
### 判断数组中是否存在元素大于20
var numbers = new[] { 1, 3, 8, 16, 20, 30, 56, 80 };
var areAllNumbersGreaterThan10 = numbers.All(x => x > 20);

### 判断集合中是否存在元素
numbers.Any();
~~~

#### LINQ操作符之Contains
Contains操作符检查指定的元素是否存在于集合中，并返回一个布尔值。
Contains扩展方法有两个重载，第一个重载方法需要传入一个在集合中检索的值，第二个重载方法需要传入一个附加的IEqualityComparer参数来进行自定义的相等性比较器。

Contains扩展方法只比较对象的引用，而不是对象的实际值。因此，为了比较student对象的值，需要通过实现IEqualityComparer接口创建一个类，该接口比较两个student对象的值并返回布尔值。

#### LINQ操作符之Aggregate
Aggregate具有三种重载
~~~C#
public static TSource Aggregate<TSource>(this IEnumerable<TSource> source, 
                                         Func<TSource, TSource, TSource> func);

public static TAccumulate Aggregate<TSource, TAccumulate>(this IEnumerable<TSource> source, 
                                         TAccumulate seed, 
                                         Func<TAccumulate, TSource, TAccumulate> func);

public static TResult Aggregate<TSource, TAccumulate, TResult>(this IEnumerable<TSource> source, 
                                         TAccumulate seed, 
                                         Func<TAccumulate, TSource, TAccumulate> func, 
                                         Func<TAccumulate, TResult> resultSelector);
~~~

#### LINQ操作符之Average
LINQ的Average方法用于计算应用该方法的集合中的数值的平均值。这个Average方法可以返回可为空或不可为空的十进制、浮点或双精度值。
~~~C#
var numbers = new[] { 20, 30, 50, 60 };
var avg = numbers.Average();
~~~

#### LINQ操作符之Count
Count操作符用以返回集合中元素的数量或满足给定条件的元素的数量

#### LINQ操作符之Max、Min、Sum
LINQ的Max()方法用以返回集合中最大的数字元素。
Min操作符与Max操作符类似，只是Min用以返回集合中最小的数字元素。
LINQ的Max()方法用以计算集合中数值项的和。

#### LINQ操作符之元素操作符
操作符|描述
-----|-----
ElementAt|返回集合中指定索引处的元素。
ElementAtOrDefault|返回集合中指定索引处的元素，如果索引超出范围则返回默认值。
First|返回集合的第一个元素，或满足条件的第一个元素。
FirstOrDefault|返回集合的第一个元素，或满足条件的第一个元素。如果索引超出范围，返回默认值。
Last|返回集合的最后一个元素，或满足条件的最后一个元素
LastOrDefault|返回集合的最后一个元素，或满足条件的最后一个元素。如果不存在这样的元素，则返回默认值。
Single|返回集合中的唯一元素，或满足条件的唯一元素。
SingleOrDefault|返回集合中的唯一元素，或满足条件的唯一元素。如果不存在这样的元素或集合不包含恰好一个元素，则返回默认值。


#### LINQ操作符之集合操作符
操作符|描述
-----|----
Distinct|去掉集合的重复项
Except|返回两个集合的不同，第一个集合的元素不能出现在第二个集合中
Intersect|返回两个集合的交集，即元素同时出现在两个集合中
Union|返回两个序列中的唯一元素，这意味着出现在两个序列中的任何一个中的唯一元素

~~~C#
### 去重
var numbers = new[] { 1, 2, 2, 5, 5, 6, 9, 8, 9, 10 };
var result = numbers.Distinct();

### 返回两个集合的差值
var numbers = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
var numbers2 = new[] { 1, 2, 5, 11 };
var result = numbers.Except(numbers2);

### 返回两个集合的交集
var result = numbers.Intersect(numbers2);

### 返回两个集合的并集
var result = numbers.Union(numbers2);
~~~
#### LINQ操作符之切分操作符
操作符|描述
-----|-----
Take| 从序列的开头返回指定数量的连续元素
TakeWhile|只要满足指定的条件，就会返回序列的元素。
Skip|跳过序列中指定数量的元素，然后返回剩余的元素。
SkipWhile|只要满足指定的条件，就会跳过序列的元素。

`tips:`TakeWhile和Where的区别在于，TakeWhile是从前往后计算，如果遇到不满足Func条件，则提前退出。
#### LINQ操作符之连接操作符
Concat操作符用于连接两个序列，生成一个新序列。
#### LINQ操作符之等式操作符
SequenceEqual()用于判断两个序列中的内容是否一致。
#### LINQ操作符之生成操作符
操作符|描述
-----|------
DefaultEmpty|返回指定序列的元素；如果序列为空，则返回单一实例集合中的类型参数的默认值。
Empty|初始化集合
Range|生成指定范围内的整数的序列
Repeat|生成包含一个重复值的序列