---
title: Kubernetes基本部署概念
description: Kubernetes基本部署概念
date: 2024-02-23
slug: container/kubernetes/basic_deployment_concepts
image: 
categories:
    - Container
    - Kubernetes
tags:
    - Container
    - Kubernetes
---

> <font color="red">**在K8s中，进行资源命名时，大部分支持的命名方式为DNS子域名命名方式，即xxx-xxx的方式，在大部分资源命名时，不支持_，所以在命名时但需要进行驼峰划分时，推荐使用-进行划分**</font>

# 命名空间（Namespaecs）
## 查看命名空间
```shell
# 列出集群所有命名空间
kubectl get namespaces
kubectl get ns

# 获取命名空间摘要信息
kubectl get ns ${namespace_name}

# 获取命名空间详情信息
kubectl describe ns ${namespace_name}
```
> ns: namespace命令的缩写，在k8s中可用ns命令指代namespaces，下面统一采用ns代替namespaces

## 查看带有命名空间对象下资源
```shell
# 以pods资源为例
kubectl get pods --ns=${namespace_name}
```
# 文件存储
在K8s中通过卷（volume）来存储文件信息。在容器中的的文件内容在磁盘上临时存放，当容器崩溃或被销毁时，容器上的生命周期、状态，文件信息等内容都会丢失。若是我们希望能够将容器内的文件信息和容器外的磁盘上文件进行关联，需要通过挂载卷的方式进行挂载，这些当容器出错时挂载的内容将不会被销毁。

## 持久卷（pv，Persistent Volumes）
K8s中的持久卷（Persistent Volumes，简称PV）是一种特殊类型的存储资源，它可以为 Kubernetes中的Pod 供持久性的数据存储。持久卷的特点如下：
1. 数据持久性：持久卷的数据在 Pod 退出时不会丢失，即使 Pod 被删除，卷中的数据也会保留。这使得持久卷成为一个非常适合存储数据持久性的解决方案。
2. 独立于 Pod：每个持久卷都是独立于 Pod 的，卷的存储和访问并不依赖于某个特定的 Pod。这意味着可以使用不同的持久卷来存储不同的数据，或者将多个持久卷分配给同一个Pod。
3. 动态分配：持久卷可以在Kubernetes中动态创建和删除。当一个PersistentVolumeClaim需要使用持久卷时，Kubernetes会自动为其分配一个空闲的持久卷。当PersistentVolumeClaim不再需要该卷时，Kubernetes会自动释放该卷，以便其他PersistentVolumeClaim使用。
4. 存储类型：持久卷可以存储各种类型的数据，如Block存储、File存储、网络文件系统（如NFS）等。这使得持久卷可以用于各种不同的场景和用例。
5. 共享访问：多个PersistentVolumeClaim可以共享同一个持久卷。这可以提高存储利用率，减少卷的创建和删除次数，从而简化了 Kubernetes 中的存储管理。

总之，持久卷是 Kubernetes 中一种非常灵活和强大的存储解决方案，它提供了数据持久性、独立性、动态分配、多种存储类型和共享访问等特点，使得在管理和存储数据时具有很高的灵活性和效率。

### 卷容量
一般来说，在PV持久卷里面需要声明卷容量，该容量通过`spec.capacity.storage`指定

### 卷模式（volumeMode）
在PV持久卷中，支持两种卷模式：
- Filesystem（文件系统）：
	- 属性设置为Filesystem的卷会被Pod挂载到某个目录
	- 如果卷的存储来自某块设备而该设备目前为空，Kuberneretes会在第一次挂载卷之前在设备上创建文件系统。
- Block（快设备）：
	- 将卷作为原始块设备来使用，其上没有任何文件系统
	- 这种模式对于为 Pod 提供一种使用最快可能方式来访问卷而言很有帮助
	- Pod和卷之间不存在文件系统层。另外，Pod中运行的应用必须知道如何处理原始块设备

在yaml中可以通过`spec.volumeMode`指定卷模式，该模式是可选的，默认为Filesystem

### 访问模式（accessModes）
在PV上的访问模式有：
- ReadWriteOnce（RWO）：卷可以被一个节点以读写方式挂载。ReadWriteOnce访问模式仍然可以在同一节点上运行的多个Pod访问该卷。 对于单个Pod的访问，请参考ReadWriteOncePod访问模式。
- ReadOnlyMany（ROX）：卷可以被多个节点以只读方式挂载。
- ReadWriteMany（RWX）：卷可以被多个节点以读写方式挂载。
- ReadWriteOncePod（RWOP）：卷可以被单个Pod以读写方式挂载。如果你想确保整个集群中只有一个Pod可以读取或写入该PVC，请使用ReadWriteOncePod访问模式。

在yaml中可以通过`spec.accessModes`指定卷模式，该模式是可选的，默认为Filesystem

