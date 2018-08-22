---
title: vue-cli 使用
categories:
  - vuejs
tags:
  - vue
  - nodejs
  - vue-cli
abbrlink: 58b4b1b8
date: 2018-06-05 18:43:05
---



> 安装 vue-cli

```sh
# 全局安装
npm install -g vue-cli
```
如果安装失败，可以使用`npm cache clean`清理缓存，然后再重新安装



>  生成项目

首先需要在命令行中进入到工作区间，然后输入

```sh
vue init webpack vue-project
```
全局安装`webpack`命令`npm install -g webpack`

vue-project 是自定义的项目名称，命令执行之后，会在当前目录生成一个以该名称命名的项目文件夹



> proxyTable 代理设置

前后端分离项目本地测试，可以使用 proxyTable 代理到后台服务器，方便调试

```json
proxyTable: {
    '/api': {
        targer: 'https:localhost:8080/service/api',
        secure: false,
        changeOrigin: true,
        pathRewrite: {
            '^/api': '/'
        }
    }
}
```

使用时直接**/api/接口名**就行了 

