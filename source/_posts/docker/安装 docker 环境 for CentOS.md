---
title: 安装 docker 环境 for CentOS
categories:
  - docker
tags:
  - docker-compose
abbrlink: 2fff8f5e
date: 2018-09-05 10:48:48
---

# 安装 docker
- 删除老版本

  ```sh
  $ sudo yum remove docker \
                    docker-client \
                    docker-client-latest \
                    docker-common \
                    docker-latest \
                    docker-latest-logrotate \
                    docker-logrotate \
                    docker-selinux \
                    docker-engine-selinux \
                    docker-engine
  ```





<!-- more -->




- 配置  repo

  ```sh
  $ sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
    
  $ sudo yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo
  
  # Optional
  $ sudo yum-config-manager --enable docker-ce-edge
  $ sudo yum-config-manager --enable docker-ce-test
  $ sudo yum-config-manager --disable docker-ce-edge
  ```


- 安装 docker

  ```sh
  $ sudo yum install docker-ce
  ```


- - 安装指定版本 docker

    ```sh
    # List and sort the versions available in your repo
    # 可以查看所有仓库中所有docker版本，并选择特定版本安装
    $ yum list docker-ce --showduplicates | sort -r
    docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
    $ sudo yum install docker-ce-<VERSION STRING>
    ```


- 启动并加入开机启动

  ```sh
  # 启动 docker
  $ sudo systemctl start docker
  # 停止运行 docker
  $ sudo systemctl stop docker
  # 开机启动
  $ sudo systemctl enable docker
  # 禁止开机启动
  $ sudo systemctl disable docker
  ```



- 验证安装

  ```sh
  $ docker version
  Client:
   Version:           18.06.1-ce
   API version:       1.38
   Go version:        go1.10.3
   Git commit:        e68fc7a
   Built:             Tue Aug 21 17:23:03 2018
   OS/Arch:           linux/amd64
   Experimental:      false
  
  Server:
   Engine:
    Version:          18.06.1-ce
    API version:      1.38 (minimum version 1.12)
    Go version:       go1.10.3
    Git commit:       e68fc7a
    Built:            Tue Aug 21 17:25:29 2018
    OS/Arch:          linux/amd64
    Experimental:     false
  ```


# docker-compose

## 安装

- 安装

  ```sh
  $ sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
  ```


- 可执行权限

  ```sh
  $ sudo chmod +x /usr/local/bin/docker-compose
  ```


- 验证安装

  ```sh
  $ docker-compose --version
  docker-compose version 1.22.0, build 1719ceb
  ```
