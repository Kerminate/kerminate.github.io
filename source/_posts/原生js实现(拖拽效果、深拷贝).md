---
title: 原生js实现(拖拽效果、深拷贝)
date: 2018-03-16 21:14:33
tags: 前端
---

通过原生的 `onmousedown` `onmousemove` 和 `onmouseup` 事件实现元素的拖拽效果

通过递归实现深拷贝
<!--more-->
### 实现拖拽

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title></title>
  <style>
    #drag {
      position: absolute;
      top: 100px;
      left: 300px;
      width: 300px;
      height: 200px;
      background-color: red;
    }
  </style>
  <script>
    window.onload = function () {
      let obj
      let diffX
      let diffY
      let drag = document.getElementById('drag')

      drag.onmousedown = function (el) {
        let e = el || window.event // window.event兼容IE6.7，获得鼠标对象
        let target = e.target || e.srcElement // e.srcElement兼容IE，获得当前事件源
        diffX = e.clientX - target.offsetLeft // e.clientX为鼠标的水平坐标，target.offsetLeft为该元素左边距版面距离，求得鼠标距元素左边界距离
        diffY = e.clientY - target.offsetTop
        obj = target
      }

      document.onmousemove = function (el) {
        if (obj) {
          let e = el || window.event
          let left = e.clientX - diffX // 此时元素左边界距离版面距离
          let top = e.clientY - diffY     
          if (left < 0) left = 0
          else if (left > window.offsetWidth - obj.offsetWidth) left = window.offsetWidth - obj.offsetWidth
          if (top < 0) top = 0
          else if (top > window.offsetHeight - obj.offsetHeight) top = window.offsetHeight - obj.offsetHeight
          
          obj.style.cursor = 'move'
          obj.style.left = left + 'px'
          obj.style.top = top + 'px'
        }
      }

      document.onmouseup = function (el) {
        if (obj) {
          obj.style.cursor = ''
        }
        obj = null
      }
    }
  </script>
</head>

<body>
  <div id="drag"></div>
</body>
</html>
```

### 实现深拷贝
```javascript
function deepClone (data) {
  let des = Object.prototype.toString.call(data).substring(8).split('')
  des.splice(-1)
  let type = des.join('')
  let obj
  if (type === 'Array') {
    obj = []
  } else if (type === 'Object') {
    obj = {}
  } else {
    return data
  }
  if (type === 'Array') {
    for (let i = 0; i < data.length; i++) {
      obj.push(deepClone(data[i]))
    }
  } else if (type === 'Object') {
    for (let key in data) {
      obj[key] = deepClone(data[key])
    }
  }
  return obj
}

console.log(deepClone({ a: 1, b: 2, c: [1, 2, { e: 4 }] })) // { a: 1, b: 2, c: [ 1, 2, { e: 4 } ] }
```