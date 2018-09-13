---
title: VSCode 插件与配置推荐
date: 2018-08-24 23:16:35
tags: 工具
---
在如今的前端时代，工具链已是开发中必不可少的一环。Sublime 的闭源，Atom 的启动性能较差，以及随着 VSCode 的逐渐成熟，已经成为前端开发中必不可少的神奇。
在 VSCode 中，通过配置以及丰富的插件，可以让你在敲代码时有事半功倍的效果。

**持续更新中...**
<!--more-->

# 配置
在 VSCode 中，自带支持 emmet 语法，但是默认是关闭的，在首选项中进行如下配置
```json
"emmet.triggerExpansionOnTab": true
```
在 jsx 中写标签不能快速补全，可配置
```json
"emmet.includeLanguages": {
  "javascript": "javascriptreact"
}
```
在编写某类语言代码时，配置 2 格 tab，粘贴和保存时自动修正格式
[] 内写语言，例如 html, css, less, typescript, javascriptreact等
```json
"[javascript]": {
  "editor.tabSize": 2,
  "editor.formatOnPaste": false,
  "editor.formatOnSave": false
},
```
使用 eslint 格式化代码, 我使用了 standard 风格的 eslint
```json
"eslint.autoFixOnSave": true,
"eslint.validate": [
  "javascript",
  "javascriptreact",
  "html",
  "vue"
],
"eslint.options": {
  "extensions": [
    ".js",
    ".vue"
  ]
},
"standard.autoFixOnSave": true,
```

# 插件
- TODO Highlight: TODO 高亮
- React.js code snippets: react 的 snippets
- React Standard Style code snippets: 因为我个人偏爱 standard 风格的 eslint，所以使用这款插件做 react 的 snippets 例如
rcc →	class component skeleton
ren → render method

![](http://or7tt6rug.bkt.clouddn.com/rcc-recycle.gif)

# 快捷键
Command + d 选中一个单词后，使用该快捷键再选中下一个相同单词，可以一直下去
Command + 鼠标左键  可显示多光标