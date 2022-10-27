---
title: xunit-单元测试
date: 2022-10-27 14:18:24
tags: 测试
categories: NetCore
---

### 使用xunit进行单元测试
#### 为什么要进行自动化测试？

<!--more-->

- 手工测试的局限性
	- 手工测试不能覆盖所有代码路径。
	- 基本的功能性测试用例在每一轮测试中都不能少。由于工作量往往较大，属于重复性的、非智力性的和非创造性，并要求准确细致，使用机器比人类更有优势。
	- 许多死锁、资源冲突、多线程等有关的不正确 ，通过手工测试很难捕捉到。
	- 系统压力、性能测试，需要模拟大数据或大并发用户等各种测试场景，很难通过手工测试执行。
	- 系统可靠性测试，需要模拟系统长时间运行，以验证系统能否稳定运行，难以通过手工测试执行。
	- 如果有大量（几千）的测试用例，须要在短时间内（1天）完成，手工测试几乎不可能做到。

- 自动化测试的优点
	- 避免重复工作：对于功能已经完整和成熟的软件，每发布一个新的版本，其中大部分功能和界面都和上一个版本相似或完全相同，这部分功能特别适合于自动化测试，从而可以让测试达到测试每个特征的目的。
	- 提高测试效率：DCC版本的发布周期往往比较短，也就是开发周期只有短短的几个月，而在测试期间是每天/每2天都要发布一个版本供测试人员测试，一个系统的功能点有几千个上万个，人工测试是非常的耗时和繁琐，这样必然会使测试效率低下。
	- 保证每次测试的一致性和可重复性：由于每次自动化测试运行的脚本是相同的，所以每次执行的测试具有一致性，人是很难做到的。由于自动化测试的一致性，很容易发现被测软件的任何改变。
	- 更好的利用资源－－周未/晚上。理想的自动化测试能够按计划完全自动的运行，在开发人员和测试人员不可能实行三班倒的情况下， 自动化测试可以胜任这个任务， 完全可以在周末和晚上执行测试。这样充分的利用了公司的资源，也避免了开发和测试之间的等待。
	- 解决测试与开发之间的矛盾：通常在开发的末期，进入集成测试阶段，由于每发布一个版本的初期，测试系统的错误比较少，这时开发人员在等待测试人员测试出错误的时间。事实上在迭代周期很短的开发模式中，存在更多的矛盾，但自动化测试可以解决其中的主要矛盾。

#### 自动化测试的分类
- Unit Test 单元测试， 它可以测试一个类，或者一个类的某个功能，它具有很好的深度，但是对整个应用来说它不具备很好的覆盖面。
- Integration Test 集成测试，它没有单元测试那么细致，但是具有相对较好的测试覆盖面。例如它可以测试功能的组合，以及像数据库或文件系统这样的外部资源等。
- Subcutaneous Test 皮下测试，这种测试作用于UI层的下面一层，这也意味着它对整个应用来说有很好的覆盖率，但是深度欠佳。那一个MVC结构的应用来说，它就是针对刚好在Controller下面一层的测试，对于Web service来说，它就是对节点下面那层的测试。
- UI测试，它的测试覆盖面很广，直接从UI层面进行测试，但是深度欠佳。

#### 测试的三个阶段AAA
- Arrange，这里做一些先决的设定。例如创建对象实例，数据，输入等等。
- Act，在这里执行生产代码并返回结果。例如调用方法，或者设置属性（Properties）。
- Assert，在这里检查结果。测试通过或者失败。

#### xunit进行单元测试——Assert
Assert：Assert基于代码的返回值、对象的最终状态、事件是否发生等情况来评估测试的结果。Assert的结果可能是Pass或者Fail。如果所有的asserts都pass了，那么整个测试就pass了；如果有任何assert fail了，那么测试就fail了。
xUnit提供了以下类型的Assert：
- boolean：True/False
- String：相等/不等，是否为空，以..开始/结束，是否包含子字符串，匹配正则表达式
- 数值型：相等/不等，是否在某个范围内，浮点的精度
- Collection：内容是否相等，是否包含某个元素，是否包含满足某种条件(predicate)的元素，是否所有的元素都满足某个assert
- Raised events：Custom events，Framework events(例如：PropertyChanged)
- Object Type：是否是某种类型，是否某种类型或继承与某种类型

##### boolean Assert
~~~C#
//判断是否为true
Assert.Ture(bool condition)
//判断是否为false
Assert.False(bool condition)
~~~

##### string Assert
~~~C#
//判断是否相等
Assert.Equal(string expected, string actual)
//判断是否不等
Assert.NotEqual(string expected, string actual)
//判断是否为空
Assert.Null(string expected)
Assert.NotNull(string expected)
//以..开始/结束
Assert.StartsWith(string expectedStartString, string actualString)
Assert.EndsWith(string expectedStartString, string actualString)
//是否包含某个字符串
Assert.Contains(string expectedSubstring, string actualString)
//匹配正则表达式
Assert.Matches(Regex expectedRegex, string actualString)
~~~

##### 数值 Assert
~~~C#
//相等/不等
Assert.Equal<T>(T expected, T actual)
Assert.NotEqual<T>(T expected, T actual)
//是否在某个范围内
Assert.InRange<T>(T actual, T low, T high)
//浮点的精度
Assert.Equal(double expected, double actual, int precision)
~~~

##### Collection Assert
~~~C#
//是否包含某个元素
Assert.Contains<T>(T expected, IEnumerable<T> collection)
//是否包含满足某种条件(predicate)的元素
Assert.Contains<T>(IEnumerable<T> collection, Predicate<T> filter)
//是否所有的元素都满足某个assert
Assert.All<T>(IEnumerable<T> collection, Action<T> action)
~~~

##### Object Type Assert
~~~C#
//是否是某种类型
Assert.IsType<T>(object @object)
//是否某种类型或继承与某种类型
Assert.IsAssignableFrom<祖先类>(xx):
//判断两个引用是否指向不同的实例 
Assert.NotSame(a, b):
~~~

#####  异常 Assert
~~~C#
//Assert是否抛出了特定类型的异常.
Assert.Throws<ArgumentNullException>(...)
~~~

##### Assert Events 是否发生(Raised)
~~~C#
Assert.Raises<T>()
//断定PropertyChanged的Event是否被触发了.
Assert.PropertyChanged(..) 
~~~

#### xunit进行单元测试——Attribute
xunit中常用的框架属性
属性|描述
----|----
[fact]|标记测试方法，即类中的实际测试
[Trait]|在测试中设置任意元数据
[Fact(DisplayName ="name"))]|在属性上设置 DisplayName 参数修改显示名称。
[Fact(Skip="reason")]|在属性上设置 Skip 参数以暂时跳过测试。
[Theory]|当必须执行数据驱动的测试时，将使用此属性。在这种情况下，必须使用[理论]而不是[事实]属性
[InlineData]|此属性与 [Theory] 属性一起使用，以提供将对其执行参数化测试的数据子集。
[ClassData]|当传递给 [Theory] 测试的参数不是常量时，将使用此属性。
[MemberData]|测试多组参数