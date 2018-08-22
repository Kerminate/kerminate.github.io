---
title: Git 配置
date: 2018-08-22 13:56:09
tags: 技术贴
---
当本地有多个 git 账号时，需要做如下配置
进入 ~/.ssh 文件，新建一个 config 文件，加入以下代码
```shell
# 该文件用于配置私钥对应的服务器
# Default github user(kepeilin@outlook.com)
  Host github.com
  HostName github.com
  User git
  IdentityFile /Users/kepeilin/.ssh/id_rsa

# second user(kepeilin@meituan.com)
# 建一个github别名，新建的帐号使用这个别名做克隆和更新
  Host git.A.com
  HostName git.A.com
  User git
  IdentityFile /Users/kepeilin/.ssh/id_rsa_work

# second user(kepeilin@meituan.com)
# 建一个github别名，新建的帐号使用这个别名做克隆和更新
  Host git.B.com
  HostName git.B.com
  User git
  IdentityFile /Users/kepeilin/.ssh/id_rsa_work
```