---
title: "ASP.NET Core 中使用 Dapr 发布订阅"
date: 2022-12-14T22:25:34+08:00
categories: development
tags:
  - C#
  - ASP.NET Core
  - dapr
  - dotnet
---

## 定义 subpub 组件

我们使用 Dapr 初始化时安装的 redis 作为 pubsub 的实现

创建文件 `~/.dapr/components/pubsub.yaml` （Windows 用户为 `%USERPROFILE%\.dapr\components\pubsub.yaml` ），内容如下

> Dapr 初始化后 `~/.dapr/components` 文件夹会自动创建，里面有一个 `statestore.yaml` 的组件定义。如果没有该文件夹也不用担心，手动创建即可

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: pubsub
spec:
  type: pubsub.redis
  version: v1
  metadata:
    - name: redisHost
      value: localhost:6379
    - name: redisPassword
      value: ""
```

## 创建项目

1. 创建 ASP.NET Core WebAPI 项目

   ```shell
   $ dotnet new webapi --no-openapi --no-https
   ```

2. 安装 Dapr SDK

   - dotnet CLI

     ```shell
     $ dotnet add package Dapr.AspNetCore
     ```

   - 程序包管理器控制台

     ```pwsh
     Install-Package Dapr.AspNetCore
     ```

   > 也可以在 Visual Studio 的 Nuget 包管理器中搜索安装

3. 添加 Dapr 支持到 ASP.NET Core 框架

   - **Program.cs**

     ```diff
     - builder.Services.AddControllers();
     + builder.Services.AddControllers().AddDapr();


       app.MapControllers();
     + app.MapSubscribeHandler();
     ```

4. 添加消息处理程序

   - **PubSubController.cs**

     ```csharp
     using Dapr;
     using Microsoft.AspNetCore.Mvc;

     namespace pubsub
     {
         public class PubSubController : ControllerBase
         {
             private readonly ILogger<PubSubController> _logger;

             public PubSubController(ILogger<PubSubController> logger)
             {
                 _logger = logger;
             }

             [Topic("pubsub", "Test")]
             [HttpPost("test")]
             public async Task<IActionResult> SubscribeTest()
             {
                 var sr = new StreamReader(HttpContext.Request.Body);
                 var body = await sr.ReadToEndAsync();
                 _logger.LogInformation("received cloud event from Test: {message}", body);
                 return Ok();
             }
         }
     }
     ```

     > 使用 Dapr.AspNetCore 提供的 `TopicAttribute` 可以方便的订阅主题并关联到 API 接口，必须在管道中配置 `app.MapSubscribeHandler()`， 否则即使标注了 `Topic` 特性程序也不会订阅

     > 订阅主题的接口必须返回 HTTP 200 OK 状态码，否则 dapr 可能会视为消息推送不成功而进行重新推送，dapr 的消息推送规则为 AtLeastOnce，即至少一次

## 测试订阅

1. 使用 dapr 启动程序

   ```shell
   $ dapr run --app-id pubsubtest --app-port 5265 -- dotnet run
   ```

   > `--app-port` 为应用程序的 http 端口，对于 ASP.NET Core 开发环境来说，一般在 `Properties/launchSettings.json` 中可以找到

   > `--app-id` 为应用程序 id，同一组应用程序类似在发布订阅中类似消息队列中的消费者组，如果有多个程序的 app-id 相同，则只有一个程序实例能够收到消息

2. 使用 dapr 模拟发布

   ```shell
   $ dapr publish -i pubsubtest -p pubsub -t Test -d '{"data":"this is a test message"}'
   Event published successfully
   ```

3. 观察第 1 步中程序的输出

   ```shell
   == APP == info: pubsub.PubSubController[0]
   == APP ==       received cloud event from Test: {"data":{"data":"this is a test message"},"datacontenttype":"application/json","id":"5d485363-1658-4771-8c25-a84615359dde","pubsubname":"pubsub","source":"pubsubtest","specversion":"1.0","time":"2022-12-15T01:17:10+08:00","topic":"Test","traceid":"00-363f742582302ed141396ce408dbacc0-4b6f5751d4039d75-01","traceparent":"00-363f742582302ed141396ce408dbacc0-4b6f5751d4039d75-01","tracestate":"","type":"com.dapr.event.sent"}
   ```

   可以发现收到的数据是一个 JSON 格式数据， 其中 `data` 字段正是我们实际发送的内容

   ```json
   {
     "data": { "data": "this is a test message" },
     "datacontenttype": "application/json",
     "id": "5d485363-1658-4771-8c25-a84615359dde",
     "pubsubname": "pubsub",
     "source": "pubsubtest",
     "specversion": "1.0",
     "time": "2022-12-15T01:17:10+08:00",
     "topic": "Test",
     "traceid": "00-363f742582302ed141396ce408dbacc0-4b6f5751d4039d75-01",
     "traceparent": "00-363f742582302ed141396ce408dbacc0-4b6f5751d4039d75-01",
     "tracestate": "",
     "type": "com.dapr.event.sent"
   }
   ```

   这种数据格式在 dapr 中称为 CloudEvent，它记录了发送的内容，类型，主题和其他元信息，大部分情况下我们只需要 `data` 字段的内容，可以通过在 ASP.NET Core 管道中添加如下代码来对 CloudEvent 消息解包：

   ```diff
   + app.UseCloudEvents();
     app.MapControllers();
     app.MapSubscribeHandler();
   ```

   修改代码后重新启动程序并发布测试消息，可以观察到以下输出

   ```shell
   == APP == info: pubsub.PubSubController[0]
   == APP ==       received cloud event from Test: {"data":"this is a test message"}
   ```

   > 实际情况中不需要自己从 `HttpContext.Request.Body` 中读取数据，可以通过 ASP.NET Core 的模型绑定来做

## 测试发布

1. 修改现有代码

   ```diff
     private readonly ILogger<PubSubController> _logger;
   + private readonly DaprClient _daprClient;

   - public PubSubController(ILogger<PubSubController> logger)
   + public PubSubController(ILogger<PubSubController> logger, DaprClient daprClient)
     {
         _logger = logger;
   +     _daprClient = daprClient;
     }

   + [HttpPost("pub")]
   + public async Task<IActionResult> PublishTest()
   + {
   +     await _daprClient.PublishEventAsync("pubsub", "Test", new { data = "this is a test publish message" });
   +     return Ok();
   + }
   ```

2. 调用发布消息的 API 接口
   ```shell
   $ curl -XPOST http://localhost:5265/pub
   ```
3. 观察程序输出

   ```shell
   == APP == info: pubsub.PubSubController[0]
   == APP ==       received cloud event from Test: {"data":"this is a test publish message"}
   ```

   调用发布 API 后，程序发布了消息到 pubsub 组件的 Test 组件，而我们的程序又订阅了这个主题，所以会打印出我们发布出去的消息
