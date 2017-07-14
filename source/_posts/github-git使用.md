---
title: git使用
date: 2017-07-07 08:35:33
categories: [github]
tags: [git]
description:
permalink: git
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

### fatal: remote origin already exists错误
先删除远程 Git 仓库
```sh
$ git remote rm origin
```
再添加远程 Git 仓库
```sh
$ git remote add origin https://github.com/foreveryang321/dubbo-serialize.git
```
