---
title: Git学习笔记
date: 2015-09-05 10:14:03
categories: Git
tags:
- git
permalink: git-notes
---
### 一、安装
```
yum install -y git
```

### 二、配置用户信息及颜色高亮【本地仓库可选】
```
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
git config --global color.ui true
```
<!--more-->

### 三、创建仓库repository【建议全英文路径】
```
mkdir learngit
cd learngit
pwd
/Users/michael/learngit
```

### 四、常用命令
1. 版本库/仓库初始化：`git init`(先cd到目录下)
2. 把文件添加到仓库：`git add readme.txt`
3. 完成提交：`git commit -m "add readme.txt"`
4. 查看仓库当前状态：`git status`
5. 对比文件前后修改：`git diff readme.txt`

### 五、版本切换
1. 查看提交历史：`git log 或 git log --pretty=oneline --abbrev-commit`
2. 退回上个版本：`git reset --hard HEAD^`
3. 退回上上个版本：`git reset --hard HEAD^^`
4. 退回指定ID对应的版本：`git reset --hard commit_id`
5. 查看命令历史：`git reflog`
6. 查看工作区和最终版本库（最新版）里的区别：`git diff HEAD -- readme.txt`

### 六、工作区（Working directory）、暂存区(stage)、版本库(repository)的后悔药
1. 版本库回撤（已git commit）：`git reset --hard commit_id`
2. 暂存区回撤（已git add）: `git reset HEAD filename`
3. 工作区回撤（已修改，未git add过）：`git checkout -- filename`(实际是用版本库里的版本替换工作区的版本)
4. 删除文件：`git rm filename`（到暂存区），`git commit`后到repo生效

### 七、远程仓库操作（Github为例）
1. 生成ssh密钥：用户主目录下输入`ssh-keygen -t rsa -C "email@example.com"`
2. .ssh目录下可看到id_rsa（私钥）、id_rsa.pub（公钥）
3. Github上增加ssh key，填入上述公钥内容
4. 新建Github仓库，名称learngit
5. 关联远程仓库:`git remote add origin git@github.com:satorioh/learngit.git`
6. 第一次推送本地仓库到远程，并关联同步：`git push -u origin master`（后续不用加参数u）
7. 克隆远程仓库：`git clone git@github.com:satorioh/gitskills.git`
8. 查看远程库信息：`git remote -v`
9. 本地创建和远程分支对应的分支(名称最好一致)：`git checkout -b branch-name origin/branch-name`
10. 建立本地分支与远程分支的关联：`git branch --set-upstream origin/branch-name branch-name`
11. 抓取远程分支：`git pull`

### 八、分支管理
1. 创建分支：`git branch dev`
2. 切换分支：`git checkout dev`
3. 查看当前分支：`git branch`
4. 合并分支：`git merge dev(ff模式)、git merge --no-ff -m "" dev`(no ff普通模式)
5. 删除分支：`git branch -d dev`
6. 冷冻当前工作区内容：`git stash`
7. 查看冷冻的内容：`git stash list`
8. 解冻内容：`git stash apply stash@{0}`(不丢弃stash)，`git stash pop`(同时丢弃stash)
9. 删除未合并的分支：`git branch -D <name>`

### 九、标签管理
1. 新建标签：`git tag v1.0`(默认为HEAD)
2. 查看所有标签：`git tag`
3. 查看某个标签的详细内容：`git show v1.0`
4. 推送一个本地标签：`git push origin <tagname>`
5. 推送全部未推送过的本地标签：`git push origin --tags`

### 十、Git配置文件
1. 仓库的Git配置文件：.git/config（ls -al查看）文件
2. 当前用户的Git配置文件：用户主目录的.gitconfig文件中


参考文献

 - [廖雪峰的Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

