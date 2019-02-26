---
title: Koa 使用 Decorator 装饰路由
date: 2019-01-12 10:40:46
tags: Node
categories: Node
---

## 引言

在 koa 里使用路由一般都是这样写
```javascript
const router = requier('koa-router')
router.get('/users', async (ctx, next) => {
  ctx.body = 'users'
})
```

这样的路由设置显得不是那样的直观，而反观 Java 中 Spring 的写法，就感觉更简洁以及更优雅
```java
@Controller
public class UserController {
  @RequestMapping('/users')
  String Users() {
    return 'users';
  }
}
```
<!--more-->
而在 **[Nest](https://github.com/nestjs/nest)** 中其实已经实现了这样的写法, 因为 nest 是由 Typescript 编写的，TS 本身就支持 Decorator 的写法，再结合 **[reflect-metadata](https://github.com/rbuckton/reflect-metadata)** 来实现内部路由装饰器。

```javascript
@Controller('users')
class UsersController {
  @Get()
  getAllUsers(req, res, next) {}

  @Get('/:id')
  getUser(req, res, next) {}

  @Post()
  addUser(req, res, next) {}
}
```

在这里，我们使用原生的 javascript 来实现在 koa 中使用装饰器路由

## 什么是 Decorator
首先，我们先认识一下 Decorator，它是 es6 新增的函数。

```javascript
@testable
class MyTestableClass {
  // ...
}

function testable(target) {
  target.isTestable = true;
}

MyTestableClass.isTestable // true
```

上面的代码中，`@testable` 就是一个修饰器，它为 `MyTestableClass` 这个类加上了静态属性 `isTestable`

这是一个最简单的例子，一般情况下，我们是需要给类的实例添加方法，所以需要在 prototype 上添加属性，对 js 的原型链不太熟悉的同学可以先看下这篇文章 **[详解 Javascript 的原型链与继承(从 ES5 到 ES6)](http://kerminate.me/2018/02/09/%E8%AF%A6%E8%A7%A3%20Javascript%20%E7%9A%84%E5%8E%9F%E5%9E%8B%E9%93%BE%E4%B8%8E%E7%BB%A7%E6%89%BF-%E4%BB%8E%20ES5%20%E5%88%B0%20ES6/)** , 同时可以在修饰器上传入参数

### 类的修饰

```javascript
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;
  }
}

@testable(true)
class MyTestableClass {}

let obj = new MyTestableClass();
obj.isTestable // true
```

### 方法的修饰
在修饰方法的时候，修饰器函数一共接收三个参数，分别是 `target` (类的原型对象)，`name` (所要修饰的属性名)，`descriptor` (该属性的描述对象)
```javascript
class Boy {
  @speak('中文')
  show () {
    console.log('I can speak ' + this.language)
  }
}

function show (language) {
  return function (target, name, descriptor) {
    /**
    *  修饰器内参数的值如下
    *  function (
    *    target: Boy {}
    *    name: run
    *    descriptor: {
    *      value: [Function: run],
    *      enumerable: true,
    *      configurable: false,
    *      writable: true
    *    }
    *  )
    */
    target.language = language
    return descriptor
  }
}

const luke = new Boy()
luke.run() // I can speak 中文
```

这里我们就简要介绍一下 Decorator，如果想要深入了解，可以看下阮一峰老师的 **[ES6入门](http://es6.ruanyifeng.com/#docs/decorator)**

## babel 转码
> Babel 默认只转换新的 JavaScript 句法（syntax），而不转换新的 API ，比如 Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法（比如 Object.assign）都不会转码。Babel 默认不转码的 API 非常多，详细清单可以查看 [definitions.js](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js) 文件

而这里的 Decorator 也同样需要安装 babel 的 plugin。
如果你使用的是 babel6，那么需要安装 `transform-decorators-legacy`

```shell
npm install --save-dev babel-preset-env transform-decorators-legacy
```

.babelrc 可以这样配置
```json
{
  "presets": ["env"],
  "plugins": ["transform-decorators-legacy"]
}
```
如果你使用的是 babel7，那么需要安装 `@babel/plugin-proposal-decorators`

```shell
npm install --save-dev @babel/preset-env @babel/plugin-proposal-decorators
```

```json
{
  "presets": ["@babel/preset-env"],
  "plugins": ["@babel/plugin-proposal-decorators"]
}
```

之后在项目的启动文件头部，还需要引入 babel-register 和 polyfill
```javascript
// babel6
require('babel-register')()
require('babel-polyfill')

// babel7
require('@babel/register')()
require('@babel/polyfill')
```

## Koa 中使用装饰器路由
先看看传统的 koa 写法（不带装饰器）
```javascript
const koa = require('koa')
const Router = require('koa-router')

const app = new Koa()
const router = new Router()

router.get('/users', async (ctx, next) => {
  ctx.body = 'users'
})

app.use(router.routes()).use(router.allowedMethods())
```

首先，我们先实现 controller 类的装饰器
```javascript
const symbolPrefix = Symbol('prefix')
const controller = path => target => (target.prototype[symbolPrefix] = path)
```

这里用到了 ES6 新增的数据类型 Symbol，它表示独一无二的值。由于每一个 Symbol 值都是不相等的，这意味着 Symbol 值可以作为标识符，用于对象的属性名，就能保证不会出现同名的属性。

接着定义两个工具函数
- `formatPath` 能够格式化路由，使不带 '/' 的路由带上它
- `isArray` 输出数组

```javascript
const formatPath = path => path.startsWith('/') ? path : `/${path}`
const isArray = obj => Array.isArray(obj) ? obj : [obj]
```

设置路由的映射关系
```javascript
const routerMap = new Map()
/**
*  Map({
*   target: any // 类的实例
*   method: 'GET' // http method
*   path: '/list' // 接口路径
*  }, routerFunction)
*/
const router = conf => (target, name, descriptor) => {
  conf.path = formatPath(conf.path)
  routerMap.set({
    target,
    ...conf
  }, target[name])
}
```

设置完后，新建一个 Route 的类，能够接收 Koa 实例和 api 文件夹路径。
```javascript
const Router = require('koa-router')
const glob = require('glob')

class Route {
  constructor (app, apiPath) {
    this.app = app
    this.apiPath = apiPath
    this.router = new Router()
  }

  init () {
    // 将 api 文件接口全部同步载入
    glob.sync(path.resolve(this.apiPath, './*.js')).forEach(require)

    for (let [conf, controller] of routerMap) {
      const controllers = isArray(controller)
      const prefixPath = conf.target[symbolPrefix] ? formatPath(conf.target[symbolPrefix]) : ''
      const routerPath = prefixPath + conf.path
      this.router[conf.method](routerPath, ...controllers)
    }

    this.app.use(this.router.routes())
    this.app.use(this.router.allowedMethods())
  }
}
```

再分别使用之前的 router 函数给各种请求方法配好映射
```javascript
const methods = {}
;['head', 'options', 'get', 'post', 'put', 'del', 'patch', 'all'].forEach((key) => {
  methods[key] = function (path) {
    return router({
      methods: key,
      path
    })
  }
})
```

除了路由方法，我们还需要对中间件也进行一次装饰器修饰
```javascript
const middleware = (...mids) => {
  return (...args) => {
    const [target, name, descriptor] = args
    target[name] = isArray(target[name])
    target[name].unshift(...mids)
    return descriptor
  }
}
```

因为可能一个路由会调用多个中间件，所以使用 `...mids` 表示传进来的中间件，再将其一个个加到对应 routerMap 的前面。这样，每次发起请求时，都会先按传入的中间件顺序进行调用，之后才执行路由的方法

所以的步骤都完成之后，我们来看看最后使用装饰器的代码长什么样
```javascript
const { controller, get, post, middleware } = require('../lib/decorator')
const { log, validate } = require('../middlewares/user')

@controller('api/user')
class userController {
  @get('/info')
  @middleware(log, validate)
  async getUserInfo (ctx, next) {
    ctx.body = {
      username: 'Kerminate',
      job: 'programmer'
    }
  }
}

export default userController
```

`controller`, `get`, `post`, `middleware` 是我们之前对路由作了一层包装后暴露出来的接口，`log`, `validate` 则是两个中间件

完整项目可参见[源码](https://github.com/Kerminate/koa-decorators-router)

## 参考
- [ES6入门](http://es6.ruanyifeng.com/#docs/decorator)
- [用Decorator控制Koa路由](https://www.bougieblog.cn/article/Qk9VMjJHSUU.html)
- [koa-router封装](https://github.com/soraping/koa-decorators-router)