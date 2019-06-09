---
title: 参加 VueConf2019
date: 2019-06-08 22:19:50
tags: Conf
categories: Conf
---
千呼万唤始出来，终于等到了 VueConf2019。本次 VueConf 选在了交大百年历史的文治堂里，下面我们一起来一睹风采！
![](https://i.loli.net/2019/06/09/5cfc73dcdf2e255399.jpeg)

<!--more-->
大会当天，我一早便来到了目的地。其中在来的路上还抢了上影节的票，因为搞错时间，最后转手一卖还赚了40，哈哈哈哈哈😁。到了文治堂门口，领了大会T恤和胸牌，就迅速进入大会堂里占据了有利位置。

### 尤雨溪 @ State of Vue
首先，尤大先简要介绍了下 Vue 的发展现状。Chrome DevTools 有约 90 万的周活用户，而 React 有 160 万，可以看到 Vue 的开发者也是越来越多了。而 Vue 的官方团队大约为 20人，大多数都是兼职的开发者。让我意外的是，整个 Vue 团队的唯一全职开发是蒋豪群（@sodatea），目前维护 CLI 及相关工具链。
之后，就是重头戏，尤大分享了下 3.0 的进展。
![](https://i.loli.net/2019/06/09/5cfc7ded226b945950.jpeg)
当然，里面的关键点也就是图中的高亮处：**更快** 和 **加强 TypeScript 支持**。

#### 更快
更快主要是做了三点：
- Object.defineProperty -> Proxy
- Virtual DOM 重构
- 更多编译时优化
    - Slot 默认编译成函数
    - Monomorphic vnode factory
    - Compiler-generated flags for vnode/children types

大家都知道，在 Vue 2.0 里是使用 `Object.defineProperty` 来进行数据侦测。但是其实它有不少的缺点，比如它在监听数组变化的时候，破坏了 **[JS 引擎的渲染](https://github.com/dt-fe/weekly/blob/master/62.%E7%B2%BE%E8%AF%BB%E3%80%8AJS%20%E5%BC%95%E6%93%8E%E5%9F%BA%E7%A1%80%E4%B9%8B%20Shapes%20and%20Inline%20Caches%E3%80%8B.md)**，而且为了做到数据侦测，也重写了数组的多个原生方法，开发者有时候在改变数组和对象的值时还必须使用 Vue 提供的 API（vm.$set）。在 3.0，会使用 `Proxy` 来进行数据侦测，它属于 ES6，不需要对原始对象做太多改动，效率更高，当然它也有缺点，比如无法支持 IE11。因此尤大也说了在 3.0 之后，会把数据侦测的内容单独抽成模块，对现代浏览器使用 `Proxy` 做到更高效率，同时也保留了 `Object.defineProperty` 来兼容 IE11。

对于 Virtual DOM 的优化是在兼容手写 render function 的同时，最大化地利用模板的静态信息。按照尤大提到的“动静结合”，也就是区别处理 v-if,v-for 这种动态节点，将节点关系切分为一个一个的区块树（Block Tree），减少无谓的遍历。
![](https://i.loli.net/2019/06/09/5cfc9785872df20068.jpeg)

新策略将 vdom 更新性能由与模板整体大小相关提升为与动态内容的数量相关。

#### 加强 TypeScript 支持
一个最大的变动是，撤销了 Class API。

##### 为什么撤销 Class API？

原本目的是更好的 TS 支持，但是 Props 和其它需要注入到 this 的属性导致类型声明依然存在问题，Decorators 提案的严重不稳定使得依赖它的方案具有重大风险。除了类型支持以外 Class API 并不带来任何新的优势，而其实我们有更好的选择...

##### Function-based API
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
- Tree-shaking 友好
- 代码更容易被压缩

可能熟悉 React 的同学会发现，这与 React Hooks 的写法特别像，没错。它的表现与 Hooks 很一致，但是其实又有不少区别。
对比 React Hooks，它有同样的逻辑组合，复用能力。
区别是，因为 Vue 底层是通过依赖收集，进而只会渲染发生变化的组件，因此 Class API 与 Hooks 不同，实际上只调用一次，没有内存/GC 压力，不存在内敛回调导致子组件永远更新的问题。

Vue 3.0 更多的详情可以参考 [RFC](https://github.com/vuejs/rfcs).

### 回到自身
上面大多数内容都是直接搬了尤大分享的东西，结合了一点点自身的理解。原本嘛，这应该只是一篇会后的感受，不需要讲那么多的技术细节，但是考虑到 Vue3.0 早就让人翘首以盼，而且毕竟是第一次见尤大，所以就将大概内容重新阐述了一遍。后面，就不会讲那么多费脑的东西，回到自身，单纯的个人视角和感受，hhhhh。

尤大讲完后是来自汇丰银行的卢俊杰，分享了 Vue 的单元测试之旅。不过，后来被我发现尤大竟然在场下偷偷地看 NBA...
![](https://i.loli.net/2019/06/09/5cfca2937dcbd47323.jpeg)

卢俊杰分享的单测挺细致的，也讲到了不少小技巧，后来他问在场有多少团队有在项目中加入单测。让我意外的是，数百人中，只有寥寥数人举起了手😂。包括我司在内，大多数都没有加单测，我也只在个人项目中使用过单测。当然这也与国内现在大部分都是敏捷开发，快速迭代有关，而在外企或者银行这种比较保守，慢节奏的公司里可能更能落地。当然日后将单测在团队里落地也是我的未来规划之一。

接着是 Winter的闪电分享，还有 React China 的发起人题叶的“踢馆”（这是我第二次见到他了，上次是在 18 年的 FCC 秋季交流会上）。然后就到了午饭时间。这次活动不包午饭，我就只能到外面独自觅食。吃完饭在交大逛了一圈，粗粗地领略了下百年沉淀的名校气息。

很快，下午的分享就开始了。蒋豪群（Vue 团队里唯一的全职开发）带来了 Vue Beyond Vue Loader，主要是讲了一些 Vue 除了单文件组件以外的各种用法。然后是来自百度的袁源讲了下 Vue 开发 ECharts 踩坑指南。跟国情密切相关，后面带来的大多是与小程序相关的分享，有快手内部实现的小程序平台，百度的 Mars 框架（小程序 + H5 同构开发方案），以及 uni-app 架构师带来的分享。

终于到了下午茶时间。比较戏剧的是，尤大被堵在厕所外，一群人围着他要签名。见到如此场景，我二话不说，一个健步冲了上去，也加入了讨要签名的人群中😄。经过一番努力，我不仅得到了尤大的签名，还获得了与他的合影，哈哈哈哈哈。
![](https://i.loli.net/2019/06/09/5cfcac330356226964.jpeg)

合完影瞬间感觉票价回本了😊，吃了🍞和☕️后，继续回到现场。

下面的分享我还是挺喜欢的。是来自蚂蚁金服的真山带来的 VuePress。相信很多前端同学都有自己的博客，而静态网站的生成工具也很多，包括 Hexo, Jekyll, Wordpress 等等。我本身使用的是 Hexo, 而 VuePress 给了用户在 Markdown 里写 Vue 的机会。还可以直接使用 Vue 去写一个自定义主题，是不是听起来就有让你去撸一个主题的冲动😁。

最后，献上本次活动的大合照！
![](https://i.loli.net/2019/06/09/5cfcb19eb5e0c64334.jpeg)