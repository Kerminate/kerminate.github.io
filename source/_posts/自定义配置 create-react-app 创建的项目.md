---
title: 自定义配置 create-react-app 创建的项目
date: 2018-07-17 21:02:15
tags: 技术贴
---

## 配置 standard 风格的eslint
项目使用 create-react-app 搭建
```
npx create-react-app forum-fe
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
/scripts/
/dist/
/*.js
/test/unit/coverage/
```

此时打开项目中文件，standard 的 eslint 效果已经生效
![](http://or7tt6rug.bkt.clouddn.com/react-eslint3%281%29.png)
Ctrl + S 保存后，格式化成功!
![](http://or7tt6rug.bkt.clouddn.com/react-eslint4%281%29.png)

## 配置 antd 的按需加载
安装 react-app-rewired (一个对 create-react-app 进行自定义配置的社区解决方案)
```
npm install --save-dev react-app-rewired
```
修改 package.json 里面的启动项
```json
"scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test --env=jsdom",
    "eject": "react-scripts eject"
  },
```
在项目根目录下创建一个 `config-overrides.js` 文件，对 webpack 配置进行扩展
```
/* config-overrides.js */

module.exports = function override (config, env) {
  // do stuff with the webpack config...
  return config
}
```
使用 `babel-plugin-import` 实现 antd 的按需加载
```
npm install --save-dev babel-plugin-import
```
再修改 `config-overrides.js`
```
/* config-overrides.js */
const { injectBabelPlugin } = require('react-app-rewired')
module.exports = function override (config, env) {
    config = injectBabelPlugin(['import', { libraryName: 'antd', style: 'css'}], config)
    return config
}
```
## 自定义 ant-design 的主题色
安装 less 及其相关配置
```
npm install --save-dev less less-loader react-app-rewire-less
```
> 楼主亲测 当前版本的 less 无法自定义主题色，需要安装 2.7.3 版本

```
npm install --save-dev less@2.7.3
```
再修改 `config-overrides.js`
```
const { injectBabelPlugin } = require('react-app-rewired')
const rewireLess = require('react-app-rewire-less')
module.exports = function override (config, env) {
  config = injectBabelPlugin(['import', { libraryName: 'antd', style: true }], config)
  config = rewireLess.withLoaderOptions({
    modifyVars: { '@primary-color': '#1DA57A' }
  })(config, env)
  return config
}
```
在 modifyVars 里修改 antd 的主要样式变量