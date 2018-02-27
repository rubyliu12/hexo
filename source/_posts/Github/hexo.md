---
title: hexo初体验
date: 2017-07-07 08:36:56
categories: [Github]
tags: [Github, hexo]
description:
permalink: hexo
---
# 安装部署hexo + github page

### 初体验
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

### 更换主题
```sh
cd themes
git clone https://github.com/litten/hexo-theme-yilia.git
```
配置主题 `_config.yml` 文件，修改里面的 `theme` 为 `hexo-theme-yilia`


### 配置github仓库
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

### 绑定域名
在你本地的 `hexo` 项目根目录的 `source` 目录下创建 `CNAME` 文件，并在 `CNAME` 中输入绑定的域名
