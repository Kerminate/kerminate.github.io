---
title: Vue + Koa 搭建 ACM OJ
date: 2018-05-31 11:27:19
tags: 技术贴
---

花了两个多月时间，我与 [lazzzis](https://github.com/lazzzis) 完成了第二版本的Putong OJ，因为中间忙着春招以及毕业设计等，项目最近才正式上线。

项目线上地址：[http://acm.cjlu.edu.cn/](http://acm.cjlu.edu.cn/)
项目前端地址：[https://github.com/acm309/PutongOJ-FE](https://github.com/acm309/PutongOJ-FE)
项目后端地址：[https://github.com/acm309/PutongOJ](https://github.com/acm309/PutongOJ)

> 这里求一下star啊\(^o^)/~

本OJ前端架构为 Vue2.5 + vue-router + vuex + axios + iview + stylus + webpack3.6
后端架构为 Koa2 + MongoDB + redis

<!--more-->

## 开发背景
我们学校 acm 起步较晚，最早的 OJ 是由 Hust OJ 魔改而来，界面写的比较粗糙。2年前，那届的 acm 队长本来决定使用 Vue + Go 重写一下 OJ，但是因为一些原因，他跑路了，最后只 fork 了一个开源 OJ。一年前，[lazzzis](https://github.com/lazzzis) 开始重构 OJ，采用了 Vue + node，开发出了 Putong OJ 的第一个版本。今年，由于老师增加了功能上的一些需求，再加上后端数据结构又发生了一些变化，以及对第一个版本不太满意，我与 lazzzis 再次重构，开发出了 Putong OJ V2 版本。 

## 技术选型
考虑过要用 React 开发，(好吧，说实话写OJ的时候我还不会React),但是 Vue 上手简单且中文资源丰富，所以决定使用 Vue 全家桶。最初 vue1.0 时官方推荐 vue-resource，后来在 2.0 时 Vue 官方不再推荐 vue-resource， 而是推荐使用 axios 作前后端通信。开发初期一开始用了 element 作为 vue 的 UI 库，后来转而使用了 iview。其实这两个 UI 库相当像，都是 ant-design 风格的，api也比较一致，在我眼里比较打的区别是 element 的组件更大，iview 的更小巧（视觉上的大小，element small 的跟 iview 的 default 差不多大）。

后端其实我们不太喜欢 Java，最后用了轻量又方便的 node。数据库用了 MongoDB，主要是方便 js 操作，同时用 redis 做数据缓存，并做了简单的消息队列。

## 预览
主题色采用了 lazzzis 比较喜欢的骚紫。

![](http://or7tt6rug.bkt.clouddn.com/%E7%BB%9F%E8%AE%A1.jpg)
![](http://or7tt6rug.bkt.clouddn.com/%E6%AF%94%E8%B5%9B.jpg)

### 实现功能
OJ分为web端和判题端，这边主要分析web端，判题端 由 [Acdream的判题端](https://github.com/KIDx/Judger) 魔改而来。web 端共有消息模块，题目模块，讨论模块，状态模块，排行模块，比赛模块与管理员模块七大模块。
本OJ提供了两种用户，普通用户和管理员用户。顾名思义，普通用户只能答题，参加比赛，发帖，查看信息等，管理员用户拥有对信息，题目，比赛等增删改查的权限。

- 消息模块
    就是 OJ 的首页，含有列表页和消息详情页，主要就是管理员发布的消息。
- 题目模块
    OJ 的核心模块之一，含有题目列表页和题目详情页，题目详情页里有 6 个 tab 页，题目描述，提交，我的提交，统计，编辑，测试数据。其中编辑和测试数据两个 tab 页仅管理员可见。
- 讨论模块
    其实就是讨论区，用户可在上面发帖评论。
- 状态模块
    用户提交题目的判题结果。
- 排行模块
    用户排名，有分组功能，便于老师统计结果
- 比赛模块
    核心模块之一，含有比赛列表和比赛详情页，比赛详情页有 6 个 tab 页，总览，题目，提交，状态，排名，编辑。其中编辑页仅管理员可见。
- 管理员模块
    核心模块之一，含有创建消息，创建题目，创建比赛，用户管理四个功能页。

给大家提前注册了一个普通用户账号，账号 123456 ，密码 123456 ，欢迎去试用一下。

## 前端
先来看看前端的项目结构，通过脚手架 vue-cli 构建
```
├── dist // 生成打包好的文件
│   ├── static
│   │   ├── css
│   │   ├── fonts
│   │   ├── img
│   │   └── js  
│   └── index.html
└── src
    ├── main.js // 项目入口
    ├── router // 路由文件，说明了各个路由将会使用的组件
    │   ├── index.js // router 的配置以及引用组件
    │   └── routes.js // 定义各个路由
    ├── assets // 网站 logo 图资源
    ├── components // 一些小组件
    ├── store // vuex 文件
    │   └── modules // 子模块
    ├── utils // js 工具方法
    └── views // 路由对应的组件 (这些组件在 router.js 中都被引入)
        ├── Admin
        ├── Contest
        ├── News
        └── Problem

```
前端一共有三十多张页面，但其实大多数都是只有图表，页面逻辑并不复杂。
iview 按需加载，减小前端打包大小。
为了保证首屏加载的速度，对部分路由进行懒加载。

```javascript
// 路由懒加载
const ProblemStatistics = r => require.ensure([], () => r(require('@/views/Problem/Statistics')), 'statistics')
const ProblemEdit = r => require.ensure([], () => r(require('@/views/Problem/ProblemEdit')), 'admin')
const Testcase = r => require.ensure([], () => r(require('@/views/Problem/Testcase')), 'admin')
const ContestEdit = r => require.ensure([], () => r(require('@/views/Contest/ContestEdit')), 'admin')
const NewsEdit = r => require.ensure([], () => r(require('@/views/News/NewsEdit')), 'admin')
const ProblemCreate = r => require.ensure([], () => r(require('@/views/Admin/ProblemCreate')), 'admin')
const ContestCreate = r => require.ensure([], () => r(require('@/views/Admin/ContestCreate')), 'admin')
const NewsCreate = r => require.ensure([], () => r(require('@/views/Admin/NewsCreate')), 'admin')
const UserManage = r => require.ensure([], () => r(require('@/views/Admin/UserManage/Usermanage')), 'admin')
const UserEdit = r => require.ensure([], () => r(require('@/views/Admin/UserManage/UserEdit')), 'admin')
const GroupEdit = r => require.ensure([], () => r(require('@/views/Admin/UserManage/GroupEdit')), 'admin')
const AdminEdit = r => require.ensure([], () => r(require('@/views/Admin/UserManage/AdminEdit')), 'admin')
const TagEdit = r => require.ensure([], () => r(require('@/views/Admin/UserManage/TagEdit')), 'admin')
```

同时前端用了不少第三方组件实现小的需求。
- **vue-echarts**: 基于 Vue 的 Echarts组件，在项目中用于展示提交结果的统计分析图。
- **vue2-editor**: 基于 Vue 的富文本编辑器，用于题目内容的编辑，支持图片上传等基本功能。
- **Vue.Draggable**: 基于 Vue 的拖拽组件，方便管理员对比赛题目顺序做改动。
- **vue-clipboard2**: 基于 Vue 的剪切板，方便用户复制代码。
- **vuex-router-sync**: 使 vue-router 的 $route 能够在 vuex 中的 state 访问到。
- **highlight.js**: 页面里代码高亮。

## 后端
```
├── config // 项目配置（数据库等）
├── model // 数据库 model
├── routes // 后端路由
├── controllers // 主要功能实现
├── services // 主要服务（判题、邮件提醒、更新）
├── utils // js 工具函数
├── test // 测试
├── app.js
└── manage.js

```

后端使用 koa2 开发，使用 async/await 代替回调，避免 callback hell. 主要数据都保存在 MongoDB 中，使用的 node 的 mongoose 包。为了避免多人同时提交题目，造成的高并发问题，接口遵循 RESTful 设计，使用 redis 对判题做了队列缓存。用户提交的题目会进入 redis 中，再一个个弹出队列交给判题端处理。正常 ACM 比赛最后一小时会进行封榜（不再进行排名和ac题目的更新，但是会更新用户的提交次数），在这里也用了 redis 对比赛排行榜进行更新，比赛过程中只将数据保存在 redis 中，并实现封榜，赛后再将比赛所有信息保存到 mongo 中。

```javascript
// 比赛时返回比赛排行榜
const ranklist = async (ctx) => {
  const contest = ctx.state.contest
  const ranklist = ctx.state.contest.ranklist
  let res
  const deadline = 60 * 60 * 1000
  await Promise.all(Object.keys(ranklist).map((uid) =>
    User
      .findOne({ uid })
      .exec()
      .then(user => { ranklist[user.uid].nick = user.nick })))

  if (Date.now() + deadline < contest.end) {
    // 若比赛未进入最后一小时，最新的 ranklist 推到 redis 里
    const str = JSON.stringify(ranklist)
    await redis.set(`oj:ranklist:${contest.cid}`, str) // 更新该比赛的最新排名信息
    res = ranklist
  } else if (!isAdmin(ctx.session.profile) &&
    Date.now() + deadline > contest.end &&
    Date.now() < contest.end) {
    // 比赛最后一小时封榜，普通用户只能看到题目提交的变化
    const mid = await redis.get(`oj:ranklist:${contest.cid}`) // 获取 redis 中该比赛的排名信息
    res = JSON.parse(mid)
    Object.entries(ranklist).map(([uid, problems]) => {
      Object.entries(problems).map(([pid, sub]) => {
        if (sub.wa < 0) {
          res[uid][pid] = {
            wa: sub.wa
          }
        }
      })
    })
    const str = JSON.stringify(res)
    await redis.set(`oj:ranklist:${contest.cid}`, str) // 将更新后的 ranklist 更新到 redis
    // 比赛结束
    res = ranklist
  }
  ctx.body = {
    ranklist: res
  }
}
```

项目使用 docker 进行一键部署。写了 Dockerfile 对 web 端进行镜像定制，在 docker-compose 中配置项目所需的所有镜像。[部署过程](https://github.com/acm309/PutongOJ)

## 最后
篇幅有限，无法展现更多的内容，有兴趣的话可以进入项目地址阅读源码，当然，如果觉得项目还不错的话 👏，就给个 star ⭐️ 鼓励一下吧~