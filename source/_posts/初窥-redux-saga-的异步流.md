---
title: 初窥 redux-saga 的异步流
date: 2018-09-21 14:02:51
tags: 技术帖
categories: 技术帖
---
redux-saga 和 redux-thunk 是最广为人知的2种 redux 的异步流处理方案。在认识 redux-saga 之前，我们先再看一看 redux-thunk。

在 redux 里，dispatch 出的 action 是一个对象，而 redux-thunk 中间件可以返回一个函数，函数处理完之后再 dispatch 一个对象到 reducer 里。redux 的源码也相当简单：
```javascript
function createThunkMiddleware (extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument)
    }
    return next(action)
  }
}
```
<!--more-->

从源码里可以看到 redux-thunk 先判断到来的 action 是否为函数，如果是函数的话，就调用这个函数，否则就执行下一个 action。

redux-thunk 的缺点很明显，redux 只是执行了这个函数，不会在乎函数主体是什么。每一个异步操作都发起一个有副作用的 action，这样异步代码会分布在每一个 action里，形式不统一，也不易维护。

而 redux-saga 则是将所有的异步操作都统一放到了 saga的文件函数里，使异步操作可以被集中处理，同时也达到了形式上的统一，易于维护。

### 如何使用 redux-saga
redux-saga 包括三个部分
> - worker saga
> 做所有的工作，如调用 API，进行异步请求，并且获得返回结果
> - watcher saga
> 监听被 dispatch 的 actions，当接收到 action 或者知道其被触发时，调用 worker saga 执行任务
> - root saga
> 立即启动 sagas 的唯一入口

首先在 store 的入口文件里加上对应的中间件
```javascript
import { createStore, applyMiddleware, compose } from 'redux'
import createSagaMiddleware from 'redux-saga'
import reducer from './reducers'
import sagas from './sagas'

const sagaMiddleware = createSagaMiddleware()
const store = createStore(reducer, applyMiddleware(sagaMiddleware))
sagaMiddleware.run(sagas)

export default store
```
之后在 sagas 的文件夹里集中写 saga 的代码
```javascript
import axios from 'axios'
import { put, takeEvery } from 'redux-saga/effects'
import { GET_INIT_LIST } from './actionTypes'

function* getInitList (url) {
  try {
    const res = yield call(axios.get, url)
    yield put({ type: 'GET_DATA_SUCCESS', res })
  } catch (e) {
    yield put({ type: 'GET_DATA_FAIL', e })
  }
}

function* mySaga() {
  yield takeEvery(GET_INIT_LIST, getInitList(action.url));
}

export default mySaga;
```

### 常用 API
#### take,takeEvery,takeLatest
创建一条 Effect 描述信息，指示 middleware 等待 Store 上指定的 action。 Generator 会暂停，直到一个与 pattern 匹配的 action 被发起
```javascript
function* mySaga() {
  yield take(GET_INIT_LIST, getInitList(action.url));
}
```
与 take 有类似功能的 API 还有 takeEvery 和 takeLatest
- takeEvery 允许多个 getInitList 实例同时启动，在某个特定时刻，尽管之前还有一个或多个 getInitList 尚未结束，我们还是可以启动一个新的 getInitList 任务
- 和 takeEvery 不同，在任何时刻 takeLatest 只允许一个 getInitList 任务在执行。并且这个任务是最后被启动的那个。 如果已经有一个任务在执行的时候启动另一个 getInitList ，那之前的这个任务会被自动取消。

#### put
用于触发 action，功能上类似于dispatch
```javascript
function* getInitList() {
  yield put({ type: 'GET_DATA_SUCCESS', res })
}
```

#### call(fn, ...args)
call 创建了一条描述结果的信息, 用于调用异步逻辑, 指示 middleware 调用 fn 函数并以 args 为参数。fn 既可以是一个普通函数，也可以是一个 Generator 函数。
如果结果是一个 Generator 对象，middleware 会执行它,如果结果是一个 Promise，middleware 会暂停直到这个 Promise 被 resolve，resolve 后 Generator 会继续执行。 
```javascript
function* fetchProducts() {
  const products = yield Api.fetch('/products')
}
```
假设我们想测试上面的 generator:
```javascript
const iterator = fetchProducts()
assert.deepEqual(iterator.next().value, ??) // 我们期望得到什么？
```
而如果我们使用 call 去调用这个 ajax 方法
```javascript
function* fetchProducts() {
  const products = yield call(Api.fetch, '/products')
}
```
测试的代码就可以这样写了
```javascript
const iterator = fetchProducts()
assert.deepEqual(
  iterator.next().value,
  call(Api.fetch, '/products'),
  "fetchProducts should yield an Effect call(Api.fetch, './products')"
)
```