### 回收策略（persistentVolumeReclaimPolicy）
目前的回收策略有：
- Retain：手动回收
- Recycle：简单擦除 (rm -rf /thevolume/*)
- Delete：删除存储卷

### local卷
local卷所代表的是某个被挂载的本地存储设备，例如磁盘、分区或者目录。local 卷只能用作静态创建的持久卷。不支持动态配置。

当我们需要挂载集群上某一宿主机的卷时，可以使用local卷进行挂载。

以下是一个PV使用local卷的示例：
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
	# 卷路径
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
			  # 卷所在集群节点名称
              - example-node
```
### nfs卷
nfs卷能将NFS(网络文件系统) 挂载到你的Pod中。
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-storage
  nfs:
    server: xxxxx
    path: "/"
```
## 持久卷声明（pvc，PersistentVolumesClaims）
在上述声明的PV卷是集群中的资源。声明了资源之后是无法直接对资源操作的，在K8s中通过持久卷声明（pvc，PersistentVolumesClaims）声明对资源对请求和检查工作。

即PV声明资源，PVC声明对资源的访问方式，通过PVC来访问持久卷资源。

> 同PV一样，PVC同样支持访问模式，卷模式，具体参考上文

### 资源 
同Pod一样，PVC可以请求特定数量的资源。但在中，请求的资源是存储。

通过`spec.resources.requests.storage`来声明请求的资源

PVC参考示例：
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  storageClassName: local-storage
  resources:
    requests:
      storage: 8Gi
```

## PV绑定PVC
- 动态绑定：在PV和PVC中通过`spec.storageClassName`声明存储类。如果集群中有合适的PV可用（即与PVC规格匹配并且未被绑定），则Kubernetes会自动将两者绑定在一起。如果没有可用PV，则可能根据存储类触发动态供应流程创建新的PV。
- 静态绑定：管理员可以手动创建PV，并为其设置特定的标签或者注解。然后在创建PVC时，可以明确指定需要匹配的标签，这样Kubernetes会在已有PV中寻找符合标签条件的PV进行绑定。

# Deployment
在K8s中，通过Deployment定义Pod和ReplicaSet的状态来提供声明式更新能力。通过在Deployment声明期望服务的目标状态，当运行中的Pod等资源以非目标状态访问时，Deployment控制器（Controller）以受控速率更改实际状态，将目标资源变为期望状态。

deployment示例
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: example-ns
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
          env:
            - name: "TZ"
              value: "Asia/Shanghai"
		  volumeMounts:
            - name: local-storage
              mountPath: /data/data/upload
              subPath: data
      volumes:
        - name: local-storage
          persistentVolumeClaim:
            claimName: ysocr-pvc
```
- metadata: Deployment元数据信息
	- name: 定义了deployment名称，且后续的Pod和ReplicaSet等资源以该name开头
	- namespace: 指定该deployment所在命名空间
	- label: 在Deployment中定义的标签，以`key: value`的方式定义
- spec.replicas: 声明的Pod和ReplicaSet资源的分片数量
- spec.selector: 定义所创建的ReplicaSet如何查找要管理的Pod。这里选择在Pod模板中定义的标签进行管理
- spec.template: 
	- metadata: 定义pod的元数据和
	- spec: pod容器配置
		- container: 创建一个容器并使用
			- ports.containerPort: 定义容器内暴露的端口
			- env: 容器环境变量配置
			- volumeMounts: 卷挂载
				- name: volumes定义的卷名称
				- mountPath: 容器内路径
				- subPath: 相对于卷的路径
		- volumes: 卷配置，通过关联PVC，挂载卷和容器

# Service（服务）
Deployment中定义了容器信息和容器的状态等内容，不同于Docker Compose或Docker Swarm同一服务下的容器可以相互访问。K8s中Deployment仅定义了容器的状态相关信息，如果希望在命名空间内部或者在宿主机上访问内部资源，需要通过Service的方式来公开程序为网络服务。

Servic 是将运行在一个或一组 Pod 上的网络应用程序公开为网络服务的方法。

## 服务类型（spec.type）
K8s允许通过服务类型来设置容器对外提供服务的方式

- ClusterIP
	- 这类服务仅在Kubernetes集群内部可见，不暴露给集群外部
	- ClusterIP服务使用`ClusterIP`类型，将Pods的网络接口与集群IP地址关联起来
	- 这种服务适用于在集群内部进行通信的情况
	- 该服务为默认Service服务类型，当不进行服务类型配置时，默认使用该配置
- NodePort
	- 这类服务在集群内部和外部都可以访问
	- 通过为Pods分配一个NodePort，NodePort服务允许用户在外部访问Pods，从而实现服务在集群内部和外部的统一访问
	- 适用于宿主机需要访问集群内部服务的情况
- LoadBalancer
	- 这类服务在集群内部和外部都可以访问
	- 这类服务将Pods的网络接口与集群external IP地址关联起来。LoadBalancer 服务在集群外部是可见的，可以接收来自外部的流量
	- 通常，LoadBalancer服务用于负载均衡和高可用场景，当Pod有多个分片时，可以考虑使用LoadBalancer进行负载均衡
- ExternalName
	- 这类服务将Pods的网络接口与外部的域名关联起来
	- ExternalName服务允许用户为服务指定一个外部域名，并将该域名解析为Pods的网络接口
	- 这种服务适用于将服务提供以域名的方式提供给外部用户的情况

## 端口定义（spec.ports）
在Kubernetes中，Services的spec.ports定义了服务的外部访问端口。这些端口在集群内是可见的，并且允许集群内的节点访问该服务。

`spec.ports`参数是一个包含多个端口对象的数组，每个端口对象表示一个外部访问端口。`spec.ports`参数对象的常见参数有：
- name: 当Services中只定义单个端口时，端口名称是可选的，当在一个Services中定义多个端口时，必须为所有端口提供名称，以使它们无歧义
- protocol: 支持的协议，可选参数，默认为`TCP`
- port: 服务访问端口，集群内部访问的端口，暴露到集群上的端口，定义了服务对外的访问接口，可以通过`clusterIp:port`的形式访问
- targetPort: 表示服务内部使用的端口。是Pod控制器中定义的端口（应用访问的端口）。一般情况下port和targetPort保持一致
	- 在Services中使用Pod控制器定义的端口时，需要在Pod中声明containerPort参数
- nodePort: 提供了集群外部客户端访问 Service 的一种方式，指定nodePort后，可以通过`nodeIp:nodePort`的形式进行访问
	- 默认情况下，为了方便起见，Kubernetes控制平面会从某个范围内分配一个集群外部可访问端口号（默认：30000-32767）

Services示例：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - name: name-of-service-port
      protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```
