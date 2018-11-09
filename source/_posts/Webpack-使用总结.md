---
title: Webpack 使用总结
date: 2018-11-09 09:41:36
tags: 技术帖
---

为了让 Tree Shaking 对第三方 npm 包生效，需要通过 mainFields 配置模块的入口描述
```javascript
module.exports = {
  resolve: {
    // 针对 Npm 中的第三方模块优先采用 jsnext:main 中指向的 ES6 模块化语法的文件
    mainFields: ['jsnext:main', 'browser', 'main']
  }
}
```
以上配置的含义是优先使用 jsnext:main 作为入口，如果不存在 jsnext:main 就采用 browser 或者 main 作为入口。 虽然并不是每个 Npm 中的第三方模块都会提供 ES6 模块化语法的代码，但对于提供了的不能放过，能优化的就优化。