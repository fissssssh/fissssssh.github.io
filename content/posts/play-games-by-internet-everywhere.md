---
title: "Play Games by Internet Everywhere"
date: 2022-07-19T11:07:48+08:00
draft: true
---

# 前置条件

- 一台搭载 NVIDIA 显卡的 PC 游戏主机，并且已经安装 NVIDIA Experience（以下简称 **GamePC**）
- 一台具有公网 IP 的 VPS（以下简称 **VPS**）
- 任意安装了 moonlight 的普通电脑（性能不要太差）

# 原理

通过 NVIDIA Shield 将游戏流式传输到网络

# 操作

1. 在 **GamePC** 上打开 NVIDIA Experience，点击设置（齿轮图标），找到 SHIELD 并开启，将你想要玩的游戏添加到其中
2. 在 **GamePC** 上安装 monnlight 的 internet hosting tool
3. 在 **VPS** 上安装 frp server
4. 在 **GamePC** 上安装 frp client 并启动
5. 放行 **VPS** 的 frp server 端口 以及 moonlight internet hosting tool 端口
6. 在 安装 moonlight client 的机器上打开它，点击添加主机，输入 **VPS** 的地址
7. 点击该主机，首次使用须先进行配对操作，你的 moonlight client 会给你一个验证码，你需要在 **GamePC** 上输入该验证码进行确认
8. 开始游玩

# 已知问题

- Win10/11 在 UAC 界面光标不可见的问题（ Nvidia 的问题）
