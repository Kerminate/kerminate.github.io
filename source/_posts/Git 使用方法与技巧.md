---
title: Git 使用方法与技巧
date: 2018-07-11 21:04:51
tags: 技术帖
---
在团队里工作了一段时间，逐渐适应了多人协作开发，这里总结一下 git 的使用方法与一些技巧。

## 1. 基本操作（正常提交流程）

```
git init   // 版本库初始化
git add -A  // 将本地修改的所有文件提交到暂存区
git commit -m "你的提交信息"  // 将暂存区文件提交到版本库
git push origin [你当前的分支]  // 将当前分支提交到远程仓库
```
<!--more-->

## 2. 分支管理
```
git checkout -b dev  // 创建新分支 dev，并切换到该分支
git branch  // 显示本地分支
git branch -a  // 显示所有分支(本地和远程)
git checkout master  // 切换到 master 分支
git merge dev  // 将 dev 分支合并到 master 分支
git branch -d dev  // 将本地的 dev 分支删除
git push origin -d dev  // 将远端的 dev 分支删除
```

## 3. 撤销回退 
### 使用 git reset
首先认识一下 `git status` 命令，显示当前暂存区的消息
本例中修改了 README.md，但是并没有 add
![](https://i.loli.net/2018/11/20/5bf3a7c6377cb.jpg)
之后 `git add -A` 将文件保存在暂存区，git status 查看此时暂存区状态

![](https://i.loli.net/2018/11/20/5bf3a7c79100d.jpg)

#### 撤销所有本地代码（git add 之前）
```
git checkout .  // 撤销所有修改
git clean -df  // 移除没有 track 的文件，-d 表示目录，-f 表示 force
```

#### git add 后撤销
```
git reset HEAD .  // 撤销所有 add 文件
git reset HEAD -filename  // 撤销单个 add 文件
```
通过 `git status` 查看暂存区内无状态，说明此时本地分支代码没有改动

#### git commit 后撤销
```
git reset --hard head^ // 回退到上个 commit 版本
```

##### 版本回退（git push 后撤销）
先使用 git log 查看信息，找到你想回退到版本的 id, 再通过 git reset 回退，可以清空中间的提交记录

```
git log
```
![](https://i.loli.net/2018/11/20/5bf3a7bb321f0.jpg)

```
git reset --hard 5aabd83035a2a92db15b95d0f43cb797288a915f
git push origin HEAD --force
```
这里 HEAD 指当前版本，把回退后的当前版本强行提交到远程仓库

#### git merge 后撤销
```
git checkout [行merge操作时所在的分支]
git reset --hard [merge前的版本号]
```

### 使用 git revert
- **git reset** 是直接删除指定的 commit，把HEAD 向后移动了一下
- **git revert** 是一次新的特殊的 commit，HEAD 继续前进，本质和普通 add commit 一样，仅仅是 commit 内容很特殊：提交的内容是与前面普通 commit 文本变化的反操作

假设按时间顺序依次有 commit1, commit2, commit3, commit4,对应有 version1, version2, version3, version4 四个状态.
revert 某次commit，最终文件状态为 commit4 前，即version3 状态)
```
git revert 57af6d9f0f8e4fdb646e86bd189c2346f6bd5458
```
![](https://i.loli.net/2018/11/20/5bf3a7bbda57e.png)

## 4. 标签管理
> 标签可以针对某一时间点的版本做标记，常用于版本发布。

```
git tag  // 输出所有标签
git tag v1.0  // 创建标签 v1.0
git show v1.0  // 查看标签 v1.0 的版本信息
git checkout v1.0  // 切换到 v1.0 标签
git tag -d v1.0  // 删除 v1.0 标签
git push origin v1.0  // 将标签 v1.0 提交到远程仓库
git push origin -tags  // 将所有标签都提交到远程仓库
```

## 5. 常用技巧
### 使用 git stash
当你正在进行项目开发，已经修改了部分代码，但是发现现有分支上有个 bug 要解决，但你又不想把代码 commit 到本地仓库，这时候可以使用 git stash 储存当前工作目录的中间状态。
可以直接使用 git stash 储存，也可以通过 git stash save "你的备注" 做一个message
我在本地修改了根目录的 README.md, 并删除了 tutorial 下的 readme.txt
```
git stash save "rm tutorial/readme.txt"
```
之后查看 stash 栈
```
git stash list
```
![](https://i.loli.net/2018/11/20/5bf3a7bb69ad0.jpg)
此时代码已经回到了你修改前的版本，本地修好 bug 后，将代码 push 到远端，再使用 git stash pop 或 git stash apply 来恢复之前的工作状态
- **git stash pop**: 这个指令将缓存堆栈中的第一个 stash 删除，并将对应修改应用到当前的工作目录下.
- **git stash apply**: 将缓存堆栈中的 stash 多次应用到工作目录中，但并不删除 stash 拷贝.使用该命令时可以通过名字指定使用哪个 stash，默认使用最近的 stash

示例：
```
git stash apply stash{0}  // 将 stash 栈里的第一个恢复
```

![](https://i.loli.net/2018/11/20/5bf3a7c6865c2.jpg)
恢复后会显示暂存区内文件的修改情况，与之前一致

### 使用 git rebase
多人协作时，使用 git rebase 代替 git merge 来变基，减少难看的 merge 类型的 commit
```
git stash
git pull —-rebase
git stash apply
## 手动解决冲突
git add -A
git rebase —-continue
## 如果此时提示 No rebase in progress? 则表示已经没有冲突了；否则上面两步要重复多次
git commit -m “xxx”
git push origin [branch] -f
```
其中
```
git pull = git fetch + git merge
git pull –-rebase = git fetch + git rebase
```
在合并分支时，直接使用 git rebase 代替 git merge

### fork 后的分支与源库同步
一般参与开源项目时，需要从源库中 fork 出一个分支，在本地开发好后再向源仓库发出 pull request，但是如何保证 fork 出的仓库与源库能保持同步。
以 https://github.com/dt-fe/weekly.git 这个仓库为例，
手动 fork 这个仓库到个人的 github 仓库下
```shell
git clone https://github.com/Kerminate/weekly.git  // clone 到本地
git remote add upstream https://github.com/dt-fe/weekly.git  // 指定源库
```
查看远程仓库信息
![](https://i.loli.net/2018/11/19/5bf28a00963e2.png)
```
git checkout -b dev  // 在新的分支上开发
git add -A
git commit -m "feat: add some change."
git push origin dev
```
之后手动向源库发起 PR，被 merge 之后，可以在本地 master 分支同步最新代码
```
git checkout master
git pull upstream master
git push
```

### .gitignore 失效
由于 .gitignore 只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改 .gitignore 是无效的。 要解决这个问题，需要先把本地缓存删除（改变成未track状态），然后再提交。
```
git rm -r --cached .
git add -A
git commit -m "feat: ignore xxx files"
```

## 6. 遇到的问题
本地创建一个分支推到远端之后，如果远端代码更新，执行 `git pull` 会失败，如图
![](https://i.loli.net/2018/11/20/5bf3a7c5d3183.png)
终端给出了 2 种方法，第一种 pull 的时候写上对应的远端分支和本地分支，形成映射关系，还有一种就是将本地分支和远程分支关联，这样以后就可以直接用 `git pull` 拉取最新代码
```
git pull <remote> <branch>  // 写上对应的映射分支
git branch --set-upstream-to=origin/<branch> <remote>  // 关联远端分支和本地分支
```
![](https://i.loli.net/2018/11/20/5bf3a7c6c8b56.png)
