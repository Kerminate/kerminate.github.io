---
title: nvm,nrm,yrm 的使用
date: 2019-10-15 17:08:18
tags: 工具
categories: 工具
---

最近把电脑上的 Node 版本升级到了 10.16.3 的稳定版本，但是后面在项目开发的时候，发现有些项目重装依赖后打包失败，最后发现是因为 Node 版本的问题。但我又不想降级到 Node8.0，这时候怎么办？

我们就需要借助于 nvm 这个 Node 版本管理工具了，同时我还会介绍下 nrm 和 yrm 这两个管理源的工具。
<!--more-->

## NVM
### 安装
```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.0/install.sh | bash
```

如果出现了 `nvm: command not found` 的错误，可以按照官方教程解决问题

![](https://i.loli.net/2019/10/15/7coPODMBQSf3ltH.png)

### 查看所有命令
```shell
nvm -h
```

经常会出现忘记一些 shell 命令的情况，这时候只有输出 nvm 所有命令，就知道各个命令的作用了。

### 查看已安装的 Node 版本
```shell
nvm list
```

![](https://i.loli.net/2019/10/15/TH3Vv4YLnomMqpd.png)

上图是我电脑目前已经安装的 Node 各版本情况，让我们一起分析下
- 绿色箭头所指向的就是当前使用的版本
- node 和 stable 指的是当前的稳定版本
- iojs iojs的最新稳定版本
- lts/* node lts 系列最新的稳定版本
- lts/argon,lts/boron,lts/carbon 分别指 lts 的三个大的版本的最新版本
- N/A 表示该版本没有装

我这里没有装 lts/argon 版本，那么可以直接使用以下命令安装

### 安装指定别名的 Node
```shell
nvm install lts/argon
```

也可以通过版本号来直接安装，先查询目前 Node 的所有版本
### 查询所有 Node 版本
```shell
nvm ls-remote
```

然后会有很长一串列表，挑选一个版本号来安装，比如 Node9
### 安装指定版本号的 Node
```shell
nvm install v9.0.0
```

现在安装了很多 Node 版本，我们需要学会如何切换 Node 版本

### 切换 Node 版本
```shell
nvm use v8.16.1
```

这样就将版本切换成了 `v8.16.1`，再次打印本地版本，发现已经切换成功

![](https://i.loli.net/2019/10/15/xRyfDKGQbuAgstc.png)

还可以通过切换别名来切换版本
```shell
nvm use stable
```

但是这样切换版本只是临时切换，重新打开一个 terminal 窗口，版本就会恢复到默认版本，我们需要手动修改默认版本

### 修改 Node 默认版本
```shell
nvm alias default v8.16.1
```

## NRM
### 安装
```shell
npm install -g nrm
```

### 查看当前所有 npm 源
```
nrm ls
```

![](https://i.loli.net/2019/10/15/n5NF7XDqbZpBAuR.png)

### 切换 npm 源
```
npm use taobao
```

### 新增 npm 源
```
npm add <源名称> <源地址>
```

## YRM
### 安装
```
yarn global add yrm
```

### 查看当前所有 yarn 源
```
yrm ls
```

![](https://i.loli.net/2019/10/15/91ObmcNrf7uy4oD.png)

### 切换 yrm 源
```
yrm use taobao
```

### 新增 yrm 源
```
yrm add <源名称> <源地址>
```