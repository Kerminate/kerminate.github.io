---
title: Javascript 优化计算斐波那契数列
date: 2018-03-19 19:18:29
tags: 算法
---
先介绍一下斐波那契数列, 它是一个形如这样的序列：1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, ...
如果设 F(n) 为该数列的第n项（n∈N*），那么这句话可以写成如下形式：:F(n) = F(n-1) + F(n-2)

常用的解法如下
```javascript
function fibonacci (n) {
  if (n < 2) return 1
  return fibonacci(n - 1) + fibonacci(n - 2)
}
```
这种写法使函数调用函数，所有的 **调用帧** 会形成一个 **调用栈**，最后导致堆栈溢出
```javascript
fibonacci(10) // 89
fibonacci(100) // 堆栈溢出
fibonacci(500) // 堆栈溢出
```
<!--more-->

## 优化方法

### 数组缓存
使用数组记录斐波那契数列的中间值，使它不用被多次计算
```javascript
let fibonacci = (function () {
  let arr = []
  return function (n) {
    if (arr[n] !== undefined) {
      return arr[n]
    }
    arr[n] = n < 2 ? 1 : fibonacci(n - 1) + fibonacci(n - 2)
    return arr[n]
  }
})()
```

### 高效迭代
不通过递归，在内部一次while循环计算
```javascript
function fibonacci (n) {
  let x = 1, y = 1, i = 1
  while (i < n) {
    y = x + y
    x = y - x
    i++
  }
  return y
}
```

### 尾递归优化
结合上面的算法，使用 `ES6` 的尾递归优化写法
```javascript
function fibonacci (n, ac1 = 1, ac2 = 1) {
  if (n <= 1) return ac2
  return fibonacci(n - 1, ac2, ac1 + ac2)
}
```