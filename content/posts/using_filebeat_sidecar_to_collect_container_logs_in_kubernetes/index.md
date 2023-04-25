---
title: "使用Filebeat Sidecar在Kubernetes中收集容器日志"
date: 2023-04-25T17:26:40+08:00
tags:
  - ELK
  - K8s
---

当今时代，Kubernetes 已成为了容器编排的事实标准。但是，在处理容器日志时，需要一种高效而可靠的方式来将数据从容器发送到日志分析工具中进行分析。本文将介绍如何部署并使用 Filebeat 收集容器日志器，并将数据通过 Logstash 传输到日志聚合平台。本文的重点是 Filebeat 部署，有关 Logstash 的问题请阅读[《如何使用 Logstash 将日志写入阿里云 SLS 服务》](/posts/how-to-write-logs-to-aliyun-sls-using-logstash/)

# 简介

## Filebeat

Filebeat 是 Elastic 公司开源的、高性能的轻量级日志数据收集器，可以自动化地收集和汇总多种不同来源的日志数据，然后将其发送到 Elasticsearch 和 Logstash 等不同种类的日志分析工具中。Filebeat 的设计目标是为了方便用户在不同的场景中使用，同时确保运行的高效性和可靠性。

# 准备工作

首先我们需要一个可以产生日志的应用，假设该应用部署文件如下：

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:v1.0.0
          ports:
            - name: web
              containerPort: 80
```

该应用会往容器的 `/app/Logs/` 路径下输出多个日志文件，文件命名格式为 `log<suffix>.txt`

# 部署 Filebeat Sidecar

## 挂载日志目录

为了使 Filebeat 能够采集到 `my-app` 的日志，我们需要使用 `volume` 来挂载日志目录，这里选用 `emptyDir` 卷来挂载日志目录

> emptyDir 是 Kubernetes 中的一个 Volume 类型，它是一个临时的存储卷，可以用来在容器之间共享数据。emptyDir 存储卷所存储的数据只存在于该 Pod 的生命周期中，一旦 Pod 被删除，emptyDir 存储卷中的数据也会被清除。

修改上述部署文件如下：

```yml {hl_lines=["23-28"]}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:v1.0.0
          ports:
            - name: web
              containerPort: 80
          volumeMounts:
            - name: log-dir
              mountPath: /app/Logs
      volumes:
        - name: log-dir
          emptyDir: {}
```

在上述文件中，我们定义了一个名为 `log-dir` 的 `emptyDir` 卷，并将其挂载到了 `my-app` 容器的 `/app/Logs` 目录。这样程序输出的日志实际输出到了 `log-dir` 卷里面

## 部署 Filebeat

修改上述部署文件如下：

```yml {hl_lines=["0-13","39-46","50-55"]}
apiVersion: v1
kind: ConfigMap
metadata:
  name: log-config
data:
  filebeat.yml: |
    filebeat.inputs:
      - type: log
        paths:
          - /usr/share/filebeat/logs/log*
    output.logstash:
      hosts: ["logstash:5044"]
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:v1.0.0
          ports:
            - name: web
              containerPort: 80
          volumeMounts:
            - name: log-dir
              mountPath: /app/Logs
        - name: log
          image: docker.elastic.co/beats/filebeat:8.7.0
          volumeMounts:
            - name: log-dir
              mountPath: /usr/share/filebeat/logs
            - name: log-config
              mountPath: /usr/share/filebeat/filebeat.yml
              subPath: filebeat.yml
      volumes:
        - name: log-dir
          emptyDir: {}
        - name: log-config
          configMap:
            name: log-config
            items:
              - key: filebeat.yml
                path: filebeat.yml
```

在上述部署文件中，我们通过 ConfigMap 定义了 Filebeat 的配置：

```yml
filebeat.inputs:
    - type: log
    paths:
        - /usr/share/filebeat/logs/log*
output.logstash:
    hosts: ["logstash:5044"]
```

上述配置会让 Filebeat 采集 `/usr/share/filebeat/logs/` 路径下所有以 `log` 开头的日志文件，并输出到 Logstash 服务的 5044 端口

为了让 Filebeat 能够应用配置，我们创建了一个名为 `log-config` 的 `configMap` 卷，并选择其中的 `filebeat.yml` 字段挂载到 `/usr/share/filebeat/filebeat.yml`，通过此操作能使 Filebeat 正确使用 ConfigMap 中的配置

同时我们将 `log-dir` 也挂载到了 `/usr/share/filebeat/logs` 目录，该目录与配置文件中的目录对应，所以 Filebeat 能够正确采集此目录下的日志文件，即程序输出的日志文件

至此，Filebeat 已经能够正确采集程序输出的日志并发送给 Logstash
