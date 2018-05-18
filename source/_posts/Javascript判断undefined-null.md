---
title: 'Javascript判断undefined, null, {}, [], '', 0'
date: 2018-03-02 18:51:50
tags: 前端
---
先通过下面代码看 `undefined`, `null`, `{}`, `[]`, `''`, `0` 的布尔值和类型
<!--more-->
```javascript
let a = null
let b = {}
let c = []
let d = ''
let e = 0
let f

console.log(!a) // true
console.log(!b) // false
console.log(!c) // false
console.log(!d) // true
console.log(!e) // true
console.log(!f) // true

console.log(typeof a) // object
console.log(typeof b) // object
console.log(typeof c) // object
console.log(typeof d) // string
console.log(typeof e) // number
console.log(typeof f) // undefined

console.log(a == null) // true
console.log(b == null) // false
console.log(c == null) // false
console.log(d == null) // false
console.log(e == null) // false
console.log(f == null) // true
console.log(f === undefined) // true

console.log(Array.isArray(a)) // false
console.log(Array.isArray(b)) // false
console.log(Array.isArray(c)) // true
console.log(Array.isArray(d)) // false
console.log(Array.isArray(e)) // false

console.log(Object.prototype.toString.call(a)) // [object Null]
console.log(Object.prototype.toString.call(b)) // [object Object]
console.log(Object.prototype.toString.call(c)) // [object Array]
console.log(Object.prototype.toString.call(d)) // [object String]
console.log(Object.prototype.toString.call(e)) // [object Number]
console.log(Object.prototype.toString.call(f)) // [object Undefined]
```
从上面代码得出判断过程
- 先判断这些变量的布尔值, 其中 `null` , `undefined` , `0` 和 `''` 的布尔值都为 `false`
- 再通过判断它们的类型, `null` 的类型为 `object`, `undefined` 的类型为 `undefined`, `0` 为 `number`, `''` 为 `string`
- `{}` 和 `[]` 的布尔值都为 `true`, 通过 `Array.isArray` 方法判断是数组还是对象
- 最简单的方法是通过 `Object.prototype.toString.call()` 直接判断

顺便看一下各个数据类型 `typeof` 和 `instanceof` 的输出结果
```javascript
let obj = {}
let arr = [1, 2]
let num1 = 1
let num2 = new Number(1)
let str1 = 'string'
let str2 = new String('string')
function fun () {}

console.log(typeof obj) // object
console.log(typeof arr) // object
console.log(typeof num1) // number
console.log(typeof num2) // object
console.log(typeof str1) // string
console.log(typeof str2) // object
console.log(typeof fun) // function
console.log(typeof undefined) // undefined
console.log(typeof null) // object

console.log(obj instanceof Object) // true
console.log(arr instanceof Array) // true
console.log(arr instanceof Object) // true
console.log(num1 instanceof Number) // false
console.log(num1 instanceof Object) // false
console.log(num2 instanceof Number) // true
console.log(num2 instanceof Object) // true
console.log(str1 instanceof String) // false
console.log(str1 instanceof Object) // false
console.log(str2 instanceof String) // true
console.log(str2 instanceof Object) // true
console.log(fun instanceof Object) // true
console.log(fun instanceof Function) // true

console.log(undefined instanceof Object) // false
console.log(null instanceof Object) // false
```