#### fork(fn, ...args)
创建一条 Effect 描述信息，指示 middleware 以 无阻塞调用 方式执行 fn。
```javascript
const task = yield fork(takeLatest, 'GET_PERSON_DATA', getPersonData)
```
这里面通过 takeLatest 去监听 type: 'GET_PERSON_DATA' 的 action，当有一个这样的 action 被触发，就会 fork 出一个 task
这边讲下 fork 与 call 的区别
- fork 是非阻塞的，非阻塞就是遇到它，不需要等它执行完, 就可以直接往下运行
- call 是阻塞，阻塞的意思就是一定要等它执行完, 才可以直接往下运行
- **fork是返回一个任务，这个任务是可以被取消的；而call就是它执行的正常返回结果！（非常重要）**

#### cancel
一旦任务被 fork，可以使用 yield cancel(task) 来中止任务执行。取消正在运行的任务。
```javascript
while (yield take('START_BACKGROUND_SYNC')) {
    // 启动后台任务
    const bgSyncTask = yield fork(bgSync)
    // 等待用户的停止操作
    yield take('STOP_BACKGROUND_SYNC')
    // 用户点击了停止，取消后台任务
    // 这会导致被 fork 的 bgSync 任务跳进它的 finally 区块
    yield cancel(bgSyncTask)
  }
```

#### all
效果与 Promise.all 相对应,创建一个 Effect 描述信息，用来命令 middleware 并行地运行多个 Effect，并等待它们全部完成
```javascript
yield all([
  call(fetchResource, 'users'),
  call(fetchResource, 'comments')
])
```
当并发运行 Effect 时，middleware 将暂停 Generator，直到以下任一情况发生：
- 所有 Effect 都成功完成：返回一个包含所有 Effect 结果的数组，并恢复 Generator。
- 在所有 Effect 完成之前，有一个 Effect 被 reject：在 Generator 中抛出 reject 错误。

#### race
效果与 Promise.race 相对应,创建一个 Effect 描述信息，用来命令 middleware 在多个 Effect 间进行竞赛
```javascript
const [response, cancel] = yield race([
  call(fetchUsers),
  take('CANCEL_FETCH')
])
```
如果 fetchUsers 先 resolve，那么 response 将是 fetchUsers 的结果，并且 cancel 将是 undefined
如果在 fetchUsers 完成之前，Store 上先发起了一个 'CANCEL_FETCH' 类型的 action，那么 response 将是 undefined，并且 cancel 将是被发起的 action

#### delay
返回一个 effect 描述信息，用于阻塞执行 ms 毫秒，并返回 val 值
```javascript
const [posts, timeout] = yield race({
  call(fetchUsers),
  call(delay, 1000)
})
```
上面的例子其实就是限制了 fetchUsers 必须在 1秒内作出响应，否则会作超时处理。

### 优点
- 查询与责任分离，保证了action的纯洁性，符合 redux 设计思想
- 实现以同步方式写异步操作，容易理解，逻辑清晰
- 通过发送指令而不是直接调用让异步操作变得容易测试
- 监听、执行自动化
- 高级的异步控制流以及并发管理,实现颗粒更小的异步控制，通过 `fork` 实现并发任务。
- 架构上的优势：将所有的异步流程控制都移入到了 sagas，UI 组件不用执行业务逻辑，只需 dispatch action 就行，增强组件复用性

### 缺点
- action 任务拆分更细，原有流程上相当于多了一个环节,对开发者的设计和抽象拆分能力更有要求
- 代码复杂性也有所增加，比较复杂，学习成本高
- 异步请求相关的问题较难调试排查

### demo
写了一个 redux-saga 的一个小 demo，书写了 redux-saga 的主要架构，并实现了发送请求，取消请求和超时请求几个小功能。
![](https://i.loli.net/2018/11/19/5bf28b123f9c0.jpeg)
> **github 地址：https://github.com/Kerminate/redux-saga-demo**

### 参考
[Redux-saga 中文文档](https://redux-saga-in-chinese.js.org/)
[聊一聊 redux 异步流之 redux-saga](https://www.jianshu.com/p/e84493c7af35)
[Redux-Saga 实用指北](https://juejin.im/post/5ad83a70f265da503825b2b4)
[从redux-thunk到redux-saga实践](https://github.com/Pines-Cheng/blog/issues/9)[对使用Redux和Redux-saga管理状态的思考](https://www.jianshu.com/p/6bcf4573ca28?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)