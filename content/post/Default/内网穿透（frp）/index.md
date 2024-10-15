---
title: 内网穿透（frp）
description: 内网穿透（frp）
date: 2024-02-23
slug: nat_traversa/frp
image: 
categories:
    - Default
tags:
    - NAT
---

# 内网穿透
内网穿透是指在内部网络（内网）中实现穿透外网（公网）的通信。内网通常是指公司、组织或家庭等内部网络，而外网则是指互联网，也就是公共网络。内网穿透的目的是让位于内网中的设备或应用程序能够访问外部网络中的资源，实现内外网的通信。
## 实现内网穿透的常见方法
实现内网穿透的方法有很多，以下是一些常见的内网穿透技术：
1. 虚拟专用网络（VPN）：通过建立VPN连接，可以让位于内网的设备访问外网的资源，但这种方法相对复杂，需要配置和管理VPN服务器。
2. 代理服务器：通过在内外网之间设置代理服务器，可以让位于内网的设备通过代理服务器访问外网资源。这种方法简单易用，但可能会导致网络延迟和数据传输速度变慢。
3. 端口映射：通过在内外网之间映射特定的端口，可以让位于内网的设备访问外网的资源。这种方法简单易用，但需要关心端口映射的安全问题。
4. 负载均衡：通过在内外网之间设置负载均衡器，可以将内外网的流量分发到多个服务器上，从而实现内外网的通信。这种方法适用于大型企业和高性能场景，但需要部署和维护负载均衡器。
5. 隧道技术：通过在内外网之间建立隧道，可以让位于内网的设备访问外网的资源。这种方法类似于VPN，但更加简洁易用。
# frp
- [fatedier/frp GitHub 官方仓库](https://github.com/fatedier/frp)

frp是一款免费开源的专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS、Websocket、P2P 等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。
## frp的实现原理
frp是基于C/S模式实现的内网穿透服务代理应用，通过在公网IP上部署服务器端应用，客户端部署在内网上。当访问服务端暴露的应用时，反向代理到内网的服务实现内网穿透的代理。
## frps服务端应用部署
在frp中，分为客户端`frpc`和服务端`fprs`，frp应用支持以`toml`配置文件的方式和参数的方式进行服务部署。frp应用通过`-c`参数指定配置文件进行读取
```shell
frps -c frps.toml
frpc -c frpc.toml
```
### 配置文件部署
在部署`frps`时，需要指定绑定端口用于接收客户端连接，默认情况下绑定端口为`7000`，当需要修改用于客户端连接的端口时，可以修改此配置。在配置文件中通过`bindPort`参数进行配置
```toml
# frps.toml
bindPort = 7000
```
通过docker compose部署
```yaml
version: '3.9'
services:
  frps:
    image: fatedier/frps:v0.53.2
    hostname: frps
    container_name: frps
    volumes:
      - "./frps.toml:/frps.toml"
    command:
      - "-c"
      - "/frps.toml"
    ports:
      - "7000:7000"
```
### 参数配置
同样的，frp也支持在启动时直接以参数的方式进行服务部署，通过`--bind_port`可以设置`frps`的绑定端口
```shell
frps --bind_port 7000
```
通过docker compose部署
```yaml
version: '3.9'
services:
  frps:
    image: fatedier/frps:v0.53.2
    hostname: frps
    container_name: frps
    command:
      - "--bind_port"
      - "7000"
    ports:
      - "7000:7000"
```
## frpc客户端部署
在`frpc`客户端中，主要配置为
- 配置`frps`连接信息
- 配置内网应用

在`frpc`中需要配置的内容相对较多，推荐以文件的方式进行配置，以下是`frpc.toml`的简单示例
```toml
# 服务端地址
serverAddr = "x.x.x.x"
# 服务端配置的bindPort
serverPort = 7000

[[proxies]]
# 代理应用名称，根据自己需要进行配置
name = "ssh"
# 代理类型
type = "tcp"
# 客户端代理应用IP
localIP = "127.0.0.1"
# 客户端代理应用端口
localPort = 8080
# 服务端反向代理端口
remotePort = 7001
```
如上述示例所示，为客户端`frpc`代理配置，通过tcp协议的方式将本地8080端口服务代理到`serverAddr:7001`上，代理后，可以通过`serverAddr:7001`直接在公网上访问到本地服务。

通过docker compose部署frpc应用：
```yaml
version: '3.9'
services:
  frpc:
    image: fatedier/frpc:v0.53.2
    hostname: frpc
    container_name: frpc
    volumes:
      - "./frpc.toml:/frpc.toml"
    command:
      - "-c"
      - "/frpc.toml"
    network_mode: "host"
```
- `fatedier/frpc`镜像在启动时，直接运行`frpc`命令，`frpc`命令到默认配置文件默认参数为`./frpc.ini`文件，所以必须要挂载配置文件到容器内，否则无法直接启动`frpc`服务
- 通过`frpc`进行内网穿透，`frpc`需要可以直接访问到需要内网穿透代理到应用，所以采用`network_mode: host`参数将`frpc`服务到网络设置为宿主机网络，否则容器内无法直接访问宿主机上的应用，网络根据代理应用进行配置
	- 由于docker是基于Linux内核开发的，在Mac、Windows系统上对Docker应用本质是通过虚拟机运行对，所以`network_mode: host`对应的网络为虚拟机网络，而不是宿主机网络，所以该参数在Mac、Windows系统上不受支持，可以考虑直接以应用的方式在以上系统部署
- 部署后，内网服务的公网访问方式为`serverAddr:remotePort`，`bindPort`仅用于frp交互使用，所以在上面的`frps`通过docker方式中，需要将内网对应的`remotePort`和宿主机关联配置好
	- 在部署`frps`指定`ports`时可以考虑给定范围的方式进行部署
> 更多frp部署应用方式请参考[frp 官方文档](https://gofrp.org/)
