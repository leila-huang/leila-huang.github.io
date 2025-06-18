---
title: git subtree使用
date: 2025-05-22 16:00:12
tags:
  - git
  - subtree
categories:
  - git
---

## 前言

不同的项目中出现了很多页面和组件都重复的地方，时间久了，进行代码优化就需要同步更新多个项目，容易出现遗漏的情况。故尝试使用 subtree 管理重复代码。

## 现在有哪些方案？

- [Git Submodule](https://link.juejin.cn?target=http%3A%2F%2Fgit-scm.com%2Fdocs%2Fgit-submodule)：这是 Git 官方以前的推荐方案
- [Git Subtree](https://link.juejin.cn?target=https%3A%2F%2Fmedium.com%2F%40porteneuve%2Fmastering-git-Subtrees-943d29a798ec)：从 [Git 1.5.2](https://link.juejin.cn?target=http%3A%2F%2Flwn.net%2FArticles%2F235109%2F) 开始，Git 新增并推荐使用这个功能来管理子项目
- [npm](https://link.juejin.cn?target=https%3A%2F%2Fwww.npmjs.com%2F)：node package manager，实际上不仅仅是 node 的包管理工具
- [composer](https://link.juejin.cn?target=https%3A%2F%2Fgetcomposer.org%2F)：暂且认为他是 php 版 npm、php 版 Maven 吧
- [bower](https://link.juejin.cn?target=http%3A%2F%2Fbower.io%2F)：针对浏览器前端的包管理工具（Web sites are made of lots of things — frameworks, libraries, assets, utilities, and rainbows. Bower manages all these things for you.）

## 使用方法

> 父项目称为 Px 项目；
>
> subtree 项目称为 S 项目；
>
> subtree 在父项目中的目录称为 S 目录

### 命名 subtree git 地址

```
git remote add  别名 S项目git地址
```

![image-20220708140043551](https://s2.loli.net/2022/07/08/EVMtos2Hnki8lIa.png)

### 将 S 项目加入到 P1 项目中

```shell
cd P1项目的路径
git subtree add --prefix=用来放S项目的相对路径 别名 xxx分支
```

命令执行后会把 S 项目的代码下载到 --prefix 所指定的目录，并在 P1 项目里自动产生一个 commit（就是把 S 目录的内容提交到 P1 项目里，里面也会有 S 项目的 push 记录）

![image-20220708135407684](https://s2.loli.net/2022/07/08/wCIghSVE1UtabAJ.png)

### 修改 S 项目内容后提交

先把**P1 项目代码提交到远程**后，再进行如下命令

```shell
cd P1项目的路径
git subtree push --prefix=S项目的路径 别名 xxx分支
```

### 拉取 S 项目最新代码

```
git subtree pull --prefix=S项目的路径 别名 xxx分支
```

### split：从子树中提取一个新的合成项目历史记录

> 目的是提升 subtree push 效率，结合图片理解

```
git subtree split --prefix=S项目的路径 --rejoin
git subtree push -P S项目的路径 别名 xxx分支
```

1. 图中提交了三次 git，只有中间提交是改动了 subtree![image-20220712154810842](https://s2.loli.net/2022/07/12/nWOZjaplvLJB2mK.png)

2. split&push 后，subtree 的 tree 是这个样子，如果没有 split,那在这里能看到的提交记录将会有三个

   ![image-20220712154958890](https://s2.loli.net/2022/07/12/hqi5Fj3MaW2y1Ew.png)

## 总结

目前看来对于重复 **页面 组件 hook** 之类管理上很方便，但如果多个项目中重复代码有差异性，或是并不完全引入 subtree 内容时，会出现引入多余代码的问题。

这个时候如何管理好呢？是单独切分支管理还是？待后续实践后发现。

## 参考链接

1. 用 Git Subtree 在多个 Git 项目间双向同步子项目：https://juejin.cn/post/6844903762176262157
1. Git subtree 用法与常见问题分析：https://juejin.cn/post/6881580754854215687#heading-14
