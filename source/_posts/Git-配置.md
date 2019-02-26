---
title: Git 配置
date: 2018-08-22 13:56:09
tags: 技术帖
categories: 技术帖
---
## 如何管理多个 git 的 SSH 私钥
一般本地的 id_rsa 公钥时，默认为你 github 账号的公钥，如果在公司办公，需要在公司的 git 仓库里提交代码，就需要再生成一个公钥
```shell
# 生成一个新的公钥
ssh-keygen -t rsa -C "your_email@example.com” -f ~/.ssh/id_rsa_work
```
<!--more-->
进入 ~/.ssh 文件，新建一个 config 文件，加入以下代码
```shell
# 该文件用于配置私钥对应的服务器
# Default github user(your_email@outlook.com)
  Host github.com
  HostName github.com
  User git
  IdentityFile /Users/your_name/.ssh/id_rsa

# second user(your_email@company.com)
# 使用公司的域名和公司的git账户
  Host git.A.com
  HostName git.A.com
  User git
  IdentityFile /Users/your_name/.ssh/id_rsa_work
```
此时 ssh 下的目录结构应该如下
![](https://i.loli.net/2018/11/19/5bf28b1668ea9.png)