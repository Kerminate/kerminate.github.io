---
title: 使网页中 div 元素居中(垂直居中+水平居中)的方法
date: 2018-03-02 21:49:15
tags: 前端
---

这边使用三种方法处理元素居中的问题。
放上示范的html代码：

```html
<body>
  <div class="main">
    <h1>居中</h1>
  </div>
</body>
```
<!--more-->

1. 方法一
`div` 使用绝对布局，设置 `margin: auto`;并设置 `top、left、right、bottom` 的值相等即可，不一定要都是0
```css
.main {
  text-align: center; /*让div内部文字居中*/
  width: 300px;
  height: 100px;
  margin: auto;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
```

2. 方法二
仍然是绝对布局，让 `left` 和 `top` 都是50%，这在水平方向上让 `div` 的最左与屏幕的最左相距50%，垂直方向上一样，所以再用 `transform` 向左（上）平移它自己宽度（高度）的50%，也就达到居中效果了
```css
.main {
  text-align: center;
  background-color: #fff;
  border-radius: 20px;
  width: 300px;
  height: 350px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
}
```

3. 方法三
使用 `flex布局`
```css
body {
  display: flex;
  justify-content: center;
  align-items: center;
}
```