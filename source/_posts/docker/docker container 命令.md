---
title: docker 命令
categories:
  - docker
tags:
  - container
abbrlink: 71631b3f
date: 2018-09-05 10:48:48
---



# docker container ls

> 列出所有容器

- 用法

  ```sh
  docker container ls [options]
  ```



<!-- more -->




- 选项

  | 参数           | 默认值 | 描述                             |
  | -------------- | ------ | -------------------------------- |
  | `-all, -a`     | false  | 显示所有容器(默认只显示运行的)   |
  | `--filter, -f` |        | 根据提供的条件过滤输出           |
  | `--format`     |        | 使用Go模板打印容器               |
  | `-last, -n`    | `-1`   | 显示最后创建的容器(包括所有状态) |
  | `--latest, -l` | false  | 显示最新创建的容器(包括所有状态) |
  | `-no-trunc`    | false  | 不要截断输出                     |
  | `--quiet, -q`  | false  | 只显示数字ID                     |
  | `-size, -s`    | false  | 显示文件大小                     |



# docker container rm

> 删除一个或多个容器

- 用法

  ```sh
  docker container rm [options] CONTAINER [CONTAINER...]
  ```

- 选项

  | 参数            | 默认值 | 描述                                |
  | --------------- | ------ | ----------------------------------- |
  | `--force, -f`   | false  | 强制移走正在运行的容器(使用SIGKILL) |
  | `--link, -l`    | false  | 删除指定的链接                      |
  | `--volumes, -v` | false  | 删除与容器关联的卷                  |



# 相关命令

| 命令                     | 描述                                     |
| ------------------------ | ---------------------------------------- |
| docker container attach  | 附加到正在运行的容器                     |
| docker container commit  | 从容器的更改创建一个新的映像             |
| docker container cp      | 在容器和本地文件系统之间复制文件/文件夹  |
| docker container create  | 创建一个新的容器                         |
| docker container diff    | 检查容器文件系统上文件或目录的更改       |
| docker container exec    | 在运行容器中运行命令                     |
| docker container export  | 将容器的文件系统导出为tar存档            |
| docker container inspect | 显示一个或多个容器的详细信息             |
| docker container kill    | 杀死一个或多个运行容器                   |
| docker container logs    | 获取容器的日志                           |
| docker container ls      | 列出容器                                 |
| docker container pause   | 暂停一个或多个容器内的所有进程           |
| docker container port    | 列出端口映射或容器的特定映射             |
| docker container prune   | 取出所有停止的容器                       |
| docker container rename  | 重命名容器                               |
| docker container restart | 重新启动一个或多个容器                   |
| docker container rm      | 删除(移除)一个或多个容器                 |
| docker container run     | 在新容器中运行命令                       |
| docker container start   | 启动一个或多个停止的容器                 |
| docker container stats   | 显示容器的实时流资源使用统计信息         |
| docker container stop    | 停止一个或多个运行容器                   |
| docker container top     | 显示容器的正在运行的进程                 |
| docker container unpause | 取消暂停一个或多个容器内的所有流程       |
| docker container update  | 更新一个或多个容器的配置                 |
| docker container wait    | 阻止一个或多个容器停止，然后打印退出代码 |