---
title: Docker Swarm网络管理
description: Docker Swarm网络管理
date: 2024-01-15
slug: container/swarm/swarm_network
image: 
categories:
    - Container
    - Swarm
    - Docker
    - Network
tags:
    - Container
    - Swarm
    - Docker
    - Network
---

# Docker Swarm网络
在初始化Docker Swarm后，观察Docker网络，可以发现Docker网络中多了`docker_gwbridge`、`ingress`等网络，以Docker Swarm部署一个服务后，会发现在Docker中默认创建了一个类型为overlay的网络
- docker_gwbridge: Docker Swarm默认桥接网络，将ingress和overlay类型网络连接到桥接网络，默认情况下，服务运行的每个容器都连接到其本地 Docker 守护程序主机的docker_gwbridge网络中。
- ingress: 一种特殊类型的overlay网络，有助于服务节点之间的负载平衡。
  - 当任何集群节点在已发布的端口上收到请求时，它会将请求传递给IPVS模块。IPVS跟踪参与该服务的所有IP地址，选择其中一个，然后通过ingress网络将请求路由到该地址。
- overlay: Overlay网络参与管理Swarm和Docker守护进程之间的通信。在启动Swarm服务时，会创建默认的overlay网络

Docker Swarm在初始化时会自动创建`docker_gwbridge`、`ingress`，启动服务时，自动创建该服务的overlay网络。一般情况下使用默认的`docker_gwbridge`、`ingress`、`overlay`网络即可，若自动生成的网络在子网中已存在，或需要手动配置网络，可以在Docker Swarm创建网络前手动创建网络，以自定义网络设置。

# 自定义ingress网络
ingress网络主要对集群中的服务进行负载均衡。未设置ingress网络的情况下，将会影响集群多分片任务的使用。

在Docker网络中，有且仅有一个ingress类型网络，Docker允许你手动自定义ingress网络。若是先有的网络中存在冲突网段，需要修改ingress网络。可以收到删除先有的ingress网络，重新创建ingress网络，指定网段。

- 在删除ingress网络前，需要先停止ingress连接的服务
- 删除现有的ingress网络
```shell
docker network rm ingress
```
- 创建新的ingress网络
  - ingress网络是一种特殊的overlay网络。所以其驱动为overlay
  - 通过`--ingress`标志设置网络为ingress网络
  - docker中有且仅有一个ingress网络，可以将ingress网络设置为自己喜欢的名称
```shell
docker network create \
  --driver overlay \
  --ingress \
  --subnet=10.11.0.0/16 \
  --gateway=10.11.0.1 \
  --opt com.docker.network.driver.mtu=1500 \
  my-ingress
```

# 自定义docker_gwbridge
`docker_gwbridge`是docker swarm的虚拟网桥。在初始化或加入Swarm集群时，会自动创建`docker_gwbridge`网络。

不同于`ingress`，`docker_gwbridge`网络需要在初始化或加入Swarm集群前创建。

- 若在配置`docker_gwbridge`已加入集群，请先退出集群
```shell
docker swarm leave --force
```
- 创建`docker_gwbridge`网络:
  - enable_icc: 启用或禁用容器间连接
  - enable_ip_masquerade: 启用IP伪装
  - 在创建`docker_gwbridge`网络时会报Swarm异常的日志，不用管该问题，直接创建即可
```shell
docker network create \
    --subnet 192.168.1.0/24 \
    --opt com.docker.network.bridge.name=docker_gwbridge \
    --opt com.docker.network.bridge.enable_icc=false \
    --opt com.docker.network.bridge.enable_ip_masquerade=true \
    docker_gwbridge
```
- 重新初始化或加入集群

# 自定义overlay网络
在通过Swarm部署服务时，会自动创建overlay网络，若是需要提前指定overlay网络，使服务只在固定的网段部署，可以通过创建overlay网络的情况处理。

创建`overlay`网络
```shell
docker network create \
    --driver overlay \
    --subnet=192.168.1.0/24 \
    your-overlay-network
```

若在已创建`overlay`网络的情况下，在服务中使用`overlay`网络，可以在`compose`文件中添加如下配置:
```yaml
version: '3.8'
services:
  your-service-name:
    ...
    environment:
      - TZ=Asia/Shanghai
    networks:
      - your_network

networks:
  your_network:
    # 使用外包网络，即已创建网络
    external: true
    name: your-overlay-network
```