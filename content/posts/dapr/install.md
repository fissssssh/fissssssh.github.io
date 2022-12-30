---
title: "安装 Dapr"
date: 2022-12-14T16:39:07+08:00
categories: development
tags:
  - dapr
---

## 前置条件

Dapr 可以脱离 Docker 运行，但不在本篇所讲范围内，**_本篇内容中的操作都是基于 Docker 安装完成并运行正常的情况下的操作_**

- Docker
  > Docker 安装官方文档描述十分清晰。 Windows 用户推荐安装带界面的 [Docker Desktop](https://docs.docker.com/desktop/install/windows-install/)（更符合 Windows 人的操作习惯吧），Linux 用户安装 [Docker Engine](https://docs.docker.com/engine/install/) 即可。

## 开始安装

Dapr 的安装分为两部分：

1. 安装 Dapr CLI
2. 安装 Dapr Runtime

有两种方式可以进行安装，在线安装方式会去 github 下载对应的资源，网络不好的同学可以使用离线安装的方式

### 在线安装

- 安装 Dapr CLI

  [Dapr 官网](https://docs.dapr.io/getting-started/install-dapr-cli/)有各个操作系统详细的安装方法，这里讲一下通用的二进制安装方法：

  1. 首先去 Dapr CLI 的[发布页](https://github.com/dapr/cli/releases/tag/v1.9.1)（目前最新是 1.9.1 版本）下载对应操作系统版本的压缩包，命名格式为`dapr_<os_name>_<cpu_arch>.(tar.gz|zip)`，如果操作系统或者 CPU 架构没选对，则 Dapr CLI 无法正常运行
     > 通常来说 Windows 用户下载`dapr_windows_amd64.zip`，Linux 用户下载`dapr_linux_amd64.tar.gz`
  2. 解压到任意文件夹，并将该文件夹路径加入`PATH`环境变量
     > Linux 用户可直接创建软连接到`/usr/local/bin/dapr`， Windows 用户推荐将文件解压至`%USERPROFILE%\bin\`文件夹，并将`%USERPROFILE%\bin\`添加到`PATH`环境变量中，后续有其他的可执行文件也可以放入该文件夹，不用再动环境变量
  3. 打开控制台或者终端输入`dapr`，如果有相关的内容输出则安装成功

- 安装 Dapr Runtime

  安装 Dapr Runtime 也叫初始化 Dapr。

  本地 Dapr 环境的初始化很简单， 只需要`dapr init`即可

### 离线安装

1. 下载[Dapr Install-Bundle](https://github.com/dapr/installer-bundle/releases/tag/v1.9.5)，下载文件的选择同 Dapr CLI
2. 解压，并将 daprbundle/dapr（dapr.exe）移动到可执行文件夹下（参考[在线安装](#在线安装)中安装 Dapr CLI 的第 2 步）
3. 移动到 daprbundle 文件夹下执行 `dapr init --from-dir .`
4. 离线安装不会安装 zipkin 和 redis，可以使用下列指令进行安装

   ```shell
   $ docker run --name "dapr_zipkin" --restart always -d -p 9411:9411 openzipkin/zipkin
   $ docker run --name "dapr_redis" --restart always -d -p 6379:6379 redislabs/rejson
   ```

### 验证

打开控制台或终端输入`dapr version`，可以看到已经安装的版本

```shell
$ dapr version
CLI version: 1.9.1
Runtime version: 1.9.5
```

如果出现上述内容，恭喜您，您已经成功安装本地 dapr 环境！
