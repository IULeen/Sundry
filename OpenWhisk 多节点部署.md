前提条件：已部署好K8s集群，并安装好Helm

- 官方教程：[apache/openwhisk-deploy-kube: The Apache OpenWhisk Kubernetes Deployment repository supports deploying the Apache OpenWhisk system on Kubernetes and OpenShift clusters. (github.com)](https://github.com/apache/openwhisk-deploy-kube#Deploying OpenWhisk)、

### 部署步骤：

#### 1. 配置 Dynamic Storage Provisioning 

- 在部署OpenWhisk之前，需要配置一下k8s的动态存储

#### 2. 部署OpenWhisk



### Reference 

1. OpenWhisk在k8s 上部署的配置说明

   - 配置选择的详细说明
     - https://github.com/apache/openwhisk-deploy-kube/blob/master/docs/k8s-diy.md

2. k8s 的部署方式

   - 在Ubuntu 18.04 上部署 K8s
     - Kubernetes cluster example with Ubuntu
     - https://github.com/apache/openwhisk-deploy-kube/blob/master/docs/k8s-diy-ubuntu.md

3. OpenWhisk 配置参考教程

   - 博客教程

     - 在 Kubernetes 环境下部署 OpenWhisk 服务
       - https://zhuanlan.zhihu.com/p/269388017

     - 个人博客教程：Deploying OpenWhisk on Kubernetes on OpenStack
       - https://blog.zhaw.ch/splab/2019/02/21/deploying-openwhisk-on-kubernetes-on-openstack/

   - 官方GitHub教程：
     - OpenWhisk Deployment on Kubernetes
       - https://github.com/apache/openwhisk-deploy-kube
     - 持久性存储，根据知乎教程讲解，及官方教程，似乎要自己配置 NFS 服务以及 持久性存储 配置
       - https://github.com/apache/openwhisk-deploy-kube/blob/master/docs/k8s-nfs-dynamic-storage.md
     - Deploying OpenWhisk on "build-it-yourself" Kubernetes clusters
       - 配置文件参数的介绍：https://github.com/apache/openwhisk-deploy-kube/blob/master/docs/k8s-diy.md
       - 配置文件参数的详细介绍：https://github.com/apache/openwhisk-deploy-kube/blob/master/docs/configurationChoices.md