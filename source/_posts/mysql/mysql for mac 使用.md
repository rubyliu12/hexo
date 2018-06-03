---
title: mysql for mac 使用
categories:
  - mysql
tags: []
abbrlink: 4ae0cbaa
date: 2018-05-25 09:35:05
---



# 配置环境变量

```sh
$ cd ~
# 如果没有 .bash_profile 文件，可以通过 touch 来创建一个
$ touch .bash_profile
# 编辑已经存在的文件，先打开文件
$ open -e .bash_profile
# 使配置生效
$ source .bash_profile
```

> 在 .base_profile 中加入如下内容

```sh
export PATH=$PATH:/usr/local/mysql/bin
```



<!-- more -->



# 修改root默认密码

```bash
$ mysqladmin -u root -p password "new password"
```



# 终端登录 mysql

```sh
$ mysql -u root -p
```

