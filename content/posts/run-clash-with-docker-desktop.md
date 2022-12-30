---
title: "Run Clash With Docker Desktop"
date: 2022-07-09T12:21:08+08:00
draft: false
categoires: misc
tags:
  - clash
  - docker
---

## 创建`docker-compose.yml`

```yml
version: '3.9'
services:
  clash:
    container_name: core
    image: dreamacro/clash
    volumes:
      - ./config.yaml:/root/.config/clash/config.yaml:ro
    ports:
      - "7890:7890" # proxy port(change port to your config.yaml)
      - "9090:9090" # api port(change port to your config.yaml)
    restart: always
  yacd:
    container_name: web_ui
    image: haishanh/yacd
    ports:
      - "80:80" # change port which you want
    depends_on:
      - clash
    restart: always
```

## 启动

把你的clash配置文件 `config.yaml` 和 `docker-compose.yml` 放入同一个文件夹

然后执行 `docker compose up -d` 启动

打开 `http://localhost:<your_yacd_exposed_port>` 进入web UI 可以管理切换clash的节点和代理模式

## 设置系统代理
在 系统设置 -> 网络和Internet -> 代理 -> 手动设置代理 中填写clash暴露的http端口

点击使用代理服务器下方的切换开关即可开启和关闭系统代理

> 快捷切换可使用[ProxySwitch](https://github.com/fissssssh/ProxySwitch)