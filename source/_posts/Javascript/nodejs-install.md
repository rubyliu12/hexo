---
title: nodejs安装
date: 2017-07-07 08:43:05
categories: [Javascript]
tags: [Javascript, nodejs]
description:
permalink: nodejs
---
# NodeJS安装

### 配置淘宝npm
国内访问外网是很慢的，安装NodeJS是自带的npm地址默认是：http://registry.npmjs.org，访问很慢，所以一般换成国内的地址。

> 通过config命令

```bash
npm config set registry http://registry.cnpmjs.org npm info underscore
#（如果配置成功，这个命令会有字符串response）
```
<!-- more -->
> 命令行指定

```bash
npm --registry http://registry.cnpmjs.org info underscore
```
> 编辑node_modules/npm/.npmrc加入下面内容

```bash
registry = http://registry.cnpmjs.org
```

如果上面的npm地址不行的话，可以试试淘宝的npm
https://registry.npm.taobao.org


node_modules/npm/.npmrc
prefix=D:\MyTools\nodejs\node_global （修改全局路径）
cache=D:\MyTools\nodejs\node_global （修改全局路径）

### 修改默认模块下载路径
在安装完nodejs后，npm install -g jshint是被安装在默认路径（C:\Users\用户名\AppData\Roaming\npm）下的

可以通过修改nodejs安装路径下的node_modules/npm/.npmrc文件
prefix=D:\MyTools\nodejs\node_global （修改全局路径）
cache=D:\MyTools\nodejs\node_global_cache （修改全局路径）

请注意prefix、cache不能设置成一样的路径，否则通过npm安装模块时，会报错
