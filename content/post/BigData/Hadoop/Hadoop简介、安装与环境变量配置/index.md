---
title: Hadoop简介、安装与环境变量配置
description: Hadoop简介、安装与环境变量配置
date: 2023-07-06
slug: big_data/hadoop/hadoop_init
image: 
categories:
    - BigData
    - Hadoop
tags:
    - BigData
    - Hadoop
---

# Hadoop简介
Hadoop是由Apache基金会开源的具有可靠、可扩展、分布式计算的大数据框架。Hadoop可以简单的从单个服务器扩展到数千台机器，提供分布式的计算和存储服务。Hadoop通过检测和处理应用程序层的故障来为服务器集群提供高可用性服务。

# Java环境安装
> Hadoop作为大数据框架，更多的是作为集群面向服务器使用，所以本系列的内容以Linux服务器为主。为更好的学习使用Hadoop，在低成本的条件下，推荐可以通过docker、podman之类的容器手段启动容器开启集群的方式来更好的学习Hadoop（有条件的可以选择虚拟机或上云）。

Hadoop是一个用Java语言开发的大数据框架，Hadoop的使用依赖于Jre，所以在正式安装配置Hadoop前，我们需要先安装配置好Java环境。
目前最新版的Hadoop官方文档显示，Hadoop支持Java8和Java11（运行时）。为了更好的兼容Hadoop的使用，本系统采用OpenJDK SE 8版本。

