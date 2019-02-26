---
title: Javascript 交换变量的几种方法
date: 2018-01-26 21:23:25
tags: 算法
categories: Javascript
---

最近在用 javascript 刷 leetcode 的时候，遇到一道题，需要交换变量的值，突然发现用 js 实现有多种方法，下面简单列举一下：

<!--more-->

```
// 这是初始变量
let a = 3
let b = 4
```
```
// 增加一个临时变量
let mid = a
a = b
b = mid
```
```
// 通过和形式
a += b
b = a - b
a -= b
```
```
// 通过差形式
a -= b
b = a + b
a = b - a
```
```
// 使用异或运算
a ^= b
b ^= a
a ^= b
```
```
// 使用解构赋值
[a, b] = [b, a]
```
