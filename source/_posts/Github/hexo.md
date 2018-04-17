---
title: hexo初体验
date: 2017-07-07 08:36:56
categories: [Github]
tags: [Github, hexo]
description:
permalink: hexo
---
# node.js

## 安装node.js
去官网现在并安装，这里安装路径选到D:\nodejs

## 改变原有的环境变量
我们要先配置npm的全局模块的存放路径以及cache的路径，例如我希望将以上两个文件夹放在NodeJS的主目录下，便在NodeJs下建立"node_global"及"node_cache"两个文件夹，输入以下命令改变npm配置
```shel
npm config set prefix "D:\nodejs\node_global"
npm config set cache "D:\nodejs\node_cache"
```
这个不配置也可以？（在系统环境变量添加系统变量NODE_PATH，输入路径D:\nodejs\node_global\node_modules）
此后所安装的模块都会安装到改路径下

## 安装淘宝npm（cnpm）
### 输入以下命令
```shel
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
### 输入cnpm -v输入是否正常
```shel
cnpm -v
```
<!-- more -->

# 安装部署hexo + github page

## 初体验
```sh
npm install -g hexo-cli
```

本地目录 `hexo` 文件夹，进去这个文件夹，依次执行下面的命令
```sh
cd hexo
hexo init
# Hexo随后会自动在目标文件夹建立网站所需要的文件
npm install
```
<!-- more -->
启动本地Hexo服务
```sh
hexo server
```

Create a new post
```sh
hexo new 'blog name'
```
在hexo new文章时，需要stop server，否则会出现2次这个文章

执行下面的命令，将markdown文件生成静态网页
```sh
hexo generate
```

## 更换主题
```sh
cd themes
git clone https://github.com/litten/hexo-theme-yilia.git
```
配置主题 `_config.yml` 文件，修改里面的 `theme` 为 `hexo-theme-yilia`


## 配置github仓库
在`_config.yml` 文件最后添加如下内容
```yml
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/xxx/xxx.github.io.git
  # ssh模式
  #repository: https://github.com/xxx/xxx.github.io.git
  branch: master
```
配置好后，一次执行
```sh
hexo clean
hexo g
hexo d
```
出现 `Deploy done: git` 说明配置成功

如果提示 `ERROR Deployer not found: git`，则要安装 `hexo-deployer-git` 插件
```sh
cd hexo
npm install hexo-deployer-git --save
```

部署到github，每次部署可以执行一下命令
```sh
hexo clean
hexo generate
hexo deploy
```

## 绑定域名
在你本地的 `hexo` 项目根目录的 `source` 目录下创建 `CNAME` 文件，并在 `CNAME` 中输入绑定的域名

## 使用问题
### hexo d出现错误
执行hexo clean && hexo g && hexo d出现一下错误，多试几次又成功了，之后没有继续追查问题
```
fatal: TaskCanceledException encountered.
   ▒▒ȡ▒▒һ▒▒▒▒▒▒
bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': No error
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: fatal: TaskCanceledException encountered.
   ��ȡ��һ��������
bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': No error

    at ChildProcess.<anonymous> (D:\Workspace\foreveryang321.github.io\node_modules\hexo-util\lib\spawn.js:37:17)
    at emitTwo (events.js:106:13)
    at ChildProcess.emit (events.js:191:7)
    at ChildProcess.cp.emit (D:\Workspace\foreveryang321.github.io\node_modules\cross-spawn\lib\enoent.js:40:29)
    at maybeClose (internal/child_process.js:886:16)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:226:5)
```
