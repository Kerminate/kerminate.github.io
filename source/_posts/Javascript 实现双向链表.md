---
title: Javascript 实现双向链表
date: 2018-03-01 19:28:28
tags: 算法
---
`js` 实现双向链表, 双向链表需要增加一个 `previous` 属性
<!--more-->

```javascript
function Node (val) {
  this.val = val
  this.previous = null
  this.next = null
}

function ListNode () {
  this.head = new Node('head')
  this.find = find
  this.display = display
  this.push_back = pushBack
  this.pushFront = pushFront
  this.insert = insert
  this.remove = remove
  this.reverse = reverse
  this.clear = clear
}

function find (item) {
  let currentNode = this.head
  while (currentNode.val !== item) {
    currentNode = currentNode.next
  }
  return currentNode
}

function display () {
  let currentNode = this.head
  let arr = []
  while (currentNode.next != null) {
    arr.push(currentNode.next.val)
    currentNode = currentNode.next
  }
  console.log(arr)
}

function pushBack (item) {
  let newNode = new Node(item)
  let currentNode = this.head
  while (currentNode.next != null) {
    currentNode = currentNode.next
  }
  currentNode.next = newNode
  newNode.previous = currentNode
}

function pushFront (item) {
  let newNode = new Node(item)
  let currentNode = this.head
  if (currentNode.next == null) {
    currentNode.next = newNode
  } else {
    currentNode.next.previous = newNode
    newNode.next = currentNode.next
    currentNode.next = newNode
  }
}

function insert (item, newElement) {
  let newNode = new Node(newElement)
  let currentNode = this.find(item)
  newNode.next = currentNode.next
  newNode.previous = currentNode
  if (currentNode.next != null) currentNode.next.previous = newNode
  currentNode.next = newNode
}

function remove (item) {
  let currentNode = this.find(item)
  currentNode.previous.next = currentNode.next
  if (currentNode.next != null) currentNode.next.previous = currentNode.previous
  currentNode.previous = null
  currentNode.next = null
}

function reverse () {
  let currentNode = this.head
  let arr = []
  while (currentNode.next != null) {
    arr.push(currentNode.next)
    currentNode = currentNode.next
  }
  this.head.next = currentNode
  for (let i = arr.length - 1; i >= 0; i--) {
    currentNode = arr[i]
    if (i === arr.length - 1) currentNode.previous = null
    else currentNode.previous = arr[i + 1]
    if (i === 0) currentNode.next = null
    else currentNode.next = arr[i - 1]
  }
}

function clear () {
  let currentNode = this.head
  currentNode.next = null
}

let list = new ListNode()
list.push_back('aaa')
list.push_back('bbb')
list.insert('aaa', 'ccc')
list.pushFront('11')
list.display() // [ '11', 'aaa', 'ccc', 'bbb' ]
list.reverse()
list.display() // [ 'bbb', 'ccc', 'aaa', '11' ]
list.remove('aaa')
list.display() // [ 'bbb', 'ccc', '11' ]
list.clear() // []
```