---
title: readline实现js的输入输出
date: 2018-03-17 13:47:01
tags: Node
---

学C++的时候，有cin, cout, Java也有println和Scanner控件能够实现输入输出。在Node.js里，通过readline模块实现输入输出。
<!--more-->

## 什么是readline
readline是Node.js里实现标准输入输出的封装好的模块，通过这个模块我们可以以逐行的方式读取数据流

## 如何使用Readline
- 创建readline实例
- 学习里面的接口方法
- 学习监听与处理readline事件

### 实例1
```javascript
const readline = require('readline')

let rl = readline.createInterface({
  // process为node里的全局变量
  input: process.stdin,
  output: process.stdout
})

// 输入通过line记录
rl.on('line', (line) => {
  console.log('My name is ' + line)
})

// 结束进程时监听的方法(ctrl + c)
rl.on('close', () => {
  console.log('Bye bye!')
  process.exit(0)
})
```
输入输出如下
![](http://or7tt6rug.bkt.clouddn.com/readline.png)

### 下面通过一道网易在线编程题实践一下

#### [魔法币](https://www.nowcoder.com/test/question/32c71b52db52424c89a565e4134bfe4e?pid=6910869&tid=14179440)
- 描述
  小易准备去魔法王国采购魔法神器,购买魔法神器需要使用魔法币,但是小易现在一枚魔法币都没有,但是小易有两台魔法机器可以通过投入x(x可以为0)个魔法币产生更多的魔法币。
  魔法机器1: 如果投入x个魔法币,魔法机器会将其变为2x+1个魔法币
  魔法机器2: 如果投入x个魔法币,魔法机器会将其变为2x+2个魔法币
  小易采购魔法神器总共需要n个魔法币,所以小易只能通过两台魔法机器产生恰好n个魔法币,小易需要你帮他设计一个投入方案使他最后恰好拥有n个魔法币。

- 输入描述:
输入包括一行,包括一个正整数n(1 ≤ n ≤ 10^9),表示小易需要的魔法币数量。

- 输出描述:
输出一个字符串,每个字符表示该次小易选取投入的魔法机器。其中只包含字符'1'和'2'。

- 输入例子1:
10

- 输出例子1:
122

#### ac代码
```javascript
const readline = require('readline')
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
})

rl.on('line', (line) => {
  let num = parseInt(line)
  let arr = []
  while (num !== 0) {
    if (num % 2 === 0) {
      arr.push('2')
      num = (num - 2) / 2
    } else {
      arr.push('1')
      num = (num - 1) / 2
    }
  }
  let res = arr.reverse().join('')
  console.log(res)
  process.exit(0)
})

```

### 涉及到多行输入可看例题二

#### [射击游戏](https://www.nowcoder.com/question/next?pid=6910869&qid=126948&tid=14179440)
- 描述
小易正在玩一款新出的射击游戏,这个射击游戏在一个二维平面进行,小易在坐标原点(0,0),平面上有n只怪物,每个怪物有所在的坐标(x[i], y[i])。小易进行一次射击会把x轴和y轴上(包含坐标原点)的怪物一次性消灭。
小易是这个游戏的VIP玩家,他拥有两项特权操作:
1、让平面内的所有怪物同时向任意同一方向移动任意同一距离
2、让平面内的所有怪物同时对于小易(0,0)旋转任意同一角度
小易要进行一次射击。小易在进行射击前,可以使用这两项特权操作任意次。
小易想知道在他射击的时候最多可以同时消灭多少只怪物,请你帮帮小易。 

如样例所示: 
![](http://or7tt6rug.bkt.clouddn.com/shoot.png)
所有点对于坐标原点(0,0)顺时针或者逆时针旋转45°,可以让所有点都在坐标轴上,所以5个怪物都可以消灭。 

- 输入描述:
输入包括三行。
第一行中有一个正整数n(1 ≤ n ≤ 50),表示平面内的怪物数量。
第二行包括n个整数x[i](-1,000,000 ≤ x[i] ≤ 1,000,000),表示每只怪物所在坐标的横坐标,以空格分割。
第二行包括n个整数y[i](-1,000,000 ≤ y[i] ≤ 1,000,000),表示每只怪物所在坐标的纵坐标,以空格分割。

- 输出描述:
输出一个整数表示小易最多能消灭多少只怪物。

- 输入例子1:
5
0 -1 1 1 -1
0 -1 -1 1 1

- 输出例子1:
5

>**思路是任意找出三个点构成坐标轴，看其他点落在坐标轴上的个数，选出落在坐标轴上最多的点**

#### ac代码
```javascript
const readline = require('readline')

let rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
})

const lines = []
rl.on('line', (line) => {
  lines.push(line)
  if (lines.length === 3) {
    let n = parseInt(lines[0])
    let x = lines[1].split(' ').map((num) => parseInt(num))
    let y = lines[2].split(' ').map((num) => parseInt(num))
    let res
    if (n < 3) res = n
    else res = 3
    for (let i = 0; i < n; i++) {
      for (let j = 0; j < n; j++) {
        if (i === j) continue
        for (let k = 0; k < n; k++) {
          if (i === k || j === k) continue
          let num = 3
          for (let r = 0; r < n; r++) {
            if (r === i || r === j || r === k) continue
            if ((y[r] - y[i]) * (x[r] - x[j]) == (y[r] - y[j]) * (x[r] - x[i])) num++
            else if ((x[i] - x[j]) * (x[r] - x[k]) == (y[j] - y[i]) * (y[r] - y[k])) num++
          }
          res = Math.max(res, num)
        }
      }
    }
    console.log(res)
    process.exit(0)
  }
})
```