---
title: "如何正确在 ASP.NET Core 中返回流"
date: 2023-03-30T19:51:13+08:00
tags:
  - ASP.NET Core
  - CSharp
---

最近有位朋友说他在接口中使用 `File(Stream stream, string contentType)` 方法报错，报错内容是

```bash
2023-03-30 18:00:36.8882|ERROR|Microsoft.AspNetCore.Server.Kestrel|Connection id "OHMPHOM94BVEL", Request id "OHMPHOM94BVEL:00000004": An unhandled exception was thrown by the application.
```

大致推测是请求接口前或者接口后出的异常，也没有详细信息，于是要来了一份代码，代码（精简版）如下：

```c#
// ExcelDocument.cs

public class ExcelDocument
{
    public static Stream GetFileStream()
    {
        using var fs = new FileStream("appsettings.json", FileMode.Open, FileAccess.Read);
        var ms = new MemoryStream();
        fs.CopyTo(ms);
        return ms;
    }
}

// HomeController.cs

[HttpGet("GetFile")]
public IActionResult GetFile()
{
    var stream = ExcelDocument.GetFileStream();
    // 原本是application/vnd.ms-excel， 我这里写测试返回的appsettings.json就换了一下
    return File(stream, "application/json");
}
```

这段代码看着没啥问题，我试着请求了一下，果然有报错，我的报错如下：

```bash
fail: Microsoft.AspNetCore.Server.Kestrel[13]
      Connection id "0HMPH2LG4PQU0", Request id "0HMPH2LG4PQU0:00000004": An unhandled exception was thrown by the application.
      System.InvalidOperationException: Response Content-Length mismatch: too few bytes written (0 of 151).
```

注意错误中多了一行 `System.InvalidOperationException: Response Content-Length mismatch: too few bytes written (0 of 151).`，这很关键，说明之前的日志模板没有输出异常的详细信息

根据异常信息来看，响应头 `Content-Length` 设置为了 `151` ，但是 body 中没有写入数据（写入字节数为 `0`），大致能猜到是没读取到流的内容，为什么呢？创建的 `MemoryStream` 也没有关闭,答案就在于`Stream.CopyTo(Steam)`这个方法，让我们来看一下[官方文档](<https://learn.microsoft.com/en-us/dotnet/api/system.io.stream.copyto?f1url=%3FappId%3DDev16IDEF1%26l%3DZH-CN%26k%3Dk(System.IO.Stream.CopyTo)%3Bk(DevLang-csharp)%26rd%3Dtrue&view=net-7.0>)对这个函数的定义

```
Reads the bytes from the current stream and writes them to another stream. Both streams positions are advanced by the number of bytes copied.
```

恍然大悟！原来 `CopyTo` 会将流的 `Position` 移动，怪不得读取不到数据

解决这个问题也很简单，就是在 `CopyTo` 方法之后将流的指针移动到开始位置（流的最前端）

```c# {hl_lines=[10]}
// ExcelDocument.cs

public class ExcelDocument
{
    public static Stream GetFileStream()
    {
        using var fs = new FileStream("appsettings.json", FileMode.Open, FileAccess.Read);
        var ms = new MemoryStream();
        fs.CopyTo(ms);
        ms.Seek(0, SeekOrigin.Begin);
        return ms;
    }
}
```

```bash
$ curl http://localhost:5125/Home/GetFile
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

总结下来有两个问题：

1. 对 `Stream.CopyTo` 这个方法的不了解，未能在编码的时候注意到并避免（主要原因）
2. 日志记录模板没有配置显示异常详细信息而只显示了 `Message`，让我们没有足够的报错信息去排查错误
