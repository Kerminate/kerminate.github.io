---
title: background 与 border 设置相同颜色却展示不一样
date: 2018-09-29 10:49:09
tags: 前端
categories: CSS
---
这两天在写一个需求的时候发现，当 background 与 border 设置相同时，颜色有时候展示会不一样

![](https://i.loli.net/2018/11/19/5bf2895d614ed.png)

两张图的 background 与 border 都设置颜色相同，但是左边这张图，展示保持一致，而右边这张图却不一致
<!--more-->
来看下两张图的代码
```css
.back1 { // 左边的图
  width: 103px;
  height: 26px;
  margin-right: 30px;
  background: rgba(255, 102, 51, 1);
  border: 1px solid rgba(255, 102, 51, 1);
  border-radius: 50px;
}
.back2 { // 右边的图
  width: 103px;
  height: 26px;
  background: rgba(255, 102, 51, 0.1);
  border: 1px solid rgba(255, 102, 51, 0.1);
  border-radius: 50px;
}
```
从上面可以看出左侧图的颜色设置的透明度为1，右侧图的颜色透明度为0.1
那我们推测难道是透明度引起的问题？

我们去查 css 中的 background，发现了一个 background-clip 的属性，它表示了元素背景所在的区域
> border-box(默认值): 元素的背景从 border 区域（包括 border）以内开始保留背景
> padding-box: 元素的背景从 padding 区域(包括 padding)以内开始保留
> content-box: 元素的背景从内容区域以内开始保留

从这里就可以看出问题了，background-clip 默认值是 border-box，所以 background 默认包含了 border，当两者的颜色存在透明度时，会有一个叠加效果，导致border 那边的颜色变得更深一点（叠加效果并不是 0.1 + 0.1 = 0.2）

只需要将 background-clip 设置为 padding-box 或 content-box 就能达到颜色展示一样的效果

![](https://i.loli.net/2018/11/19/5bf2896173fa1.png)