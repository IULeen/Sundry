## K8s 入门笔记

### 第1章： Kubernetes 入门

#### 1.1 了解K8s

#### 1.2 为什么要用K8s

#### 1.3 从一个简单的例子开始

#### 1.4 Kubernetes的基本概念和术语

##### 1.4.1 资源对象概述

- K8s中的基本概念和术语大多是围绕**资源对象(Resource Object)**展开的，资源对象总体上可分为以下两类：
  - (1)某种资源的对象，例如节点Node，Pod，服务Service，存储卷Volume
  - (2)与资源对象相关的事物与动作，例如标签Label，注解Annotation，命名空间Namespace，部署Deployment，HPA，PVC等
- 资源对象一般包括几个通用属性：
  - 版本
  - 类别Kind
  - 元数据Metadata
    - 名称：资源对象名称要唯一
    - 标签
    - 注解
- 可以采用YAML或JSON格式声明(定义或创建)一个K8s资源对象，每个资源对象都有自己的特定结构定义，并且统一保存在ETCD这种非关系型数据库中
- 所有资源对象都可以通过kubectl工具或API编程调试执行增删查改等操作

##### 1.4.2 集群类 Cluster

- 集群Cluster表示一个由Master和Node组成的K8s集群
- (1) Master，集群的控制节点，运行着以下关键进程
  - Kubernetes API Server (kube-apiserver)
  - Kubernetes Controller Manager (kube-controller-manager)
  - Kubernetes Scheduler (kube-shceduler)
  - etcd 服务
- (2) Node，集群的工作节点，运行着以下关键进程
  - kubelet
  - kube-proxy
  - 容器运行时

##### 1.4.3 应用类

- (1) Service 与 Pod
  - Service
  - Pod：
    - 每个Pod有一个特殊的被称为 “根容器”的Pause容器。Pause容器对应的镜像术语K8s平台的一部分
    - 每个Pod还包含一个或多个紧密相关的用户业务容器。

  - 为何使用Pod以及Pod特殊组成结构的设计原因：
    - 1.
    - 2.

  - 每个Pod有一个唯一的Pod IP，k8s支持任意两个Pod之间通过 TCP/IP 通信
    - 通常采用虚拟二层网络技术实现，例如 Flannel、Open vSwitch等

  - Pod 的分类：
    - 普通Pod
    - 静态Pod：Static Pod

  - Endpoint：
    - Pod IP + containerPort (一个容器对外暴露的端口号)

  - Pod Volume：
    - 对应于Docker Volume的概念，定义在Pod上，被各个容器挂载到自己的文件系统中

  - Event：
    - 一个事件的记录，记录事件的最早产生时间、最后重现时间、重复次数、发起者、类型，以及导致此事件的原因等
    - Event通常会被关联到某个具体的资源对象上，是排查故障的重要参考信息

  - Pod 及周边对象
    ![image-20220928112016838](C:\Users\14112\AppData\Roaming\Typora\typora-user-images\image-20220928112016838.png)

- (2) Label 与标签选择器
  - Label与标签选择器共同构成了k8s系统中核心的应用模型，可对被管理对象进行精细的分组管理，同时实现了整个集群的高可用性。

- (3) Pod 与 Deployment
  - Pod、Deployment与Service的逻辑关系
    ![image-20220928113146931](C:\Users\14112\AppData\Roaming\Typora\typora-user-images\image-20220928113146931.png)

- (4) Service 的 ClusterIP地址
  - Cluster IP地址是一种虚拟IP地址

- (5) Service的外网访问问题
  - 首先弄明白K8s的三种IP
    - Node IP：Node的IP地址
      - K8s集群中每个节点的物理网卡的IP地址，是一个真实存在的物理网络

    - Pod IP：Pod的IP地址
      - 它是Docker Engine根据docker0网桥的IP地址段进行分配的，通常是一个虚拟二层网络
      - Pod IP之间的通信，真实的TCP/IP流量是通过Node IP所在的物理网卡流出的

    - Service IP：Service的IP地址
      - Service 的 Cluster IP地址属于集群内的地址，无法在集群外直接使用这个地址
      - 为了解决这个问题，K8s首先引入了NodePort这个概念
        - 在K8s集群的每个Node上都为需要外部访问的Service开启一个对应的TCP监听端口，
        - 外部系统只要用***任意一个***Node的IP地址+NodePort端口号即可访问此服务

  - NodePort的确功能强大且通用性强，但也存在一个问题，即每个Service都需要Node上一个端口。而这又是一个有限的物理资源：
    - 能够让多个Serverless共用一个对外端口呢？
    - 后来增加的**Ingress资源对象**就解决了这个问题
      - 一定程度上，可以把Ingress实现机制理解为基于Nginx的支持虚拟主机的HTTP代理

- (6) 有状态的应用集群
  - Deployment对象用来实现无状态服务的多副本自动控制功能，
  - 那么有状态的服务，例如ZooKeeper集群、MySQL高可用集群、Kakfa集群等是怎么实现自动部署和管理的呢？
    - 一开始是依赖 StatefulSet 解决的，后来发现功能不够通用和强大
    - 后来又出现了 Kubernetes Operator

  - 有状态集群的特殊共性，同时在一定意义上也是缺点：
    - 每个节点都有固定的身份ID，通过这个ID，集群中的成员可以相互发现并通信
    - 集群的规模是比较固定的，集群规模不能随意变动
    - 集群中的每个节点都是有状态的，通常会持久化数据到永久存储中，每个节点在重启后都需要使用原有的持久化数据
    - 集群中的节点的启动顺序（以及关闭顺序）通常也是确定额
    - 如果磁盘损坏，则集群中的某个节点无法正常运行，集群功能受损

