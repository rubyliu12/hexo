---
title: git 使用
categories:
  - Git
tags: []
abbrlink: 4896de77
date: 2017-07-07 08:35:33
---
# 基本命令

```sh
git add
git commit
git push
git status
git rm <-f>

git fetch
git marge
git pull
```



<!-- more -->



# fatal: remote origin already exists错误

先删除远程 Git 仓库
```sh
$ git remote rm origin
```
再添加远程 Git 仓库
```sh
$ git remote add origin https://github.com/xxx/xxxx.git
```

简易的命令行入门教程:

Git 全局设置:
```bash
git config --global user.name "xxxx"
git config --global user.email "xxxx@qq.com"
```
创建 git 仓库
```bash
mkdir ts
cd ts
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/xxx/xxxx.git
# git pull --allow-unrelated-histories
git push -u origin master
# git push --set-upstream origin master
```
已有项目?
```bash
cd existing_git_repo
git remote add origin https://github.com/xxx/xxxx.git
git push -u origin master
```



# git flow

分支共有5种类型

　　1) master，最终发布版本，整个项目中有且只有一个

　　2) develop，项目的开发分支，原则上项目中有且只有一个

　　3) feature，功能分支，用于开发一个新的功能

　　4) release，预发布版本，介于develop和master之间的一个版本，主要用于测试

　　5) hotfix，修复补丁，用于修复master上的bug，直接作用于master



# 回滚操作

```shell
# 查看记录
git log --oneline
# 回滚到上一次提交
git reset HEAD^
# 强制提交到远程仓库（会覆盖掉reset前的提交记录）
git push origin master -f
```



# fork后同步更新

```shell
git remote -v 
git remote add upstream git@github.com:xxx/xxx.git
git fetch upstream
git merge upstream/master
git push
```

