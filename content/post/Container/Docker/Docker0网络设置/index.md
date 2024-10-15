---
title: Docker0网络设置
description: Docker0网络设置
date: 2024-01-15
slug: container/docker/docker0_network
image: 
categories:
    - Container
    - Docker
    - Network
tags:
    - Container
    - Docker
    - Network
---

# Docker网络
当部署运行Docker后，Docker在启动时会默认创建三个网络
- bridge: 默认网络驱动程序。当不指定网络驱动运行容器时，默认会使用该网络。
- host: 直接访问宿主机网络，取消容器和宿主机的网络隔离
  - host网络模式可以直接使用宿主机网络，但是在Windows和MacOS中，Docker是跑在虚拟机中的，此时Docker的host网络是其所在虚拟机的网络而不是宿主机网络，所以在Windows和MacOS中采用host仍无法直接访问宿主机网络
- none: 无网络，容器与宿主机及其他容器完全隔离

通过`ip addr show`查看网络，可以发现，在运行Docker后，宿主机上多了一个`docker0的网络`，通过查看`docker network inspect bridge`可知，`docker0`网络本质上就是默认的`bridge`网络。
```shell
docker network inspect bridge
{
    "Name": "bridge",
    ...
    "Options": {
        ...
        "com.docker.network.bridge.name": "docker0"
        ...
    }
    ...
}
```
由以上信息可知，docker默认网络`bridge`就是宿主机上的`docker0`网络。

# 配置Docker0
由于`bridge`网络是在Docker启动时创建的默认网络。无法在Docker运行时修改，查看Docker官方文档可知，默认`bridge`可以通过配置文件`daemon.json`来进行配置

打开`/etc/docker/daemon.json`文件，根据以下参数进行网络配置:
```json
{
  "bip": "192.168.1.1/24",                      # bridge IP，bridge网络本身的IP地址
  "fixed-cidr": "192.168.1.0/25",               # IPv4网段范围（CIDR）
  "fixed-cidr-v6": "2001:db8::/64",             # IPv6网段范围（CIDR）
  "mtu": 1500,                                  # 最大传输单元
  "default-gateway": "192.168.1.254",           # IPv4网关地址
  "default-gateway-v6": "2001:db8:abcd::89",    # Ipv6网关地址
  "dns": ["10.20.1.2","10.20.1.3"]              # DNS地址
}
```
根据网段需求，设置默认配置，针对IPv4网络，最低要求配置如下:
```json
{
    "bip": "192.168.1.1/24",
    "mtu": 1500,				
    "fixed-cidr": "192.168.1.0/24"		
}
```