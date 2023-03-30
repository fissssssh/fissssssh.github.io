---
title: "Newtonsoft.Json 小技巧"
date: 2023-03-12T22:27:51+08:00
tags:
  - CSharp
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

## 从流进行反序列化

有些情况下我们需要反序列化来自网络请求或者文件中 JSON，它们通常是以流的形式持有，反序列化先将其读取为字符串，当 JSON 内容过大的时候会产生一个巨大的字符串对象，如果此操作比较频繁则会对性能产生比较大的影响

Newtonsoft.JSON 并没有直接提供从流反序列化的方法，但我们可以这样操作：

```csharp
using var stream = ReadStreamFromFileOrNetwork();
using var sr = new StreamReader(stream);

var serializer = JsonSerializer.Create();
return serializer.Deserialize<List<Root>>(new JsonTextReader(sr));
```

我使用以下代码对**直接从流反序列化**和**先转为字符串再反序列化**做了基准测试进行比较

```csharp
[MemoryDiagnoser, ShortRunJob]
public class DeserializeFromStreamDirectlyVsDeserializeAfterConvertStreamToString
{
    private byte[] data = null!;

    [GlobalSetup]
    public void Setup()
    {
        data = File.ReadAllBytes("large-file.json");
    }

    [Benchmark]
    public List<Root>? DeserializeFromStreamDirectly()
    {
        using var ms = new MemoryStream(data);
        using var sr = new StreamReader(ms);
        var serializer = JsonSerializer.Create();
        return serializer.Deserialize<List<Root>>(new JsonTextReader(sr));
    }

    [Benchmark]
    public List<Root>? DeserializeAfterConvertStreamToString()
    {
        using var ms = new MemoryStream(data);
        using var sr = new StreamReader(ms);
        var json = sr.ReadToEnd();
        return JsonConvert.DeserializeObject<List<Root>>(json);
    }
}
```

> 测试 JSON 文件来自 [json-iterator/test-data/large-file.json](https://github.com/json-iterator/test-data/blob/master/large-file.json)

测试结果如下：

```ini

BenchmarkDotNet=v0.13.5, OS=Windows 11 (10.0.22621.1265/22H2/2022Update/SunValley2)
AMD Ryzen 9 5900X, 1 CPU, 24 logical and 12 physical cores
.NET SDK=7.0.100
  [Host]   : .NET 7.0.0 (7.0.22.51805), X64 RyuJIT AVX2
  ShortRun : .NET 7.0.0 (7.0.22.51805), X64 RyuJIT AVX2

Job=ShortRun  IterationCount=3  LaunchCount=1
WarmupCount=3

```

| Method                                |     Mean |    Error |  StdDev |      Gen0 |      Gen1 |      Gen2 | Allocated |
| ------------------------------------- | -------: | -------: | ------: | --------: | --------: | --------: | --------: |
| DeserializeFromStreamDirectly         | 119.4 ms |  9.73 ms | 0.53 ms | 4400.0000 | 2000.0000 |  800.0000 |  60.35 MB |
| DeserializeAfterConvertStreamToString | 143.5 ms | 45.52 ms | 2.50 ms | 8250.0000 | 6000.0000 | 1750.0000 | 160.15 MB |

可以看出时间上面二者没有太大区别，但是在内存分配上足足省了 100MB!
