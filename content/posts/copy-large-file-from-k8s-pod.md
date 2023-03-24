---
title: "从 K8s 的 Pod 中拷贝大文件"
date: 2023-03-24T12:48:02+08:00
tags:
  - K8s
  - Linux
---

从 K8s 的容器中拷贝文件可以使用`kubectl cp`命令，但是对于较大的文件（100M 以上）可能就不是很好使了，拷贝过程经常会出现`EOF`错误

对于较大的文件，我是这样操作的：

1. 压缩

   通过压缩可以减小大文件的体积

   ```bash
   tar -czvf largefile.tar.gz largefile
   ```

   > 此操作也可以用于打包整个文件夹，以便达到拷贝文件夹的目的

2. 分片

   对于体积较大的文件，即使压缩后仍然有好几百 M，这时使用`kubectl cp`也不一定能够拷贝下来，所以需要对其分割

   ```bash
   split -b 50M -d largefile.tar.gz largefile.tar.gz
   ```

   此操作会将`largefile.tar.gz`按照 50M/每个文件的大小分割为多个文件，`-d`选项则是使用数字来作为后缀（默认是英文字母）

   例如：`largefile.tar.gz`有 220M，则分割后会变为

   ```bash
   largefile.tar.gz00 # 50M
   largefile.tar.gz01 # 50M
   largefile.tar.gz02 # 50M
   largefile.tar.gz03 # 50M
   largefile.tar.gz04 # 20M
   ```

3. 拷贝

   使用`kubectl cp`命令拷贝分割后的文件

   ```bash
   kubectl cp <pod-name>:largefile.tar.gz00 largefile.tar.gz00
   kubectl cp <pod-name>:largefile.tar.gz00 largefile.tar.gz01
   kubectl cp <pod-name>:largefile.tar.gz00 largefile.tar.gz02
   kubectl cp <pod-name>:largefile.tar.gz00 largefile.tar.gz03
   kubectl cp <pod-name>:largefile.tar.gz00 largefile.tar.gz04
   ```

   > 对于多个容器的 Pod,可以使用`-c`选项指定具体的容器

4. 合并
   使用`cat`命令合并多个文件

   ```bash
   cat largefile.tar.gz* > largefile.tar.gz
   ```

   > 可以使用`md5sum`命令计算合并后的文件 hash 值与容器内的文件 hash 值是否一致来判断文件是否有缺失

5. 解压

   ```bash
   tar -xzvf largefile.tar.gz
   ```

   至此，我就将大文件从 Pod 中拷贝出来了
