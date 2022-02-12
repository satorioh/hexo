---
title: Git worktree
date: 2020-01-22 10:32:52
categories: Git
tags:
- git
permalink: git-worktree
---
Git 2.5新增：从一个仓库中可以创建多个工作目录，方便多开编辑器并行开发

### 一、创建
##### 1.从已有分支创建新分支
```shell
git worktree add -b <新分支名> <新路径> <从此分支创建>
```
<!--more-->

##### 2.从已有分支直接创建
```shell
git worktree add <新路径> <从此分支创建>
```

### 二、查看
```shell
git worktree list
```

### 三、删除
##### 方法1:
```shell
git worktree remove <worktree路径>
```

##### 方法2：
```shell
rm -rf <新路径>
git worktree prune
```

##### 优点：
1.因为所有工作目录共享一个仓库，所以一个更新意味着整个更新（A 目录里对分支做的改动，B 目录里切到此分支也是改动后的；避免到时候找不到某个未推送的改动改到了哪个仓库）
2.可以快速进行并行开发，同一个项目多个分支同时并行演进
3.可避免node_modules冲突
4.相比单纯clone/copy，可使用命令更方便直观的管理

##### 注意：使用 git worktree 创建的多个目录，不能有任何两个目录在同一个分支下

参考文章：
[git worktree](https://git-scm.com/docs/git-worktree/2.24.0)
