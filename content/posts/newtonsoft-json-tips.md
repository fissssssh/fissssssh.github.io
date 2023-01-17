---
title: "Newtonsoft.Json 小技巧"
date: 2023-01-17T17:06:58+08:00
tags: c#
draft: false
---

`Newtonsoft.Json` 是一个非常受欢迎的 .NET JSON 框架

一般来说大部分用户用到的方法主要就是 `JsonConvert.SerializeObject` 和 `JsonConvert.DeserializeObject` 方法

前者用于将 .NET 对象 序列化为 JSON 字符串，后者则是将 JSON 字符串反序列化为 .NET 对象

下面我将讲述一些你可能用过或者没用过的一些小技巧

## 填充对象

我现在有一个 `{"name":"fissssssh"}` JSON 对象，当将它转为`Dictionary<string,string>`后，可以通过键名`name`获取属性值

然而这个 JSON 现在变成 `{"Name":"fissssssh"}`, 无法再通过键名 `name` 获取属性值

`Dictionary` 是支持替换键名比较器的，但是 `JsonConvert.DeserializeObject` 只会调用其无参构造函数

我们可以手动将 `Dictionary` 创建出来, 然后通过 `JsonConvert.PopulateObject` 方法将 JSON 字符串序列化并填充至指定对象，代码如下

```csharp {hl_lines=[6]}
var json = "{\"Name\":\"fissssssh\"}";

// 创建一个键名忽略大小写的字典对象
var dict = new Dictionary<string,string>(StringComparer.InvariantCultureIgnoreCase);

JsonConvert.PopulateObject(json, dict);

// 使用 name 获取 JSON 中的 Name 属性值
var name = dict["name"];
```
