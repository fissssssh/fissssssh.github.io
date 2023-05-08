---
title: "在 ASP.NET Core 中使用 Serilog"
date: 2023-05-08T15:48:23+08:00
tags:
  - ASP.NET Core
  - CSharp
---

## 添加 Serilog 包引用

```bash
$ dotnet add package Serilog.AspNetCore
$ dotnet add package Serilog.Sinks.Async
```

通过上方指令我们添加了以下两个包：

- Serilog.AspNetCore

  Serilog.AspNetCore 是基于 Serilog 框架的一个扩展库，用于在 ASP.NET Core 应用程序中使用 Serilog 来记录日志。它提供了一些方便的方法来集成 Serilog 框架到 ASP.NET Core 应用程序中，并支持从请求上下文中自动提取一些默认的日志信息，例如 HTTP 请求方法、路径和响应状态码等。通过使用 Serilog.AspNetCore，您可以轻松地将高质量日志记录添加到您的 ASP.NET Core 应用程序中。

- Serilog.Sinks.Async

  Serilog.Sinks.Async 是 Serilog 库中的一个 sinks 扩展，用于异步地将日志事件写入目标存储。通过使用 Serilog.Sinks.Async，您可以避免在应用程序中引入 IO 操作的性能损失，从而提高应用程序的整体性能。

## 配置

Serilog 支持使用 appsettings.json 进行配置

在 appsettings.json 根节点下新增 `Serilog` 节点来配置 Serilog

```json
{
  "Serilog": {
    // 引用的 Serilog 的 sink，这里使用了 console、file、async 三种
    "Using": ["Serilog.Sinks.Console", "Serilog.Sinks.File", "Serilog.Sinks.Async"],
    // Serilog 对于不同的 logger 进行最小记录级别的配置
    "MinimumLevel": {
      // 默认为 Information
      "Default": "Information",
      "Override": {
        // 对于 Microsoft 和 System 命名空间下的 logger，级别为 Warning
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    // 日志输出配置
    "WriteTo": [
      // 将日志输出到 Console
      { "Name": "Console" },
      // 将日志输出到 File，同时使用 Async wrapper，可以让日志以异步方式写入文件
      {
        "Name": "Async", // 使用 Async wrapper 提高日志写入文件性能
        "Args": {
          "configure": [
            {
              "Name": "File",
              "Args": {
                "path": "Logs/log.txt",
                "rollingInterval": "Day", // 文件按天拆分
                "fileSizeLimitBytes": 104857600, // 最大文件大小 100M
                "rollOnFileSizeLimit": true, // 如果文件达到了允许的最大尺寸，也拆分
                "formatter": "Serilog.Formatting.Compact.CompactJsonFormatter, Serilog.Formatting.Compact" // 格式化器，紧凑的 Json 压缩
              }
            }
          ]
        }
      }
    ],
    // 日志数据的增强部分，这里的意思是将 LogContext 中的数据添加到日志数据里，LogContext 类似于一个堆栈，记录了一些自定义的环境数据，如 ASP.NET Core 的 HttpContext 信息，异常信息等等
    "Enrich": ["FromLogContext"]
  }
}
```

## 接管 ASP.NET Core 默认日志

使用 Serilog 接管 ASP.NET Core 的日志很简单，只需要在 Program.cs 中添加一行代码

```csharp
builder.Host.UseSerilog((context, services, configuration) => configuration.ReadFrom.Configuration(context.Configuration));
```

该代码会从 appsettings.json 中读取 Serilog 配置并使用 Serilog 日志提供程序，这得利于.NET Core 中优秀的日志接口设计

## 结束

至此，ASP.NET Core 使用 Serilog 记录日志就配置结束了

如果以后需要使用其他的日志记录提供程序，也是按照同样的步骤：

1. 添加对应的 Nuget 包引用
2. 编写对应的日志记录提供程序配置文件
3. 接管 ASP.NET Core 日志
