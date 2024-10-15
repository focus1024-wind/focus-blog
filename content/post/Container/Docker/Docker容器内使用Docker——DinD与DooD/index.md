---
title: Docker容器内使用Docker——DinD与DooD
description: Docker容器内使用Docker——DinD与DooD
date: 2023-09-13
slug: container/docker/dind_and_dood
image: 
categories:
    - Container
    - Docker
tags:
    - Container
    - Docker
    - Docker in Docker(DinD)
    - DOcker outside of Docker(DooD)
---

# DinD与DooD简介
在部分场景中，我们需要在Docker容器内操作Docker镜像。如，容器内实现对资源的监控、服务的打包、自动化构建等操作，这些操作都需要能够与Docker服务端实现交互来实现的。

在Docker容器内操作Docker有两种模式，分别为DinD（Docker in Docker）：在Docker容器内部运行独立的Docker进程；DooD（Docker outside of Docker）：运行在Docker容器外部的Docker，即在容器内部操作外部的Docker服务端。

**DinD与DooD对比：**

| | DinD | DooD |
| ---- | ---- | ---- |
| 原理 | 在Docker容器内部安装独立的Docker客户端与服务端，实现容器内独立的的Docker操作 | 在Docker容器内部安装独立的Docker客户端，将外部Docker服务端的`.sock`文件挂载在容器内部，通过内部客户端与挂载的`.sock`文件实现Docker操作 |
| 优点 | Docker服务端之间相互隔离，无干扰，缓存，适合需要独立隔离的环境 | 能在容器内部实现与外部Docker服务端的交互，能够利用外部Docker外部服务端现有的镜像，缓存等资源 |
| 缺点 | 无法利用缓存，Docker容器销毁后环境镜像丢失 | 容器内部的操作会对外部Docker造成影响。不同容器可以查看同一宿主机的Docker daemon守护进程，并且不同实例可以修改统一Docker daemon下的镜像，存在安全问题。多个实例之间存在环境的争抢问题。 |

## DinD与DooD的应用场景
当我们需要在容器内实现对Docker外部资源的监控时，需要在容器内有一定的手段能够访问到外部的Docker服务端，实现与外部Docker服务端进行通讯，获取到Docker服务端到信息实现对资源的监控。如，常用的监控工具Prometheus，Docker监控管理面板Portainer这些都是可以通过DooD的方式实现内部容器监控外部Docker资源情况。
在一些场景中，我们需要实现CI/CD等自动化集成等操作如Jenkins时，如果采用容器的方式部署，需要在如集成容器内部进行容器打包的情况下，可以采用DinD或者DooD的模式，实现容器内部的镜像打包登操作。  

# DooD构建Docker镜像
由于Docker对容器来运行Docker的做法做出一定限制，所以无法直接在Docker容器内按照Docker。如我们参考Docker安装文档的方式在容器内部安装Docker，只会安装Docker客户端。

