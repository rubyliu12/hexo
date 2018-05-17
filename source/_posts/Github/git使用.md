---
title: git使用
categories:
  - Github
tags:
  - git
abbrlink: 4896de77
date: 2017-07-07 08:35:33
---
# git初始使用

### 基本命令
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
### fatal: remote origin already exists错误
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
git push -u origin master
```
已有项目?
```bash
cd existing_git_repo
git remote add origin https://github.com/xxx/xxxx.git
git push -u origin master
```
