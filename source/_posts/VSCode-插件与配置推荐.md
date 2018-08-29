---
title: VSCode 插件与配置推荐
date: 2018-08-24 23:16:35
tags:
---
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