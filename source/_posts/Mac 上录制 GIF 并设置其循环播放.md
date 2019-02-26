---
title: Mac 上录制 GIF 并设置其循环播放
date: 2018-09-06 09:49:20
tags: 工具
categories: 工具
---
> 经常会被问到，如何在 macOS 上制作 Gif。这里给大家介绍一下免费的录屏软件和将其制作为循环播放的方法。

<!--more-->

## 使用 Giphy Capture 制作 GIF

[GIPHY Capture](https://itunes.apple.com/us/app/giphy-capture.-the-gif-maker/id668208984?mt=12) 是由一个很火的 Gif 分享网站 GIPHY 开发的，可以直接上传至 giphy.com，也可以保存到本地。
![](https://i.loli.net/2018/11/20/5bf3a7ba2b3f3.png)
如图，可以在紫条中控制输出的片段，在 Loop Type 中控制播放的顺序，在 Pixel Size 中控制输出的尺寸大小，在 Frame Rate 中控制输出的帧率。
下面 save as... 可以保存在本地，或上传到 giphy.com
**我这里保存为 rcc.gif, 路径为 ~/Documents/tools/gif/rcc.gif**

## 使用 gifsicle 设置 GIF 循环播放
下载 [gifsicle](http://www.lcdf.org/gifsicle/), 并解压
我解压到该目录 ~/Documents/tools/gifsicle-1.91
> 进入该目录，执行 `./configure --disable-gifview --disable-gifdiff`

![](https://i.loli.net/2018/11/20/5bf3a7b79a80a.png)
> 执行 `make`

![](https://i.loli.net/2018/11/20/5bf3a7a3b9151.png)
> 进入 `src` 目录，将原 GIf 图片(路径 ~/Documents/tools/gif/rcc.gif)设置为循环播放

![](https://i.loli.net/2018/11/20/5bf3a7a3f1ccd.png)
> 将 GIF 图片设置为循环播放已完成

## 参考
[MAC制作GIF及设置GIF循环播放](https://www.jianshu.com/p/32b369b4691f)