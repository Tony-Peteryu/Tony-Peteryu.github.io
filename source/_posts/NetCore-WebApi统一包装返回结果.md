---
title: NetCore-WebApi统一包装返回结果
date: 2022-10-27 14:09:29
tags: WebApi
categories: NetCore
---

##### 背景
在项目开发的过程中，在项目初期进行项目搭建的时候，应该尽可能让项目变得完善且更加好用，这一点涉及的方面很多，有许多在项目开发前期没有考虑周全导致后期维护成本大大增加，本文主要是对Net Core中返回结果统一包装的处理

<!--more-->

##### 统一结果类分装
使返回结果结构统一，首先需要一个统一的返回结果进行包装，在项目开发过程中会有不同的返回结果产生，虽然结构可能一致，但是数据类型可能存在不一致，因此可以常用泛型进行处理，此处不使用Object主要是避免装箱/拆箱的产生

###### 定义包装类
首先定义一个返回结果的包装类
~~~C#
public class ApiReturn<T>
{
    public StatusCode Status { get; set; } = StatusCode.Success;

    private string message;

    public string Message
    {
        get
        {
            return !string.IsNullOrEmpty(message) ? message : EnumHelper.GetEnumDescription(Status);
        }
        set
        {
            message = value;
        }
    }
    public T Data { get; set; }
    
}
public enum StatusCode
{
    [Description("请求成功")]
    Success = 200,
    [Description("请求失败")]
    Fail = 400,
    [Description("请求异常")]
    Error = 500
}

~~~
上图中的EnumHelper类主要用于获取定义枚举类的Description
~~~C#
public static class EnumHelper
{
    /// <summary>
    /// 获取当前枚举描述
    /// </summary>
    /// <param name="enumValue"></param>
    /// <returns></returns>
    public static string GetEnumDescription(Enum enumValue)
    {
        try
        {
            Type type = enumValue.GetType();
            MemberInfo[] memInfo = type.GetMember(enumValue.ToString());
            if (null != memInfo && memInfo.Length > 0)
            {
                object[] attrs = memInfo[0].GetCustomAttributes(typeof(DescriptionAttribute), false);
                if (null != attrs && attrs.Length > 0)
                    return ((DescriptionAttribute)attrs[0]).Description;
            }
            return enumValue.ToString();
        }
        catch (Exception)
        {
            return "Unknown";
        }
    }
}

~~~
这样就简单实现了返回结果的包装，下面简单使用一下
~~~C#
public ApiReturn<string> Test([FromBody]string username)
{
    return new ApiReturn<string> {
        Data = username                
    };
}
~~~
###### 优化
但是这种情况下我们每次都需要new 一个ApiReturn对象，我们可以将相应进行方法封装，简化我们在返回时需要传递的参数
~~~C#
public static ApiReturn<T> SuccessResult(T data) 
{
    return new ApiReturn<T> { Status = StatusCodes.Success, Data = data };
}

public static ApiReturn<T> FailResult(string message = null)
{
    return new ApiReturn<T> { Status = StatusCodes.Fail, message = message };
}
public static ApiReturn<T> ErrorResult(string message = null)
{
    return new ApiReturn<T> { Status = StatusCodes.Error, message = message };
}

//通用返回
public static ApiReturn<T> ResponseResult(T data, StatusCodes code, string message = null)
{
    return new ApiReturn<T> { Status = code, Message = message, Data = data };
}
~~~

简单使用
~~~C#
public ApiReturn<string> Test([FromBody]string username)
{
    return ApiReturn<string>.ResponseResult(username, StatusCodes.Fail);
}
~~~
###### 进一步优化
上面这样简化了对于返回的调用，但是这样每次还是需要手动输入ApiReturn，此时我们可以借助微软针对MVC的Controller的封装进一步得到一个思路，那就是定义一个基类的Controller，我们在Controller基类中，把常用的返回结果封装一些方法，这样在Controller的子类中呢就可以直接调用这些方法，而不需要关注如何去编写方法的返回类型了，
~~~C#
public class ApiBaseContoller : ControllerBase
{
    protected ApiReturn<T> SuccessResult<T>(T data)
    {
        return ApiReturn<T>.SuccessResult(data);
    }

    protected ApiReturn<T> FailResult<T>(string message = null)
    {
        return ApiReturn<T>.FailResult(message);
    }

    protected ApiReturn<T> ErrorResult<T>(string message = null)
    {
        return ApiReturn<T>.ErrorResult(message);
    }

    protected ApiReturn<T> ResponseResult<T>(T data, StatusCodes code, string message = null)
    {
        return ApiReturn<T>.ResponseResult(data, code, message);
    }
}
~~~
简单使用
~~~C#
public class TestController : ApiBaseContoller
{
    [HttpPost]
    public ApiReturn<string> Test([FromBody]string username)
    {
        return ResponseResult(username, StatusCodes.Fail);
    }
}
~~~
还有一种做法时直接在ApiReturn<T>中借助implicit自动完成隐式转换，这样可以简化在成功时候的返回
~~~C#
public static implicit operator ApiReturn<T>(T value)
{
    return new ApiReturn<T> { Data =value};
}
~~~
简单使用
~~~C#
public ApiReturn<string> Test([FromBody]string username)
{
    return username;
}
~~~
这种做法在初始理解上面可能会有点麻烦。