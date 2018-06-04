---
layout: post
title: Git fetch & pull
categories: [TOOLS]
description: 简单工厂属于创造模型类
keywords: Git, TOOLS
---

# Git fetch & pull
fetch 和 pull 存在着较大的区别的！简单的讲就是: `git pull = git fetch + merge to local`.但细化起来怎么都不简单的啊!


## git pull 和 git fetch 有什么区别？

### 1. 先理清某些概念
首先，你的每一个操作都是要指明【来源】和【目标】的，而对于 pull 来说，【目标】就是当前分支

其次，你得清楚 git 是有 tracking 的概念的，所谓 tracking 就是把【来源】和【目标】绑定在一起，节省一些操作是需要输入的参数。tracking 只能是一对一的，没有一个 local branch tracking 多个 remote branch 这么一说，但是多个 remote 是有的。

### 2.案例分析:简单情况
假设有 master 和 develop 都是 tracking 了的，于是：

```sh
# 当你在 master 下
$ git pull
# 等于 fetch origin，然后 merge origin/master

# 当你在 develop 下
$ git pull
# 等于 fetch origin，然后 merge origin/develop
```

若只有一个 remote，假设叫 origin，那么 `git pull` 等价于 g`it pull origin`；平时养成好习惯，没谱的时候都把 【来源】带上。


但是，如果我要合并 origin/master 去 develop 呢？

```sh
# 当你在 master 下
$ git checkout develop # 切换到 develop，这就是 【目标】
$ git pull origin master  # 合并 origin/master，这就是 【来源】
```


### 3.案例分析:复杂情况
若你有多个 remote ，git pull [remote name] 所做的事情是：

1. fetch [remote name] 的所有分支
2. 寻找本地分支有没有 tracking 这些分支的，若有则 merge 这些分支，若没有则 merge 当前分支

## 总结

1. `git pull = git fetch + merge`
2. `git fetch` 拿到了远程所有分支的更新，用 `cat .git/FETCH_HEAD `可以看到其状态，若都是 `not-for-merge` 则不会有接下来的 `merge` 动作
3. `merge` 动作的默认目标是当前分支，若要切换目标，可以直接`checkout`切换分支
4. `merge` 动作的来源则取决于你是否有 `tracking`，若有则读取配置自动完成，若无则请指明【来源】

## 参考
1. [git pull 和 git fetch 有什么区别？](https://ruby-china.org/topics/15729)