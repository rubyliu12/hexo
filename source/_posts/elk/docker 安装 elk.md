---
title: docker 安装 elk 环境
categories:
  - elk
tags:
  - docker
  - docker-compose
abbrlink: 6a0c4822
date: 2018-09-05 11:32:48
---



- 安装

  > 从 Docker 仓库中拉取镜像

  ```sh
  $ sudo docker pull sebp/elk
  ```

  > 拉取指定版本镜像

  ```sh
  sudo docker pull sebp/elk:E1L1K4
  ```


- 使用

- - docker 命令运行容器

    ```sh
    $ docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it --name elk -d sebp/elk
    ```

    > -d 表示后台运行

  - Docker Compose 命令运行容器

    docker-compose.yml

    ```yaml
    elk:
      image: sebp/elk
      ports:
        - "5601:5601"
        - "9200:9200"
        - "5044:5044"
    ```

    ```sh
    $ docker-compose up -d elk
    ```


- 进入容器命令行

  ```sh
  $ docker exec -it <container-name> /bin/bash
  ```