- [OpenJDK SE 8 下载地址](https://jdk.java.net/java-se-ri/8-MR5)

## OpenJDK SE 8下载
```shell
wget https://download.java.net/openjdk/jdk8u43/ri/openjdk-8u43-linux-x64.tar.gz
```

## 解压到指定目录
大数据的学习和使用都是集群化的，为更好的进行集群设置，请尽量保证相同的操作系统和应用层配置。如，将软件和配置文件写在指定路径下，不要随意修改。
```shell
mkdir -p ${Software}/jdk
tar -xzvf openjdk-8u43-linux-x64.tar.gz --strip-components 1 -C ${Software}/jdk
```
- 直接解压会生成父目录，Java的版本不同会造成父目录的不同，所以在此处使用指定路径的方式，方便后期更好的升级环境
- 在这里和之后中通过`${}`表明根据自己的环境自定义值
- 指定路径解压前，通过`mkdir -p`命令确保指定路径存在
- --strip-components Number：解压时清除Number个引导目录，一般情况下，Number为1表示不包含打包前原目录
- -C：指定解压路径

## Java安装与环境变量配置
Linux的环境变量设置一般为在相应的文件中添加环境变量信息。根据使用权限的不同，可以配置不同的环境变量。
- 当前用户环境变量：`~/.bash_profile`
	- `~`：当前用户的工作路径
- 全局环境变量：`/etc/profile`

在这里我们以全局环境变量为例，配置Java环境变量。Java环境变量主要为配置`JAVA_HOME`，`PATH`。Java环境变量可以通过如`vim`手动打开写入方式，也可通过标准流输出追加文件内容方式写入，为更好的方便后期集群中环境脚本的开发，这里采用标准流输出追加文件内容方式配置环境变量。
```shell
echo "# >>> jdk initialize >>>" >> /etc/profile
echo "export JAVA_HOME=${Software}/jdk" >> /etc/profile
echo "export PATH=${JAVA_HOME}/bin:${PATH}" >> /etc/profile
echo "# <<< jdk initialize <<<" >> /etc/profile
```
- 开头和结尾的主要是为了标识Java安装位置，为注释内容，不生效
- Linux中`>>`表示为文档后追加文件内容
- 若无法写入，检查是否是权限的问题，可以切换为`root`账号执行操作。或写入自己环境中的配置文件

## 重载环境变量配置文件
将环境变量写入配置文件后，环境变量不会立即生效，需要重新加载配置文件，Linux中使用`source`命令重新加载配置文件。
```shell
source /etc/profile
```

## 环境配置测试
执行`java -version`和`javac -version`有正确的输出即表示Java环境配置成功。
```shell
$ java -version
openjdk version "1.8.0_43"
OpenJDK Runtime Environment (build 1.8.0_43-b03)
OpenJDK 64-Bit Server VM (build 25.40-b25, mixed mode)

$ javac -version
javac 1.8.0_43
```

# Hadoop安装与环境变量配置
## 固定IP
在集群中，最重要的就是主机与主机之间能够相互访问到。所以需要通过相应的标识来识别到对应的节点。在计算机中，可以通过域名和IP地址的方式识别到相应的服务器，在Hadoop集群的配置中也是如此。在同一网段下，进行IP设置，可以考虑采用静态IP的方式而不是DHCP动态IP，防止节点IP发生变化无法访问。
- 在使用云系统的情况下，请先自己购买云服务器的平台先设置VPC（Virtual Private Cloud）云虚拟局域网，然后在购买主机，保证购买的主机在同一个局域网，能够相互访问
- 本系统采用容器的方式组集群，借助容器的`VIP`虚拟IP的概念，可以不用考虑固定容器内部IP，而是通过容器名（类似于域名）的方式访问节点
- 虚拟机组集群用户可网上自行搜索资料，固定自己的IP

为方便访问子节点（IP不好记），可以考虑为自己的集群节点配置一个本地的host，这样可以直接用类似域名的方式直接访问集群节点。
Linux中将host主机名映射配置写在`/etc/hosts`文件中，这样之后直接访问`${HostMapName}$就可以访问相应的节点。
```shell
sudo echo "${StaticIP} ${HostMapName}" >> /etc/hosts
```

## 设置SSH免密登录
Hadoop在启动时，只需要在主节点执行Hadoop执行脚本，Hadoop会自动根据配置启动主从节点的服务。但是Hadoop在启动服务时，主节点需要访问所有节点，然后从相应节点中启动守护进程，所以配置主节点到所有节点之间的免密登录（包括主节点到主节点自身之间的免密登录）。

### 生成SSH密钥
```shell
ssh-keygen -t rsa -b 4096 -f ～/.ssh/id_rsa -N "" -q
```
- -t：指定生成密钥的算法参数
	- rsa：默认非对称加密算法，加解密速度慢，生成时间慢，安全性不如ed25519算法，但兼容性高，应用广泛
	- ed25519：带椭圆曲线的非对称加密算法，加解密速度快，生成速度快，安全性更高
	- ed25519-sk：`-sk`代表安全密钥
	- dsa：安全性不高，基本不在使用，在部分的场景中已被遗弃
	- ecdsa：带椭圆曲线的dsa算法
	- ecdsa-sk
- -b：指定密钥长度
- -f：保存密钥的文件路径
- -N：提供一个新密码，`""`表示密码为空
- -q：静默模式，直接输出密钥对，不输出实现过程中产生的信息
- 通过设置`-f`，`-N`，`-q`参数，直接生成密钥，而不是交互式生成密钥，方便后期集群脚本的开发

### 设置免密登录
```shell
ssh-copy-id ${HostMapName}
```
使用`ssh-copy-id`可以将公钥传输到指定的主机上。但是在使用`ssh-copy-id`传输公钥时，需要输入相应的账户和密码，所以可以通过`sshpass`来执行免密操作。
```shell
sshpass -p ${USER_PASSWORD} ssh-copy-id ${HostMapName}
```

## Hadoop的下载
[Hadoop 下载地址](https://hadoop.apache.org/releases.html)

这里选择写本文时的最新版3.3.6进行下载
```shell
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz 
```

## 解压到指定位置
```shell
tar -xzvf hadoop-3.3.6.tar.gz --strip-components 1 -C ${Software}/hadoop
```

## 环境变量配置
### 系统环境变量
```shell
echo "# >>> hadoop initialize >>>" >> /etc/profile
echo "export HADOOP_HOME=${Software}/hadoop" >> /etc/profile
echo "export HADOOP_CONF_DIR=${HADOOP_CONF_DIR}/etc/hadoop" >> /etc/profile
echo "export HADOOP_LOG_DIR=${HADOOP_LOG_DIR}" >> /etc/profile
echo "export HADOOP_DATA_HOME=${HADOOP_DATA_HOME}" >> /etc/profile
echo "export PATH=${HADOOP_HOME}/bin:${PATH}" >> /etc/profile
echo "export PATH=${HADOOP_HOME}/sbin:${PATH}" >> /etc/profile
echo "# <<< hadoop initialize <<<" >> /etc/profile
```
- bin：一般存放着软件的相关执行文件
- sbin：hadoop的super bin目录。是Hadoop管理脚本所在的目录，主要包含HDFS和YARN中各类服务的启动/关闭脚本。
### `${HADOOP_CONF_DIR}/workers`工作节点配置
`workers`主要功能为记录所有的数据节点的主机名或IP地址。将集群的所有节点的主机名或者IP地址写入workers文件即可。
```shell
node1
node2
node3
```
### `${HADOOP_CONF_DIR}/hadoop-env.sh`环境变量配置
在Hadoop中，一些环境变量无法读取系统的环境变量，所以需要配置在`hadoop-env.sh`文件中，理论上讲，只需要配置`JAVA_HOME`路径，其他的可以采用默认路径。
```shell
echo "export JAVA_HOME=${JAVA_HOME}" >> ${HADOOP_CONF_DIR}/hadoop-env.sh
echo "export HADOOP_HOME=${HADOOP_HOME}" >> ${HADOOP_CONF_DIR}/hadoop-env.sh
echo "export HADOOP_CONF_DIR=${HADOOP_CONF_DIR}" >> ${HADOOP_CONF_DIR}/hadoop-env.sh
echo "export HADOOP_LOG_DIR=${HADOOP_LOG_DIR}" >> ${HADOOP_CONF_DIR}/hadoop-env.sh
```
### `${HADOOP_CONF_DIR}/coer-site.xml`核心文件配置
`coer-site.xml`文件主要为hadoop的核心配置，在`coer-site.xml`文件中，我们主要需要做的事配置好hadoop主节点的通讯地址。
```xml
# fs.defaultFS：节点的通讯地址
<property>
	<name>fs.defaultFS</name>
	<value>hdfs://node1:9001</value>
</property>
```