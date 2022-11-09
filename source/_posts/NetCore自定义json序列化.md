---
title: NetCore自定义json序列化
date: 2022-11-09 14:33:11
tags: Json
categories: NetCore
---

## 背景
在进行项目开发的过程中，序列化与反序列化是会经常遇到的场景，在 `net core` 项目开发中，通常使用 `Newtonsoft.Json` 包中的进行序列化和反序列操作，这个包中的 `SerializeObject` (序列化)和 `DeserializeObject`（反序列化）能解决业务场景中90%以上的问题，但是开发过程中还是会遇到剩下的那20%的场景。下面对使用 `Newtonsoft.Json` 的自定义json序列化进行介绍。

<!--more-->

## 自定义Json序列化
在介绍自定义序列化前，先对 `Newtonsoft.Json` 中的几个自带特性进行介绍

### 特性介绍
~~~bash
[JsonIgnore]
该特性可以忽略序列化某个实体类字段

[JsonProperty("Font")]
用于设置序列化到json中的实际名称
~~~

### 源码
在自定义序列的时候我们需要实现一个converter继承自 `Newtonsoft.Json` 中的 `JsonConvert` ，并实现其中的方法，实现方法前我们需要了解需要实现方法的作用
~~~C#
public abstract class JsonConverter{

    protected JsonConverter();

    //是否开启自定义反序列化，值为true时，反序列化时会走ReadJson方法，值为false时，不走ReadJson方法，而是默认的反序列化
    public virtual bool CanRead { get; }

    //是否开启自定义序列化，值为true时，序列化时会走WriteJson方法，值为false时，不走WriteJson方法，而是默认的序列化
    public virtual bool CanWrite { get; }

    //能否对某类型进行序列化和反序列化，为true时，匹配该类型，值为false时，不匹配该类型
    public abstract bool CanConvert(Type objectType);

    //自定义反序列化方法
    public abstract object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer);

    //自定义序列化
    public abstract void WriteJson(JsonWriter writer, object value, JsonSerializer serializer);
}
~~~


通过了解上述类中方法的作用，我们在自定义序列化的时候需要 `CanRead` 属性为true，且实现 `CanConvert()`判断指定类型，同时需要实现 `WriteJson()`方法作为实际自定义输出，下面通过两个简单的例子进行介绍实际使用

### 示例

首先交代一下使用的类
~~~C#
class Model
{
    public Model(int id, string name, int age, DateTime createTime)
    {
        this.id = id;
        this.name = name;
        this.age = age;
        CreateTime = createTime;
    }

    public int id { get; set; }
    public string name { get; set; }
    public int age { get; set; }
    public DateTime CreateTime { get; set; }
}
~~~

初始化一个类
~~~C#
Model model = new Model(1, "tony", 22, DateTime.Now);
~~~

现在使用自带的序列化方法序列化结果如下

~~~json
{"id":1,"name":"tony","age":22,"CreateTime":"2022-11-09T14:40:04.1560759+08:00"}
~~~

可以看到此时 `CreateTime` 字段采用了默认的初始化,若此时我们需要将此字段序列化为时间戳的形式，我们需要如何处理呢？

实现一个converter继承自 `Newtonsoft.Json` 中的 `JsonConvert`

~~~C#
public class DateTimeConverter : JsonConverter
{
    public override bool CanRead { get { return false; } }

    public override bool CanWrite { get { return true; } }

    public override bool CanConvert(Type objectType)
    {
        return objectType == typeof(DateTime);
    }

    public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
    {
        throw new NotImplementedException();
    }

    public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
    {
        TimeSpan ts = ((DateTime)value) - new DateTime(1970, 1, 1, 0, 0, 0, 0);
        string convertTime = Convert.ToInt64(ts.TotalSeconds).ToString();
        writer.WriteValue(convertTime);
    }
}
~~~

另：此处仅介绍序列化内容，因此该类中 `ReadJson()` 方法未实现 

此时序列化时，将自定义的转换器作为参数传到序列化方法中
~~~C#
//Formatting.Indented用于格式化json
JsonConvert.SerializeObject(model,Formatting.Indented,new DateTimeConverter())
~~~

此时输出结果为

~~~json
{
  "id": 1,
  "name": "tony",
  "age": 22,
  "CreateTime": "1668004835"
}
~~~

可以看到，此时DateTime类型的字段已经被序列化为时间戳形式

若此时我们需要将id作为序列化的key，其他的属性作为value，该如何处理呢？

~~~C#
{
  "1": {
  "name": "tony",
  "age": 22,
  "CreateTime": "1668004861"
    }
}
~~~

另外实现一个转换器
~~~C#
class ModelConverter:JsonConverter
{
    public override bool CanRead { get { return false; } }
    public override bool CanWrite { get { return true; } }
    public override bool CanConvert(Type objectType)
    {
        return objectType == typeof(Model);
    }

    public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
    {
        throw new NotImplementedException();
    }

    public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
    {
        var model = value as Model;
        if (model == null) 
        {
            return;
        }
        writer.WriteStartObject();
        writer.WritePropertyName(model.id.ToString());
        //此处时间仍然转化为时间戳的形式
        writer.WriteRawValue(JsonConvert.SerializeObject(new { name=model.name,age=model.age,CreateTime=model.CreateTime}, Formatting.Indented, new DateTimeConverter()));
        writer.WriteEndObject();
    }
}
~~~

我们可以使用特性的方式处理，在Model上加上特性,该类型在序列化的时候会将该转换器加入到可用转换器中
~~~bash
[JsonConverter(typeof(ModelConverter))]
~~~

也可以用上面的方式实现自定义转换器的调用
~~~C#
JsonConvert.SerializeObject(model,Formatting.Indented,new ModelConverter())
~~~

实际开发中，可根据实际业务场景去定义所需转换器