## 构建DooD镜像
以Ubuntu:22.04镜像为基镜像，参考[Docker引擎安装官方文档](https://docs.docker.com/engine/install/ubuntu/)，构建DooD镜像。

**DooD镜像Dockerfile：**
```shell
FROM ubuntu:22.04

RUN \
    # Add Docker's official GPG key:
    apt update && \
    apt install -y ca-certificates curl gnupg && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
    chmod a+r /etc/apt/keyrings/docker.gpg && \
    # Add the repository to Apt sources:
    echo \
        "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
        "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
        tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt update && \
    # Install the Docker packages.
    apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

CMD [ "bash" ]
```

**DooD镜像的构建命令：**
```shell
docker build . \
    --file ./Dockerfile \
    --tag "ubuntu:dood"
```

## 运行DooD镜像
### 直接运行Docker
**打包后直接运行DooD镜像：**
```shell
docker run -it ubuntu:dood bash
```

**调用Docker命令：**
```shell
root@17f49a00974d:/# docker version
Client: Docker Engine - Community
 Version:           24.0.6
 API version:       1.43
 Go version:        go1.20.7
 Git commit:        ed223bc
 Built:             Mon Sep  4 12:31:44 2023
 OS/Arch:           linux/amd64
 Context:           default
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```
如上命令所示，调用`docker version`后。Docker客户端日志信息输出正常，但是服务端却显示`Cannot connect to the Docker daemon at unix:///var/run/docker.sock.`。我们平常在Docker客户端执行的命令实际上是Docker的客户端进程，实际上Docker如打包镜像，运行容器，拉取镜像登操作都是由Docker服务端操作的，Docker客户端实现通过命令来调用服务端。

### DooD模式运行Docker
通过`.sock`后缀可以看到，Docker的客户端和服务端实际上是通过socket机制进行的通讯。在采用DooD模式的情况下，我们可以讲外部的Docker服务端socket文件挂载在内部容器中，实现在容器内部，借助外部socket进行通讯，完成Docker调用。

**DooD模式运行DooD镜像：**
```shell
docker run -it -v /var/run/docker.sock:/var/run/docker.sock ubuntu:dood bash
```

**调用Docker命令：**
```shell
root@ff54c629edd0:/# docker version
Client: Docker Engine - Community
 Version:           24.0.6
 API version:       1.41 (downgraded from 1.43)
 Go version:        go1.20.7
 Git commit:        ed223bc
 Built:             Mon Sep  4 12:31:44 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Desktop 4.17.0 (99724)
 Engine:
  Version:          20.10.23
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.18.10
  Git commit:       6051f14
  Built:            Thu Jan 19 17:32:04 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.18
  GitCommit:        2456e983eb9e37e47538f59ea18f2043c9a73640
 runc:
  Version:          1.1.4
  GitCommit:        v1.1.4-0-g5fd4c4d
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
如上命令所示，通过挂载`/var/run/docker.sock`文件后启动DooD镜像，调用Docker命令不仅打印出容器内部Docker客户端的日志，也打印出了外部Docker服务端的日志。此时，我们在容器内部调用Docker命令，实际上就是交由外部的Docker服务端来进行操作，所以在容器内部的Docker操作会同步到外部宿主机中。

# DinD构建Docker镜像
## 通过官方dind镜像使用dind模式的docker
由于Docker的限制，不可以直接在Docker里面安装Docker服务端，但是Docker官方有提供构建好的DinD镜像`docker:dind`。

由于docker本质上是一种隔离技术，在Docker容器内部仍需要访问到外部的Linux资源。在DinD模式下运行Docker，也需要访问到外部Linux内核中的资源，有些资源需要root权限才可以访问。所以在启动DinD镜像时，需要添加`--privileged`参数，开启Docker容器到特权模式，否则容器内部无法访问特定资源。

**启动官方DinD镜像：**
```shell
docker run -it --privileged docker:dind bash
```

## 本地构建DinD镜像
由于Docker本身只是一种服务隔离技术，并不是传统意义上的虚拟机，并不能完全的虚拟化模拟软硬件资源，在容器的运行中还是需要访问外部资源，且由于Docker底层采用UnionFS文件系统。所以无法通过安装的方式安装Docker服务端。Docker提供有二进制免安装文件，通过二进制免安装文件，可以在一定程度上不完全依赖外部的环境，实现Docker内部访问Docker资源。

访问[GitHub的docker镜像构建仓库](https://github.com/docker-library/docker)阅读代码可知，Docker官方的dind镜像也是通过二进制文件实现的。

Docker在使用时，需要依赖一些证书和库，所以我们可以参考[Docker引擎安装官方文档](https://docs.docker.com/engine/install/ubuntu/)添加必要的仓库但不执行后续的Docker安装操作，然后安装二进制运行依赖库即可。在`ubuntu:22.04`镜像中，只需要额外安装`iptables`库即可。

### 下载解压二进制Docker文件
- [二进制安装文件官方文档: https://docs.docker.com/engine/install/binaries/](https://docs.docker.com/engine/install/binaries/)
- [二进制安装文件安装路径: https://download.docker.com/linux/static/stable/](https://download.docker.com/linux/static/stable/)

Docker官方提供了Docker的二进制文件供无网络或免安装环境下使用，可在Docker官网提供的下载地址下载对应版本的Dokcer二进制包tgz文件。

**以docker-24.0.6.tgz为例，进行下载解压**：
```shell
# Docker二进制文件下载
wget https://download.docker.com/linux/static/stable/x86_64/docker-24.0.6.tgz

# Docker二进制文件解压
docker -xzvf docker-24.0.6.tgz
```

- Docker软件分为客户端进程和服务端进程，Docker的二进制软件同样也有服务端进程`dockerd`，`containerd`
	- `dockerd`和`containerd`进程需要以守护进程的方式在后端运行
	- `dockerd`进程依赖于`containerd`和`runc`进程
		- 需要先以`nohup command &`的方式启动`containerd`和`runc`进程
		- 确保依赖库安装后，依赖进程启动后，以`nohup command &`方式启动`dockerd`进程
- `dockerd`服务端守护进程启动后，即可通过`docker`二进制文件操作docker

### 构建DinD镜像
以`ubuntu:22.04`镜像为基，构建dind镜像

**DinD镜像Dockerfile：**
```shell
FROM ubuntu:22.04

COPY ./docker /data/docker

WORKDIR /data/docker

RUN \
    # Add Docker's official GPG key:
    apt update && \
    apt install -y ca-certificates curl gnupg && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
    chmod a+r /etc/apt/keyrings/docker.gpg && \
    # Add the repository to Apt sources:
    echo \
        "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
        "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
        tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt update && \
    apt install -y iptables

# Add docker to environment
ENV PATH="/data/docker:${PATH}"

CMD [ "bash" ]
```

**DinD镜像打包命令：**
```shell
docker build . \
    --file ./Dockerfile \
    --tag "ubuntu:dind"
```

## 运行DinD镜像
DinD镜像的运行需要在特权模式下运行。同时，启动容器后，需要依次以守护进程的方式启动`runc`，`containerd`，`dockerd`等进程，才可以以DinD的模式访问Docker。

**特权模式启动dind镜像：**
```shell
docker run -it --privileged ubuntu:dind bash
```

**启动守护进程：**
```shell
nohup runc &
nohup containerd &
nohup dockerd &
```

**运行Docker命令：**
```shell
root@a09d8ec17a42:/data/docker# docker version
Client:
 Version:           23.0.6
 API version:       1.42
 Go version:        go1.19.9
 Git commit:        ef23cbc
 Built:             Fri May  5 21:13:15 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          23.0.6
  API version:      1.42 (minimum version 1.12)
  Go version:       go1.19.9
  Git commit:       9dbdbd4
  Built:            Fri May  5 21:14:22 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.6.21
  GitCommit:        3dce8eb055cbb6872793272b4f20ed16117344f8
 runc:
  Version:          1.1.7
  GitCommit:        v1.1.7-0-g860f061
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
如上命令所示，我们即打印了Docker的客户端和服务端信息。同时，当我们拉取镜像操作时，也可以看到，外部的Docker并不会发生变化，即通过DinD模式成功运行Docker，实现与外部环境的隔离。
