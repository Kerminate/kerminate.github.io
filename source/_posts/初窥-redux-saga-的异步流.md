---
title: 初窥 redux-saga 的异步流
date: 2018-09-21 14:02:51
tags: 技术帖
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
从源码里可以看到 redux-thunk 先判断到来的 action 是否为函数，如果是函数的话，就调用这个函数，否则就执行下一个 action。

redux-thunk 的缺点很明显，redux 只是执行了这个函数，不会在乎函数主体是什么。每一个异步操作都发起一个有副作用的 action，这样异步代码会分布在每一个 action里，形式不统一，也不易维护。

而 redux-saga 则是将所有的异步操作都统一放到了 saga的文件函数里，使异步操作可以被集中处理，同时也达到了形式上的统一，易于维护。

### 如何使用 redux-saga
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