- (7) 批处理应用
  - 应用特点：一个或多个进程处理一组数据（图像、文件、视频等），在这组数据都处理完成后，批处理任务自动结束。
    - 为了解决这个问题，K8s引入了新的资源对象，Job
      - Jobs控制器提供了两个控制并发数的参数：completions和parallelism

    - 后来K8s又增加了 cronjob，定时任务，可以周期性地执行某个任务

- (8) 应用的配置问题
  - 通过前面学习，初步理解了三种应用建模的资源对象：
    - 无状态服务的建模：Deployment
    - 有状态集群的建模：StatefulSet
    - 批处理应用的建模：Job

  - 在进行应用建模时，如何解决应用需要在不同的环境中修改配置的问题呢？
    - *<u>ConfigMap以及Secret部分后续实操过，可再重新阅读并理解</u>*
    - ConfigMap
      - 顾名思义：就是保存配置项（Key=value）的一个Map
      - ConfigMap是分布式系统中“配置中心”的独特实现之一

    - Secret
      - 解决的是敏感信息的配置问题，创建一个Secret对象，然后被Pod引用

- (9) 应用的运维问题
  - 自动运维的问题
    - HPA：Horizontal Pod Autoscalar
    - VPA：Vertical Pod Autoscalar


##### 1.4.4 存储类

- 存储类的资源对象主要包括四类：
  - 静态存储管理：
    - (1) Volume
  - 动态存储管理：
    - (2) Persistence Volume
    - (3) PVC
    - (4) StorageClass
- 基础的存储类资源对象：Volume，存储卷
  - Volume介绍：略
  - Volume 分类：
    - emptyDir
    - hostPath
    - 公有云Volume
    - 其他类型的Volume
      - iscsi
      - nfs
      - glusterfs
      - rbd
      - gitRepo
      - configmap
      - secret
- Persistence Volume, PV
  - 由系统动态创建，dynamically provisioned，的一个存储卷
  - 与Volume类似，但是PV并不是被定义在Pod上，而是独立于Pod之外定义的
- K8s支持的存储系统有多种，系统如何知道从哪个存储系统中创建什么规格的PV存储卷呢？
  - 这就涉及到了StorageClass与PVC
  - StorageClass：用来描述和定义某种存储系统的特征
    - StorageClass有几个关键属性：
      - provisioner：代表了创建PV的第三方存储插件
      - parameters：是创建PV时的必要参数
      - reclaimPolicy：表明了PV回收策略，策略包括删除或保留
    - StorageClass的名称会在 PVC，PV Claim，中出现
  - PVC，PV Claim
    - PVC正如其名，表示应用希望申请的PV规格，其中包括重要属性
      - accessModes：存储访问模式
      - storageClassName：用哪种StorageClass来实现动态创建
      - resources：存储的具体规格
  - StorageClass与PVC一起，作为基础，保证了动态PV管理机制

##### 1.4.5 安全类

- Service Account
- Role
- RoleBinding



### 第 2 章 Kubernetes安装配置指南

#### 2.1 系统要求

#### 2.2 使用kubeadm工具快速安装 K8s集群

##### 2.2.1 安装 kubeadm

##### 2.2.2 修改 kubeadm 的默认配置

##### 2.2.3 下载k8s的相关镜像

##### 2.2.4 运行 kubeadm init 命令安装Master节点

##### 2.2.5 将新的 Node 加入集群

##### 2.2.6 安装CNI插件

##### 2.2.7 验证K8s集群是否工作正常

#### 2.3 以二进制文件方式安装k8s安全高可用集群

##### 2.3.1 Master 高可用部署架构

##### 2.3.2 创建CA根证书

##### 2.3.3 部署安全的etcd高可用集群

- etcd 作为 k8s集群的主数据库，在安装k8s各服务之前需要首先安装和启动

##### 2.3.4 部署安全的k8s Master高可用集群

##### 2.3.5 部署Node的服务

##### 2.3.6 kube-apiserver基于token的认证机制

#### 2.4 使用私有镜像库的相关配置

#### 2.5 Kubernetes的版本升级

##### 2.5.1 二进制文件升级

##### 2.5.2 使用kubeadm进行集群升级

#### 2.6 CRI，容器运行时接口，详解

##### 2.6.1 CRI概述

##### 2.6.2 CRI的主要组件

- <img src="C:\Users\14112\AppData\Roaming\Typora\typora-user-images\image-20221010195058446.png" alt="image-20221010195058446" style="zoom:50%;" />
- 镜像拉取采用 gRPC 的方式，那么冷启动环节镜像拉取耗时较大。
  - 如何提速？

##### 2.6.3 Pod 和容器的生命周期管理

##### 2.6.4 面向容器级别的设计思路

##### 2.6.5 尝试使用新的Docker-CRI来创建容器

##### 2.6.6 CRI的进展

#### 2.7 kubectl命令行工具用法详解

##### 2.7.1 kubectl用法概述

##### 2.7.2 kubectl子命令详解

##### 2.7.3 kubectl可操作的资源对象详解

##### 2.7.4 kubectl的公共参数说明

##### 2.7.5 kubectl格式化输出

##### 2.7.6 kubectl常用操作示例





### 第 3 章 深入掌握Pod