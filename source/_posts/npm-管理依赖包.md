---
title: npm 管理依赖包
date: 2018-12-12 14:46:11
tags: Node
categories: Node
---
由于 npm 包的升级迭代速度很快，经常会出现 npm 依赖包过期或者安全性的问题。
如果项目是在 github 管理，它会在仓库首页提醒项目中存在有隐患的依赖。

![](https://i.loli.net/2018/12/12/5c10b0976c20c.png)

<!--more-->

可以点击进去查看，它会提示你哪个依赖过期。

![](https://i.loli.net/2018/12/12/5c10b78adbf03.png)

由于在 npm5.0 版本以上，npm install 都会生成一份 package-lock.json 文件，package.json 里面依赖的包如果自身依赖其他包，都会在 package-lock.json 里面有详细信息。但是你无法清晰地知道各个依赖包之间的关系。
使用下面的命令可以清晰地得到依赖树。
```
npm ls
```
![](https://i.loli.net/2018/12/12/5c10b7ed67f9c.png)

可以从中看出 package-lock.json 中包和 package.json 包的依赖关系

如果感觉信息太多，也可以单独看 package.json 中某个包的依赖情况
```
npm view hexo-deployer-git dependencies 
```
![](https://i.loli.net/2018/12/12/5c10bfbc4e39c.png)

只要找到 package-lock.json 中有问题的依赖对应的包，就可以手动最新的包。在 npm 包或对应仓库查看最新的 release 版本
```
npm install hexo-server@0.3.1
```

如果该包已完全失效，需要换成其他包，也可直接卸载
```
npm uninstall hexo-server
```

当然每次手动更新包会显得非常繁琐，npm 也提供了自动机制,使用下面的命令检查过期的依赖包
```
npm outdated
```
![](https://i.loli.net/2018/12/12/5c10c2758613b.png)

这个也有缺点，查出来的不一定是过期的，而且只能查出 package.json 里不是 latest 版本的包，无法判断出 package-lock.json 里有问题的依赖。

之后可以更新过期依赖包
```
npm update
```

如果 update 过程中出现报错，就先把 node_modules 文件夹删除，重新 `npm install`，之后再 `npm update`

如果 npm 版本已经更新到 6.0 以上，每次安装好依赖，npm 都会对依赖包进行安全审核，如果有问题，会在控制台显示

![](https://i.loli.net/2018/12/12/5c10c6104eaa9.png)

可以查看具体过期依赖详情
```
npm audit
```
![](https://i.loli.net/2018/12/12/5c10c6d7396d7.png)

之后可以直接修复依赖
```
npm audit fix
```
正常情况下，此时所有的包依赖都已经更新完毕。

但是由于很多包都是个人管理，会导致没有及时更新的情况，所以在以上操作结束后可能还是没有更新完所有依赖。

![](https://i.loli.net/2018/12/12/5c10ca5b49d9b.png)

此时只能去看仓库源码，如果源码中引的依赖也有问题的话，那就只能提 issue 或者 也可以自行修复提一个 PR 合到该仓库。