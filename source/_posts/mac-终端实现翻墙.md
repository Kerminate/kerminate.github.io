---
title: mac 终端实现翻墙
date: 2018-10-22 16:05:59
tags: 工具
categories: 工具
---
最近在使用 brew 安装一些软件时发现速度太慢，而且不少因为超时问题而无法安装。然后我在 google 上查了很多网友提供的方法，发现大多都已经用不了或者是有些细微处有所错误，故此重新写一篇 mac 终端实现翻墙的文章。
<!--more-->

## 翻墙工具 shadowsocks
现在一般人都是使用 VPN 或者 shadowsocks 的方法实现浏览器的翻墙。这边不再赘述，因为我在终端实现的翻墙走的是 shadowsocks 的代理，所以简单说下 ss 的安装。
https://github.com/shadowsocks/ShadowsocksX-NG 这是我用的 ss 的下载地址，点击进去，在 release 下面随便找个最近的包下载安装即可。

![](https://i.loli.net/2018/11/19/5bf28b1627855.png)

这个是 shadowsocks 的客户端，账号需要在 https://shadowsocks.com/ 上面注册。
这边有一个坑点，就是这个网站翻墙才能进去😂，所以你得先找个能翻墙的朋友帮你注册账号（一年大约100来块）

## 终端配置代理
在命令行输入执行以下两条指令
```
export http_proxy=http://127.0.0.1:1087
export https_proxy=http://127.0.0.1:1087
```
macOS 版的 SS 默认监控本地的HTTP端口是 1087，而 Windows 版本的则是 1080，如果改过默认端口，就使用你指定的端口
这样就完成终端翻墙了，当然我们每次翻墙都执行一次指令会比较麻烦，把指令写进 .bash_profile 方便以后操作。

## 终端代理写进 .bash_profile

```
vim ~/.bash_profile
```

进入 .bash_profile，在最后加上以下代码

```
function proxy_on(){
    export http_proxy=http://127.0.0.1:1087
    export https_proxy=http://127.0.0.1:1087
    echo -e "已开启代理"
}
function proxy_off(){
    unset http_proxy
    unset https_proxy
    echo -e "已关闭代理"
}
```

之后使该配置文件生效

```
source ~/.bash_profile
```

使用 proxy 前先查看下当前的 ip 地址
```
➜  ~ curl ip.cn
当前 IP：103.202.xxx.xx 来自：北京市
```
之后开启 proxy,再查看
```
➜  ~ proxy_on
已开启代理
➜  ~ curl ip.cn
当前 IP：103.88.xxx.xx 来自：日本 CatNetworks
```
不需要代理的时候再执行 proxy_off 关闭代理
```
➜  ~ proxy_off
已关闭代理
```
