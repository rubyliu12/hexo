---
title: docker 安装 redis
categories:
  - redis
tags:
  - docker
  - docker-compose
abbrlink: 6fffbcc4
date: 2018-09-05 14:32:48
---



- 拉取镜像

  ```sh
  # 默认拉取 redis:latest 版本
  $ docker pull redis
  
  # 拉取指定版本镜像
  $ docker pull redis:4.0.11
  ```



<!-- more -->




- 运行 redis 容器

  ```sh
  # 默认运行 redis:latest 版本
  $ docker run -p 6379:6379 --name redis -d redis
  
  # 运行指定版本 redis
  $ docker run -p 6379:6379 --name redis -d redis:4.0.11
  ```

- 使用 docker-compose

  ```yaml
  # 默认运行 redis:latest 版本
  redis:
    image: 'redis'
    ports:
      - '6379:6379'
  
  # 运行指定版本 redis
  redis:
    image: 'redis:4.0.11'
    ports:
      - '6379:6379'
  ```

  ```sh
  $ docker-compose up -d redis
  ```
