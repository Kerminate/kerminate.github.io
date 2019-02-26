---
title: 对比各类 ajax/http 库及最佳选择
date: 2018-12-15 14:41:55
tags: 技术帖
categories: 技术帖
---
在 WEB 开发中服务端和客户端的交互至关重要，客户端可以使用原生的 XMLHttpRequest 对象，服务端也可以使用 node 自带的 http 模块，而现在也有相当多的 第三方库来供我们使用。这里，我简要对一些流行的库或原生方法进行对比，并得出我们的最佳选择。

![](https://i.loli.net/2018/12/15/5c14e2873d36c.png)
主流的 ajax/http 方法库的兼容性和特性如上图所示。
<!--more-->

## XMLHttpRequest
JS 实现 AJAX 主要基于浏览器提供的 XMLHttpRequest（XHR）类，所有现代浏览器（IE7+、Firefox、Chrome、Safari 以及 Opera）均内建 XMLHttpRequest 对象。
然而，如果在项目中使用原生的 XMLHttpRequest，需要手动封装很多方法，使用起来相对麻烦。

## JQuery
JQuery 很好地封装了原生 AJAX 的代码，在 IE5，IE6上使用 ActiveX 对象来实现 Ajax，其他浏览器上都是使用 XMLHttpRequest。它在兼容性和易用性方面都做了很大的提高，让 AJAX 的调用变得非常简单。
但是随着前端浪潮的发展，它的缺点也暴露了出来。
- XMLHttpRequest 不符合关注分离的原则，配置和调用方式非常混乱
- 需要手动包一层 Promise，才能方便在现有项目开发
- JQuery 整个项目太大，单纯使用 ajax 却要引入整个 JQuery 非常的不合理

## Node HTTP
http 属于 node 原生的模块之一，这个模块现在主要用在启动 http server 的服务，它也有 request 和 get 的方法，能够完成基本的 http 请求交互，不过它提供的是相对底层的 API，且还是 callback 形式，需要自己手动封装一层 Promise。

## fetch
fetch 作为 HTML5 的新特性，主要用来代替 XMLHttpRequest。优缺点也很明显

### 优点
- 符合关注分离，没有将输入、输出和用事件来跟踪的状态混杂在一个对象里
- 更好更方便的写法
- 更加底层，提供的API丰富
- 支持 Promise，避免了 callback

### 缺点
- 在浏览器的兼容性不太好
- 只对网络请求报错，对400，500都当做成功的请求，需要封装去处理
- 默认不会带 cookie，需要添加配置项
- 原生不支持 abort，不支持超时控制
- 没有办法原生监测请求的进度

## fetch 扩展库
为了解决原生 fetch 使用环境的问题，fetch 也扩展了很多库

### fetch polyfill

为了解决 fetch 各大浏览器的兼容性问题，fetch 官方推出了它的 polyfill 包。安装方法如下
```
npm install whatwg-fetch --save
```

### node-fetch
能够支持 fetch 在 Node 环境使用。安装方法如下
```
npm install node-fetch --save
```

### isomorphic-fetch
能同时使用在各大浏览器和 Node 环境。安装方法如下
```
npm install --save isomorphic-fetch es6-promise
```

## Superagent
Superagent 是一个基于 Promise 的轻量级渐进式 AJAX API，它既适用于 Node，也适用于所有现代浏览器。安装方式如下
```
npm install superagent --save
```

### 优点
- 同时支持 Node.js 和浏览器
- 它有一个插件生态，通过构建插件可以实现更多功能
- 可配置
- 支持显示上传和下载进度
- 支持分块传输编码

### 缺点
- 其 API 不符合任何标准


## axios
Axios 是一个基于 Promise 的 HTTP 库，可用在 Node.js 和浏览器上发起 HTTP 请求，支持所有现代浏览器，甚至包括 IE8+！在浏览器端它通过 xhr 方式创建 ajax 请求，在 node 环境 下，通过 http 库创建网络请求。安装方式如下
```
npm install axios --save
```

### 优点
- 同时支持 Node.js 和浏览器
- 支持 Promise API
- 可以配置或取消请求
- 可以设置响应超时
- 支持防止跨站点请求伪造（CSRF）攻击
- 可以拦截未执行的请求或响应
- 支持显示上传进度

## request
广泛用于 Node.js，但是不能使用在浏览器环境。安装方式如下
```
npm install request --save
```
它的缺点是不能基于 Promise，所以官方也出了不少 request 的扩展库
- request-promise (uses Bluebird Promises)
- request-promise-native (uses native Promises)
- request-promise-any (uses any-promise Promises)

## 数据统计
近2年，以上库 npm 的下载量对比图如下

![](https://i.loli.net/2018/12/15/5c14e26f88e38.png)

Github 上仓库的信息

![](https://i.loli.net/2018/12/15/5c14e27b45da8.png)

## 结论
我们分析了最受欢迎的一些 HTTP 库，问题在于我应该使用哪一个是取决于你的项目。没有正确或者错误的选择，只有为项目选择更适合的工具。在这里，
我个人推荐使用 axios 库，不仅在于它能同时支持 Node 和各大浏览器，还包括它强大的功能，以及简单易上手的特性，且已经广泛运用在了 React/Vue 的项目，增长趋势也相当明显。

## 参考
[AJAX/HTTP Library Comparison](https://www.javascriptstuff.com/ajax-libraries/)
[Comparing JavaScript HTTP Requests Libraries for 2019](https://blog.bitsrc.io/comparing-http-request-libraries-for-2019-7bedb1089c83)
[ajax、axios、fetch之间的详细区别以及优缺点](https://blog.csdn.net/twodogya/article/details/80223508)
[JS HTTP 请求库哪家强？Axios？Request？Superagent？](https://zhuanlan.zhihu.com/p/52235130)
[AJAX 之 XHR, jQuery, Fetch 的对比](https://zhuanlan.zhihu.com/p/24594294)