---
title: nodejs 使用
categories:
  - nodejs
tags:
  - npm
  - cnpm
abbrlink: 8d0f4649
date: 2017-07-07 08:43:05
---


# 配置淘宝npm

国内访问外网是很慢的，安装NodeJS是自带的npm地址默认是：http://registry.npmjs.org，访问很慢，所以一般换成国内镜像地址。

> 通过config命令

```bash
npm config set registry https://registry.npm.taobao.org/ npm info underscore
#（如果配置成功，这个命令会有字符串response）
npm --registry=https://registry.npm.taobao.org
```


<!-- more -->



> 命令行指定

```bash
npm --registry https://registry.npm.taobao.org/ info underscore
```
> 编辑node_modules/npm/.npmrc加入下面内容

```bash
registry = https://registry.npm.taobao.org/
```

如果上面的npm地址不行的话，可以试试淘宝的npm
https://registry.npm.taobao.org

```sh
npm set registry https://registry.npm.taobao.org/
# 或者
npm --registry https://registry.npm.taobao.org/ info underscore
```





# 下载路径

在安装完nodejs后，`npm install -g jshint`是被安装在默认路径（C:\Users\用户名\AppData\Roaming\npm）下的

可以通过修改nodejs安装路径下的node_modules/npm/.npmrc文件
prefix=D:\MyTools\nodejs\node_global （修改全局路径）
cache=D:\MyTools\nodejs\node_cache （修改全局路径）

请注意prefix、cache不能设置成一样的路径，否则通过npm安装模块时，会报错



# npm 使用

```sh
# -S 是--save 的缩写，它可以让你安装的模块记录到package.json文件当中
npm install -S xxx

# 这是删除模块，也可以删除全局模块
npm uninstall xxx
npm uninstall -g xxx

# -D就是--save-dev 这样安装的包的名称及版本号就会存在package.json的devDependencies这个里面，而--save会将包的名称及版本号放在dependencies里面
npm install -D xxx

```



# npm registry 设置

```shell
$ npm install -g nrm
$ nrm ls
D:\Workspace\vuejs-project>nrm ls

* npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
  taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  rednpm - http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/
# 切换源
$ nrm use taobao
Registry has been set to: https://registry.npm.taobao.org/
```



# npm-check检查依赖包版本

```shell
# 安装 npm-check
npm install -g npm-check
# 检查 npm 包版本
npm-check -u -g
npm-check -u -S
npm-check -u -D
```

