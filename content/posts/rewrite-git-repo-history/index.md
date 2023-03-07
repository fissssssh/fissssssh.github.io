---
title: "重写你的 Git 仓库历史"
date: 2023-03-07T12:18:15+08:00
draft: false
tags:
  - git
  - tools
---

在你使用 git 的过程中，你是否有过以下问题：

- 提交信息写错了
- 提交了不该提交的文件（例如某些调试过程产生的文件）
- ……

如果这些问题仅存在于本地，那么有很多种解决办法，比如使用 `commit --amend`， `rebase` 等指令，大不了就 `reset --mixed` 再重新 `commit`

但是如果你过了好几个甚至好几十个提交才发现这些问题，那么用上述方法就不太好解决了

## git-filter-repo

git-filter-repo 是一个用于重写历史提交的多功能工具，它和 `git filter-branch` 比较相似，但是其性能远远优于后者

git 官方也推荐使用 `git-filter-repo` 而不是 `git filter-branch`

## 安装

### 前置

- git >= 2.22.0 at a minimum; some features require git >= 2.24.0 or later
- python3 >= 3.5

### 包管理器安装

```shell
PACKAGE_TOOL install git-filter-repo
```

### 手动安装

下载最新的 [Release](https://github.com/newren/git-filter-repo/releases) 解压后将 `git-filter-repo` 文件复制到 `git --exec-path` 文件夹下

> 如果你的 `python3` 指令被命名为 `python` ，你还需要将 `git-filter-repo` 文件第一行中的 `python3` 改为 `python`（此问题困扰了大部分 Windows 用户）

## 例子

本例子使用以下仓库，历史提交信息如下

```shell
git log --stat

commit c13190a52fddf8db5d561659ae01aa32276f865e (HEAD -> master)
Author: fissssssh <fissssssh@email.com>
Date:   Tue Mar 7 12:42:20 2023 +0800

    modify a.txt

 a.txt | 1 +
 1 file changed, 1 insertion(+)

commit 258ba82351e483d12f3876764feb2451295c3d9d
Author: fissssssh <fissssssh@email.com>
Date:   Tue Mar 7 12:41:43 2023 +0800

    commit a.txt

 .vscode/launch.json |  26 ++++++++++++++++++++++++++
 .vscode/tasks.json  |  41 +++++++++++++++++++++++++++++++++++++++++
 a.txt               |   1 +
 bigfile             | Bin 0 -> 209715201 bytes
 4 files changed, 68 insertions(+)
```

### 删除错误提交的文件

上述仓库中有两个提交，我在初次提交的时候不小心将 `.vscode` 和 `bigfile` 提交了，并且后续还追加了一个提交

现在使用 `git-filter-repo` 来删除多余的文件，执行以下指令：

```shell
git filter-repo --path a.txt
```

> `--path` 选项表示只保留此文件，可以使用多个 `--path` 选项
>
> 类似的选项还有 `--path-glob`， `--path-regex`
>
> 使用 `--invert-paths` 参数可以将选中的文件变为反选

输出如下：

```shell
Aborting: Refusing to destructively overwrite repo history since
this does not look like a fresh clone.
  (expected at most one entry in the reflog for HEAD)
Please operate on a fresh clone instead.  If you want to proceed
anyway, use --force.
```

意思就是说你这个仓库不是刚克隆了，请使用新克隆的仓库执行操作，如果你硬要操作请加上 `--force` 选项。我们这里是使用的测试仓库，所以可以不管这条消息，重新执行以下指令：

```shell
git filter-repo --path a.txt --force
```

输出如下：

```shell
Parsed 2 commits
New history written in 0.01 seconds; now repacking/cleaning...
Repacking your repo and cleaning out old unneeded objects
HEAD 现在位于 a1e776f modify a.txt
枚举对象中: 6, 完成.
对象计数中: 100% (6/6), 完成.
使用 16 个线程进行压缩
压缩对象中: 100% (2/2), 完成.
写入对象中: 100% (6/6), 完成.
总共 6（差异 0），复用 0（差异 0），包复用 0
Completely finished after 0.05 seconds.
```

让我们重新看一下提交历史

```shell
git log --stat

commit a1e776f3d90e2023fc59e66815d8392151a7f612 (HEAD -> master)
Author: fissssssh <fissssssh@email.com>
Date:   Tue Mar 7 12:42:20 2023 +0800

    modify a.txt

 a.txt | 1 +
 1 file changed, 1 insertion(+)

commit 57d35f202e92be0fa34c8e771266c0c4e5a65f1a
Author: fissssssh <fissssssh@email.com>
Date:   Tue Mar 7 12:41:43 2023 +0800

    commit a.txt

 a.txt | 1 +
 1 file changed, 1 insertion(+)
```

> 细心的你会发现提交的 hash 值也变了，这是必然的

### 更改提交文件的文件名称

现在我发现 `a.txt` 这个名称不够好，我想换成 `b.txt`，只需执行以下指令：

```shell
git filter-repo --path-rename a.txt:b.txt
```

输出如下：

```shell
Parsed 2 commits
New history written in 0.02 seconds; now repacking/cleaning...
Repacking your repo and cleaning out old unneeded objects
HEAD 现在位于 435a640 modify a.txt
枚举对象中: 6, 完成.
对象计数中: 100% (6/6), 完成.
使用 16 个线程进行压缩
压缩对象中: 100% (2/2), 完成.
写入对象中: 100% (6/6), 完成.
总共 6（差异 0），复用 2（差异 0），包复用 0
Completely finished after 0.05 seconds.
```

让我们重新看一下提交历史

```shell
git log --stat

commit 435a640f4b8a42bfe06a5695a7ff4dd061f2d3b5 (HEAD -> master)
Author: fissssssh <fissssssh@email.com>
Date:   Tue Mar 7 12:42:20 2023 +0800

    modify a.txt

 b.txt | 1 +
 1 file changed, 1 insertion(+)

commit 000c714faced2aede9ae18c294ef4b9698cd52be
Author: fissssssh <fissssssh@email.com>
Date:   Tue Mar 7 12:41:43 2023 +0800

    commit a.txt

 b.txt | 1 +
 1 file changed, 1 insertion(+)
```

### 更改提交信息

文件名改了，提交信息还没变，让我们再来修改提交信息

修改提交信息需要一个替换文件，创建 **expressions.txt** 文件：

```shell
cat <<EOF > expressions.txt
a.txt==>b.txt
EOF
```

> 你可以用任何方式去创建文件

执行以下指令：

```shell
git filter-repo --replace-message expressions.txt
```

输出如下：

```shell
Parsed 2 commits
New history written in 0.02 seconds; now repacking/cleaning...
Repacking your repo and cleaning out old unneeded objects
HEAD 现在位于 49d497e modify b.txt
枚举对象中: 6, 完成.
对象计数中: 100% (6/6), 完成.
使用 16 个线程进行压缩
压缩对象中: 100% (2/2), 完成.
写入对象中: 100% (6/6), 完成.
总共 6（差异 0），复用 4（差异 0），包复用 0
Completely finished after 0.05 seconds.
```

让我们重新看一下提交历史：

```shell
git log --stat

commit 49d497e61b0370a6f6e23ed740eaa045be569361 (HEAD -> master)
Author: fissssssh <fissssssh@email.com>
Date:   Tue Mar 7 12:42:20 2023 +0800

    modify b.txt

 b.txt | 1 +
 1 file changed, 1 insertion(+)

commit baabe05301b0a38057538d47e3b7b899cd348935
Author: fissssssh <fissssssh@email.com>
Date:   Tue Mar 7 12:41:43 2023 +0800

    commit b.txt

 b.txt | 1 +
 1 file changed, 1 insertion(+)
```

## 更新至远端仓库

> 此步骤谨慎操作！谨慎操作！谨慎操作！

由于历史提交 hash 已变更，只能**强制推送**

```shell
git push -f origin master
```

## 最后

git-filter-repo 极大的简化了重写 git 提交历史的操作，这里介绍的只是九牛一毛，更多的操作可以参考[官方用户手册](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html)
