---
title: VSCode 插件与配置推荐
date: 2018-08-24 23:16:35
tags: 工具
categories: 工具
---
在如今的前端时代，工具链已是开发中必不可少的一环。Sublime 的闭源，Atom 的启动性能较差，以及随着 VSCode 的逐渐成熟，它已经成为前端开发中必不可少的神器。
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
}
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
"standard.autoFixOnSave": true
```
在项目中新增 .eslintrc 文件，里面的规则会覆盖在编辑器设置的规则，所以不用担心你个人的eslint配置会影响到项目
保证开启 eslint 的 autoFixOnSave，Ctrl + S 效果如下

![](https://i.loli.net/2018/11/20/5bf3a7c44ec80.gif)

使对应扩展名的文件使用特定的语言
```json
"files.associations": {
  "*.tpl": "html",
  "*.cjson": "jsonc",
  "*.wxss": "css",
  "*.wxs": "javascript",
  ".**rc": "json",
  ".sequelizerc": "javascript"
}
```

全局搜索中排除的文件
```json
"search.exclude": {
  "**/node_modules": true,
  "**/bower_components": true,
  "**/dist": true,
  "**/dest": true
}
```

# 插件
- **[TODO Highlight](https://marketplace.visualstudio.com/items?itemName=wayou.vscode-todo-highlight)**: TODO 高亮
- **[Reactjs code snippets](https://marketplace.visualstudio.com/items?itemName=xabikos.ReactSnippets)**: react 的 snippets
- **[React Standard Style code snippets](https://marketplace.visualstudio.com/items?itemName=TimonVS.ReactSnippetsStandard)**: 因为我个人偏爱 standard 风格的 eslint，所以使用这款插件做 react 的 snippets 例如
rcc →	class component skeleton
ren → render method

![](https://i.loli.net/2018/11/19/5bf28a0b2f973.gif)
- **[Document This](https://marketplace.visualstudio.com/items?itemName=joelday.docthis)**: 自动补全函数的 JSDoc 注释 (Ctrl + Alt + D)
- **[koroFileHeader](https://marketplace.visualstudio.com/items?itemName=OBKoro1.korofileheader)**: 自动补全函数的注释和文件头的注释
- **[Settings Sync](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync)**: 上传 vscode 配置，可在多台电脑上同步配置
- **[Auto Close Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-close-tag)**: 自动闭合标签
- **[GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)**: 很强大的看 git blame 和 git history 的插件
  ![](https://i.loli.net/2019/01/24/5c4957059664f.png)
- **[Bracket Pair Colorizer](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer)**: 让括号拥有独立的颜色，易于区分
- **[Indenticator](https://marketplace.visualstudio.com/items?itemName=SirTori.indenticator)**: 突出目前的缩进深度
- **[Project Manager](https://marketplace.visualstudio.com/items?itemName=alefragnani.project-manager)**: 在多个项目之间快速切换

# 快捷键
- Command + d 选中一个单词后，使用该快捷键再选中下一个相同单词，可以一直下去
- Command + u 取消选中下一个匹配单词
- Command + 鼠标左键  可显示多光标
- fn + f12 跳转到函数定义的位置
- Alt + → 按照单词移动光标
- Ctrl + Alt + → 按照单词大小写移动光标
- Alt + Commond + ↓ 出现多行光标
- Alt + Command + p 快速切换项目

# 自定义快捷键
- Command + m 控制 minimap 的出现于隐藏
- Command + \ 控制 sidebar 的出现于隐藏
- Command + , 打开设置
- Command + . 打开用户设置