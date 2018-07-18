---
title: create-react-app 创建的项目配置 standard 风格的 eslint
date: 2018-07-17 21:02:15
tags: 技术贴
---
项目使用 create-react-app 搭建
```
npx create-react-app forum-fe
```
搭建完成后，进入项目目录，将配置文件暴露出来
```
npm run eject
```

之后安装 standard 相关依赖
```
npm install --save-dev eslint-config-standard eslint-config-standard-react eslint-plugin-standard eslint-plugin-promise eslint-plugin-import eslint-plugin-node eslint-plugin-react
```
<!--more-->

![](http://or7tt6rug.bkt.clouddn.com/react-eslint2%281%29.jpg)
这时候发现 eslint 和 eslint-plugin-react 版本过低，将这两个包按照要求升级
```
npm install --save-dev eslint@4.19.1 eslint-plugin-react@7.6.1
```

在项目根目录新建 .eslintrc 文件，加上
```json
{
  "extends": ["standard", "standard-react"]
}
```

在项目根目录新建 .eslintignore 文件，加上
```
/config/
/dist/
/*.js
/test/unit/coverage/
```

此时打开项目中文件，standard 的 eslint 效果已经生效
![](http://or7tt6rug.bkt.clouddn.com/react-eslint3%281%29.png)
Ctrl + S 保存后，格式化成功!
![](http://or7tt6rug.bkt.clouddn.com/react-eslint4%281%29.png)