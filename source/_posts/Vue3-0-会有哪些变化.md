---
title: Vue 3.0 会有哪些变化
date: 2019-06-29 11:28:40
tags: 技术帖
categories: 技术帖
---
前段时间在 [Vue RFCs](https://github.com/vuejs/rfcs) 提出的 Functional API 在社区中引起了轩然大波，感觉是 Vue 史上最有争议的一个特性。那么，Vue 3.0 到底会带来哪些改变呢，让我们一起来看一看。

<!--more-->

## 更快

### Object.defineProperty -> Proxy

在 Vue 2.0 里面是通过使用 `Object.defineProperty` 来进行数据侦测
```javascript
function definedReactive(data, key, val) {
  Object.defineProperty(data, key, {
    enumerable: true,
    configurable: true,
    get() {
      return val;
    },
    set(newVal) {
      if (val === newVal) return;
      val = newVal;
    }
  });
}

```
每当从 data 的 key 中读取数据时，get 函数被触发，每当往 data 的 key 中设置数据时，set 函数被触发。
但是可以看出来 `Object.defineProperty` 能够监测对象的变化，但是对于数组的一些原生方法如 `push, pop` 等就无能为力。
因此，Vue 中其实对数组的原生方法进行了覆盖，共七个（`push, pop, shift, unshift, splice, sort, reverse`），实际上就是使用了拦截器覆盖了 Array 的原型，使得在使用这几个函数的时候能够做到数据侦测。
`Object.defineProperty` 通过 `getter/setter` 来追踪变化，但它其实只能追踪一个数据是否被修改，无法追踪新增属性和删除属性，所以 Vue 又提供了 `vm.$set` 和 `vm.$delete` 方法来实现响应式地新增和删除属性。

ES6 新增的 `Proxy` 可以直接监听对象而非属性，所以以上 `Object.defineProperty` 的缺点它都没有。

除此之外，使用 `Object.defineProperty` 在监听数组变化的时候，破坏了 **[JS 引擎的渲染](https://github.com/dt-fe/weekly/blob/master/62.%E7%B2%BE%E8%AF%BB%E3%80%8AJS%20%E5%BC%95%E6%93%8E%E5%9F%BA%E7%A1%80%E4%B9%8B%20Shapes%20and%20Inline%20Caches%E3%80%8B.md)**，性能上与 `Proxy` 也有不少差距。

### Virtual DOM 重构
![](https://i.loli.net/2019/06/29/5d1736bac776433441.jpeg)

这边补充下：其实在 Vue 1.0 的时候，它的变化侦测力度更细，具体到了某个节点，在 2.0 的时候采取了一个中等粒度的解决方案，状态侦测不再细化到某个具体节点，而是某个组件。从而大大缩减了依赖数量和 watcher 数量，降低了内存开销和依赖追踪的开销。

```html
<template>
  <div id="content">
    <p class="text">Lorem ipsum</p>
    <p class="text">Lorem ipsum</p>
    <p class="text">{{ message }}</p>
    <p class="text">Lorem ipsum</p>
    <p class="text">Lorem ipsum</p>
  </div>
</template>
```

对于这么一段模板代码，在更新的时候，我们首先要求判断这是不是同一个 div，它的属性比如 id 等有没有变化，然后判断它的子元素顺序有没有发生改变，每个子元素的属性是否发生变化。

> 传统 vdom 的性能跟模板大小正相关，跟动态节点的数量无关。在一些组件整个模板内还有少量动态节点的情况下，这些遍历都是性能的浪费。

而其实上述代码中其他四个 `p` 的节点都没有发生变化，理想情况下我们只需要去 check 带有 `message` 变量的那个节点的变化。 

```html
<template>
  <div>
    <p class="text">Lorem ipsum</p>
    <p v-if="ok">
      <span>Lorem ipsum</span>
      <span>{{ message }}</span>
    </p>
  </div>
</template>
```

对于以上代码，就有了节点结构的变化：`v-if`。我们可以将它拆成内外2块。

![](https://i.loli.net/2019/06/30/5d181ce03576128336.jpeg)

对于 `v-for` 节点来说，也是一样。

![](https://i.loli.net/2019/06/30/5d181d77611d293658.jpeg)

我们会发现动态结构性节点的变化只会出现在 `v-if`，`v-for` 这类结构性指令下，因此，我们以结构性指令为边界将整个模板切分为一个一个的块，称之为 **Block tree**

![](https://i.loli.net/2019/06/30/5d182827a8ba974072.jpeg)

> 这样，新策略就会将 vdom 的更新性能与模板整体大小相关提升为与动态内容的数量相关。

除此之外，比如对一个节点设置动态 class 也是属于动态节点的。但是因为动态 class 太常见，而且经常同一个节点只有 class 发生变化，所以在 Vue 3.0 中对这类节点进行 diff 的时候就不会对 props 的对象进行比对，直接设置 class 就行了。

那么， Vue 3.0 到底有多快，尤雨溪进行了这样一个实验。

- 1000 个循环的 v-for
- 每个循环包括
  - 三层嵌套共 12 个 DOM 元素
  - 2 个动态 class 绑定
  - 1 个动态文字绑定
  - 1 个动态 id 属性绑定
- 动态更新所有绑定 100 次取平均值

> 结果：**36 ms -> 5.44 ms**

## 加强 TypeScript 支持

### 为什么撤销 Class API？

原本目的是更好的 TS 支持，但是 Props 和其它需要注入到 this 的属性导致类型声明依然存在问题，Decorators 提案的严重不稳定使得依赖它的方案具有重大风险(经历了两三次的大改，还是没有进入 stage3)。除了类型支持以外 Class API 并不带来任何新的优势，而其实我们有更好的选择...

#### Function-based API

```js
const App = {
  setup() {
    // data
    const count = value(0)
    // computed
    const plusOne = computed(() => count.value + 1)
    // method
    const increment = () => { count.value++ }
    // watch
    watch(() => count.value * 2, v => console.log(v))
    // lifecycle
    onMounted(() => console.log('mounted!'))
    // 暴露给模板或渲染函数
    return { count }
  }
}
```

对比 Class API
- 更好的 TypeScript 类型推导支持
- 更灵活的逻辑复用能力
- Tree-shaking 友好（可以使 vue 打包后的体积更小）
- 代码更容易被压缩

从逻辑复用的角度来讲，比起原来的 mixin 或者 slot 插槽，会更有优点
- 没有命名空间冲突
- 数据来源清晰
- 没有额外的组件性能消耗

可能熟悉 React 的同学会发现，这与 React Hooks 的写法特别像，没错。它的表现与 Hooks 很一致，但是其实又有不少区别。
- 同样的逻辑组合，复用能力。
- 整体上更符合 JavaScript 的直觉
- 不受调用顺序的限制，可以有条件地被调用
- 不会在后续更新时不断产生大量的内联函数而影响引擎优化或是导致 GC 压力
- 不需要总是使用 useCallback 来缓存传给子组件的回调以防止过度更新
- 不需要担心传了错误的依赖数组给 useEffect/useMemo/useCallback 从而导致回调中使用了过期的值 —— Vue 的依赖追踪是全自动的。