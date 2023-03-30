---
title: "生成 HTTP 查询字符串"
date: 2023-02-04T13:57:23+08:00
tags:
  - CSharp
  - dotnet
---

## 什么是 HTTP 查询字符串

例如此 URL：`https://example.com:80/query?key1=value2&key2=value2`

对其拆解我们可以得到以下部分：

- `https`：协议
- `://`
- `example.com`：域名
- `:80`：端口
- `/query`：路径
- `?key1=value2&key2=value2`：参数（也称作**查询字符串**）

## 如何生成 HTTP 查询字符串

### 暴力拼接

略

### System.Web.HttpUtility

```csharp
var query = HttpUtility.ParseQueryString(string.Empty);
query["a+b"] = "a%b";
query["b"] = "2+1";
var queryString = query.ToString(); // a+b=a%25b&b=2%2b1
```

`HttpUtility.ParseQueryString(string.Empty)` 会返回一个空的`NameValueCollection`，你只需要往里面填充参数然后调用`ToString()`即可生成查询字符串

> 您不能使用 `new NameValueCollection()` 来达到同样的效果，因为 `HttpUtility.ParseQueryString(string)` 返回的实际是 `HttpQSCollection` 类型，该类型是 `NameValueCollection` 的派生类型且不对外公开，所以你也无法通过 `new` 关键字来创建 `HttpQSCollection` 类型

> 此方法生成的查询字符串只会转义 `value` 且**不包含**前导字符 `?`

### Microsoft.AspNetCore.Http.QueryString

> 此方法仅适用于 SDK 为 `Microsoft.NET.Sdk.Web` 的项目

```csharp
var queryString = QueryString.Create(new Dictionary<string, string?>
{
    ["a+b"] = "a%b",
    ["b"] = "2+1",
}).ToString(); // ?a%2Bb=a%25b&b=2%2B1
```

> 此方法生成的查询字符串会转义 `key` 和 `value` 且**包含**前导字符 `?`

> 除了调用 `ToString()` 也可以调用 `ToUriComponent()`，二者效果相同
