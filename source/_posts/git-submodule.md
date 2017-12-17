---
title: Git Submodule
date: 2017-12-17 15:34:07
categories: Git
tags:
- git
permalink: git-submodule
---
Git Submodule 是一个很好的多项目使用共同类库的工具，他允许类库项目做为repository,子项目做为一个单独的git项目存在父项目中，子项目可以有自己的独立的commit，push，pull。而父项目以Submodule的形式包含子项目。<!--more-->

#### 一、在项目中使用Submodule
1.添加：

```
git submodule add git@github.com:jjz/pod-library.git pod-library
```
2.修改：

cd进入Submodule目录里，修改子目录文件，推送
```
git add.
git commit -m "xxx"
git push
```
返回父目录，查看，推送
```
git status
git commit -m "xxx"
git push
```

#### 二、更新Submodule（两种方式）

方式1.在父项目的目录下直接运行：
```
git submodule foreach git pull
```
方式2.直接在Submodule的目录下面更新：
```
>cd pod-library
git pull
```

#### 三、clone Submodule（两种方式）
方式1.采用递归参数clone整个project，submodule会被自动init并clone下来
```
git clone git@github.com:jjz/pod-project.git --recursive
```
方式2.先clone父项目，再init submodule
```
git clone git@github.com:jjz/pod-project.git
cd pod-project //submodule目录
git submodule init //初始化
git submodule update //更新
```

#### 四、删除Submodule
1.不支持直接删除Submodule,需要手动删除对应的文件:
```
cd pod-project
git rm --cached pod-library
rm -rf pod-library
rm .gitmodules
```

2.更改git的配置文件config，删除相关信息:

```
vim .git/config
[submodule "pod-library"]
  url = git@github.com:jjz/pod-library.git
```

3.提交到远程服务器
```
git commit -a -m 'remove pod-library submodule'
```

参考： [使用Git Submodule管理子模块](https://segmentfault.com/a/1190000003076028)


