#### OpenWhisk 学习笔记

#### 1. OpenWhisk 的 standalone 部署

- FaasCache 中使用的就是单节点模式，controller与invoker以一个整体进行构建

1. 虚拟机上安装一些前置软件

   - python 3

   - Java

     - `apt install openjdk-8-jdk`
     - 或者官网下载二进制压缩包，解压后配置相关环境变量

   - Docker

     - `apt install docker.io`
     - 或者使用官方的一键安装脚本
     - 安装完毕后运行一些命令，看是否安装成功

   - Nodejs

     - 首先使用脚本更新apt 资源

       - ```shell
         # Ubuntu OS
         curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
         
         # 然后进行安装
         sudo apt install nodejs
         
         # 验证版本号
         node --version
         
         npm --version
         ```

     - Linux 上安装 Node.js

       - ```shell
         # 官网下载二进制压缩包
         node-v16.14.2-linux-x64.tar.xz
         
         # 解压资源包
         tar -xvf  node-v14.18.0-linux-x64.tar.xz
         
         # 确认一下nodejs下bin目录是否有node 和npm文件
         # 建立软连接
         sudo ln -s /app/software/nodejs/bin/npm /usr/local/bin/ 
         
         sudo ln -s /app/software/nodejs/bin/node /usr/local/bin/
         
         # 检查版本号
         node --version
         
         npm --version
         ```

2. git 下载openfaas-cahing

   - ```shell
     # 使用 git 命令下载FaasCache
     
     git clone https://github.com/aFuerst/openwhisk-caching.git
     
     # 进入 下述文件夹
     cd openwhisk-caching
     
     # 版本回溯
     git checkout 38ff898
     
     # 编译构建 standalone
     ./gradlew :core:standalone:build
     
     # 编译完成后 ./bin 目录下出现 OpenWhisk-standalone.jar 包
     ```

3. 下载 wsk-cli 工具

   - ```shell
     # 在官网下载最新的安装包
     
     wget https://github.com/apache/openwhisk-cli/releases/download/1.2.0/OpenWhisk_CLI-1.2.0-linux-amd64.tgz
     
     # 使用 tar 命令进行解压
     tar zxvf test.tgz -C ./[指定目录]
     
     # 进入解压目录，设置 wsk 环境变量，或把 wsk 移到 /usr/local/bin
     cp wsk /usr/local/bin
     
     # 设置 wsk 属性
     wsk property set --apihost 'http://172.17.0.1:3233'
     --auth 'xxxxx'
     ```

4. FaasCache 相关工作

   - 注意还有以下几个镜像需要构建
     - wsk-py-build
     - lookbusy
     - python3ai-vid
   - Py package 函数制作
     - `./code/wsk-actions/py/build.sh`构建需要用到的zip函数
     - 这些函数已经构建好了，可以通过文件上传的方式传到虚拟机上
     - 将 `./sample.conf` 移至 `./bin/application.conf` 中

5. 运行 修改后的 OpenWhisk

   - ```shell
     # 使用 Java -jar 运行 openwhisk-standalond.jar
     
     java -jar openwhisk-standalond.jar --dev-mode -c application.conf | grep --color "cold hits" | cut -c 1-150 >> test.txt
     ```

   - ```shell
     # 开启新的小黑框
     cd ./code/wsk-actions/load-test
     
     # 执行 函数负载测试
     python3 sub_litmus.py
     
     # 或者执行其他负载测试函数
     # 测试的时候通过 jar 运行时的输出看 cold-hits、warm-hits数量
     ```

     

#### 2. OpenWhisk 在K8s上多结点部署

1. OpenWhisk在k8s 上部署的配置说明
   - 配置选择的详细说明
     - https://github.com/apache/openwhisk-deploy-kube/blob/master/docs/k8s-diy.md
2. k8s 的部署方式
   - 在Ubuntu 18.04 上部署 K8s
     - Kubernetes cluster example with Ubuntu
     - https://github.com/apache/openwhisk-deploy-kube/blob/master/docs/k8s-diy-ubuntu.md
3. 配置参考教程
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

#### 3. OpenWhisk standalone中一次调用的执行流程

###### 运行一个 hello (py函数) 时系统信息输出

- 异步调用
- 第一次冷启动

```shell
# controller接收到请求
[2022-04-15T06:04:39.086Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] POST /api/v1/namespaces/alftest/actions/hello blocking=false&result=false

# 进行用户身份认定
[2022-04-15T06:04:39.088Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [Identity] [GET] serving from cache: CacheKey(babecafe-cafe-babe-cafe-babecafebabe) [marker:database_cacheHit_counter:2]

# 寻找 hello 函数的 Cachekey
[2022-04-15T06:04:39.091Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [WhiskActionMetaData] [GET] serving from datastore: CacheKey(alftest/hello) [marker:database_cacheMiss_counter:6]

# 寻找 hello 函数的 document
[2022-04-15T06:04:39.091Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [MemoryArtifactStore] [GET] 'whisks' finding document: 'id: alftest/hello' [marker:database_getDocument_start:6]
[2022-04-15T06:04:39.092Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [MemoryArtifactStore]  [marker:database_getDocument_finish:7:1]

# 开启 controller activation start 并分配 activation id
[2022-04-15T06:04:39.109Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [ActionsApi]  [marker:controller_activation_start:24]
[2022-04-15T06:04:39.111Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [ActionsApi] action activation id: ca09d1cf2f534ee389d1cf2f531ee34a [marker:controller_loadbalancer_start:25]

# LeanBalancer kafka 组件为 activation 分配 主题 ‘topic0’
[2022-04-15T06:04:39.117Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [LeanBalancer] posting topic 'invoker0' with activation id 'ca09d1cf2f534ee389d1cf2f531ee34a' [marker:controller_kafka_start:32]
[2022-04-15T06:04:39.127Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [LeanBalancer] posted to invoker0[0][-1] [marker:controller_kafka_finish:42:10]

# Invoker激活，开启一个Invoker
[2022-04-15T06:04:39.130Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [InvokerReactive]  [marker:invoker_activation_start:45]

# 前面 controller 工作完成
[2022-04-15T06:04:39.131Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [ActionsApi]  [marker:controller_loadbalancer_finish:45:19]
[2022-04-15T06:04:39.132Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [ActionsApi]  [marker:controller_activation_finish:47:23]

# 记录 database_cacheMiss 数量，代表未找到 hello 函数的 document 缓存
[2022-04-15T06:04:39.132Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [WhiskAction] [GET] serving from datastore: CacheKey(alftest/hello) [marker:database_cacheMiss_counter:47]

# 获取 hello 函数的 document，并分配 rev 哈希值
[2022-04-15T06:04:39.132Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [MemoryArtifactStore] [GET] 'whisks' finding document: 'id: alftest/hello, rev: sha256-69d8f8a51ded64d656a7f72699b2c9f65685b8ffeb4535f9b5aa8d4d312d8f08' [marker:database_getDocument_start:47]
[2022-04-15T06:04:39.132Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [MemoryArtifactStore]  [marker:database_getDocument_finish:47:0]

# 获取 sha256 哈希加密后的 document 附件
[2022-04-15T06:04:39.135Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [BasicHttpService] [marker:http_post.202_counter:50:50]
[2022-04-15T06:04:39.135Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [MemoryArtifactStore] [ATT_GET] 'whisks' finding attachment 'mem:UEsDBBQAAAAIABBGjVSjRYEHEAIAAHEEAAAfABwAdmlydHVhbGVudi9iaW4vYWN0aXZhdGVfdGhpcy5weVVUCQADX45WYpiXVmJ1eAsAAQQAAAAABAAAAACNU01v2zAMvetXEB4K21jnDOstQA4dMGCHbeihlyEIDMWmE62yJEiKE__7kXKdpEWLzYBt8evxkRSzLPs6wiEoswM8YdMpjUXcq1Dz6RZa1cSiTkJdr86GsoTRHuCotBayiWqQEYGtMCgfD1KjGYBe5a3p0cRKiEe2NtLAFikftnDco0ko_SFEVgEZ8aRCZDIPY9xbA8pE9M4jfW_B2CjiHq9zbJVZuOQqsiwTIvpxKYCembPAU4Muwi_Z4zfvrZ_MXipKeB8C-qisSZYiWfjJfs-0_MFMdWn1hJcO5U7G_SLaxVx8zU6VG_PXLXvfsyyzUqjeWR8hjGE-2iCE1W1tQ82hsCJN9dzKaoexyB_uH79TnjwvxcW0ntSbyZ8jq1Z5Q1UXsyy3gf9nbjTEj7NzQMfCJa_YSmrQ-2D_BqfiOi6sclrGzvoeVivIj8rcfcmnIQRF7XCyeZI7DFe5_lhlCs5PRf5QW66VXT_NrlQ46oD_D6InkOmi3IQcbhKxAX2g4a-Xd5s3UtCtG2pym8eg6WYWqR6SL5OjKMGfSrYt_6kxxQtOpeAgj1LXBNmpE2ElmCSIy5H0zFd8gJ924HWijWhb2hRC6wNEm1QdDZtuSZcEprIUBo_XRNcbQe1OUbQ_r3hPTaPJJDNtFLu8KHV5XoNr3Eo6h6YtOKw8e8ywVF5PnJ-ts3a9_Mz38RpG_AVQSwMEFAAAAAgAgCyOVCTur6iZAAAA6wAAAAsAHABfX21haW5fXy5weVVUCQADwLJXYsCyV2J1eAsAAQQAAAAABAAAAAA9js0KwkAMhO_7FDGnFsUHKHgVH8C7RJtuC_tTsiki4ru727-cMh8zwwx-jKLgSXtjXtG1cIG7TGxMy13GQ6hIbKobA_msi09yUHyzflN6rKGdrfpKLvEMAnnOoNScLWuFBeAJMKlQsCxYL-XCrEOw2Ys3di4CwnFJH_PbSfQwfrSP4YBzQFgnCfDFLYnNXpLryxBstok_8wdQSwECHgMUAAAACAAQRo1Uo0WBBxACAABxBAAAHwAYAAAAAAABAAAApIEAAAAAdmlydHVhbGVudi9iaW4vYWN0aXZhdGVfdGhpcy5weVVUBQADX45WYnV4CwABBAAAAAAEAAAAAFBLAQIeAxQAAAAIAIAsjlQk7q-omQAAAOsAAAALABgAAAAAAAEAAADogWkCAABfX21haW5fXy5weVVUBQADwLJXYnV4CwABBAAAAAAEAAAAAFBLBQYAAAAAAgACALYAAABHAwAAAAA=' of document 'id: alftest/hello, rev: sha256-69d8f8a51ded64d656a7f72699b2c9f65685b8ffeb4535f9b5aa8d4d312d8f08' [marker:database_getDocumentAttachment_start:50]

# 将初始化的信息记录在 已有缓存条目 中，使 hello 函数 Cachekey 失效，然后记录 hello 新的 Cachekey
[2022-04-15T06:04:39.150Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [WhiskAction] write initiated on existing cache entry, invalidating CacheKey(alftest/hello), tid kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN, state WriteInProgress
[2022-04-15T06:04:39.150Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [WhiskAction] write all done, caching CacheKey(alftest/hello) Cached

# 调用 docker 程序，开启一个 容器实例
[2022-04-15T06:04:39.187Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [DockerClientWithFileAccess] running /usr/bin/docker pull openwhisk/python3action:latest (timeout: 10 minutes) [marker:invoker_docker.pull_start:101]

# 容器池中，未查询到 warm 容器，记录一个 cold
[2022-04-15T06:04:39.190Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [ContainerPool] containerStart containerState: cold container: None activations: 1 of max 1 action: hello namespace: alftest activationId: ca09d1cf2f534ee389d1cf2f531ee34a [marker:invoker_containerStart.cold_counter:105]

# 镜像拉取成功，本地镜像
[2022-04-15T06:04:43.327Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [DockerClientWithFileAccess]  [marker:invoker_docker.pull_finish:4242:4140]
[2022-04-15T06:04:43.328Z] [INFO] [#tid_sid_unknown] [StandaloneDockerContainerFactory] Pulled OpenWhisk provided image openwhisk/python3action:latest

# 运行该容器，并进行资源、网络等初始化配置
[2022-04-15T06:04:43.328Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [DockerClientWithFileAccess] running /usr/bin/docker run -d --cpu-shares 25 --memory 128m --memory-swap 128m --network bridge -e __OW_API_HOST=http://172.17.0.1:3233 --name wsk0_3_alftest_hello --pids-limit 1024 --cap-drop NET_RAW --cap-drop NET_ADMIN --ulimit nofile=1024:1024 --log-driver json-file openwhisk/python3action:latest (timeout: 1 minute) [marker:invoker_docker.run_start:4243]
[2022-04-15T06:04:43.685Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [DockerClientWithFileAccess]  [marker:invoker_docker.run_finish:4600:357]

# inspect 获取镜像的元数据信息
[2022-04-15T06:04:43.685Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [DockerClientWithFileAccess] running /usr/bin/docker inspect --format {{.NetworkSettings.Networks.bridge.IPAddress}} 3c4ee07011e447f39dca77226605e002b02ba95b30d11805f3c13a55c1c1bb11 (timeout: 1 minute) [marker:invoker_docker.inspect_start:4600]
[2022-04-15T06:04:43.732Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [DockerClientWithFileAccess]  [marker:invoker_docker.inspect_finish:4646:46]

# 将 关于 hello 函数的 初始化信息 发送到 完成配置的容器中
[2022-04-15T06:04:43.736Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [DockerContainer] sending initialization to ContainerId(3c4ee07011e447f39dca77226605e002b02ba95b30d11805f3c13a55c1c1bb11) ContainerAddress(172.17.0.4,8080) [marker:invoker_activationInit_start:4650]
[2022-04-15T06:04:44.159Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [DockerContainer] initialization result: ok [marker:invoker_activationInit_finish:5074:422]

# 容器代理，不知道干什么的
[2022-04-15T06:04:44.161Z] [INFO] [#tid_sid_unknown] [ContainerProxy] resending up to 0 from 0 buffered jobs

# 向该容器发送 hello 函数的 调用参数信息，并运行 hello 函数
[2022-04-15T06:04:44.162Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [DockerContainer] sending arguments to /alftest/hello at ContainerId(3c4ee07011e447f39dca77226605e002b02ba95b30d11805f3c13a55c1c1bb11) ContainerAddress(172.17.0.4,8080) [marker:invoker_activationRun_start:5076]
[2022-04-15T06:04:44.166Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [DockerContainer] running result: ok [marker:invoker_activationRun_finish:5081:4]

# ContainerProxy，invoker 收集日志信息
[2022-04-15T06:04:44.175Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [ContainerProxy]  [marker:invoker_collectLogs_start:5090]

# 使用 docker 收集该容器的日志信息， logs
[2022-04-15T06:04:44.175Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [ExtendedDockerClient] running /usr/bin/docker logs 3c4ee07011e447f39dca77226605e002b02ba95b30d11805f3c13a55c1c1bb11 --since 1650002683 --until 1650002685 --timestamps (timeout: 2 seconds) [marker:invoker_docker.logs_start:5090]
[2022-04-15T06:04:44.213Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [ExtendedDockerClient]  [marker:invoker_docker.logs_finish:5128:38]
[2022-04-15T06:04:44.215Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [ContainerProxy]  [marker:invoker_collectLogs_finish:5130:40]

# 将该次 hello 函数的 activation 结果进行保存，document
[2022-04-15T06:04:44.219Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [MemoryArtifactStore] [PUT] 'activations' saving document: 'id: alftest/ca09d1cf2f534ee389d1cf2f531ee34a, rev: null' [marker:database_saveDocument_start:5133]
[2022-04-15T06:04:44.288Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [MemoryArtifactStore]  [marker:database_saveDocument_finish:5203:70]

# Post 对应 activation id 完成调用的信息
[2022-04-15T06:04:44.292Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [MessagingActiveAck] posted completion of activation ca09d1cf2f534ee389d1cf2f531ee34a

# LeanBalancer 收到该 activation id 的执行完成确认信息，completion ack
[2022-04-15T06:04:44.298Z] [INFO] [#tid_kB9RyCocBItsnSJV2Twlye2wh5Oa4yeN] [LeanBalancer] received completion ack for 'ca09d1cf2f534ee389d1cf2f531ee34a', system error=false

# Message 向 Actor 信息发送出现问题，具体是有什么用不清楚
[2022-04-15T06:04:44.305Z] [INFO] Message [org.apache.openwhisk.core.loadBalancer.InvocationFinishedMessage] to Actor[akka://standalone-actor-system/user/$g#2142173049] was unhandled. [1] dead letters encountered. This logging can be turned off or adjusted with configuration settings 'akka.log-dead-letters' and 'akka.log-dead-letters-during-shutdown'.

# 使用 docker 暂停(pause) 该容器实例
[2022-04-15T06:04:44.348Z] [INFO] [#tid_sid_invokerNanny] [DockerClientWithFileAccess] running /usr/bin/docker pause 3c4ee07011e447f39dca77226605e002b02ba95b30d11805f3c13a55c1c1bb11 (timeout: 10 seconds) [marker:invoker_docker.pause_start:88517]
[2022-04-15T06:04:44.397Z] [INFO] [#tid_sid_invokerNanny] [DockerClientWithFileAccess]  [marker:invoker_docker.pause_finish:88566:49]

# 十分钟后若未再次调用该容器，使用 docker rm 清除该实例
```





- 第二次冷启动记录

```bash
[2022-04-15T07:05:17.011Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] POST /api/v1/namespaces/alftest/actions/hello blocking=false&result=false
[2022-04-15T07:05:17.014Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [Identity] [GET] serving from cache: CacheKey(babecafe-cafe-babe-cafe-babecafebabe) [marker:database_cacheHit_counter:2]
[2022-04-15T07:05:17.028Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [WhiskActionMetaData] [GET] serving from datastore: CacheKey(alftest/hello) [marker:database_cacheMiss_counter:17]
[2022-04-15T07:05:17.028Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [MemoryArtifactStore] [GET] 'whisks' finding document: 'id: alftest/hello' [marker:database_getDocument_start:17]
[2022-04-15T07:05:17.029Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [MemoryArtifactStore]  [marker:database_getDocument_finish:18:1]
[2022-04-15T07:05:17.048Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [ActionsApi]  [marker:controller_activation_start:37]
[2022-04-15T07:05:17.049Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [ActionsApi] action activation id: 7afd09c013014950bd09c01301e950f4 [marker:controller_loadbalancer_start:38]
[2022-04-15T07:05:17.056Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [LeanBalancer] posting topic 'invoker0' with activation id '7afd09c013014950bd09c01301e950f4' [marker:controller_kafka_start:44]
[2022-04-15T07:05:17.065Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [LeanBalancer] posted to invoker0[0][-1] [marker:controller_kafka_finish:53:8]
[2022-04-15T07:05:17.066Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [ActionsApi]  [marker:controller_loadbalancer_finish:54:16]
[2022-04-15T07:05:17.066Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [ActionsApi]  [marker:controller_activation_finish:55:18]
[2022-04-15T07:05:17.068Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [BasicHttpService] [marker:http_post.202_counter:57:57]
[2022-04-15T07:05:17.073Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [InvokerReactive]  [marker:invoker_activation_start:62]
[2022-04-15T07:05:17.080Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [WhiskAction] [GET] serving from datastore: CacheKey(alftest/hello) [marker:database_cacheMiss_counter:65]
[2022-04-15T07:05:17.080Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [MemoryArtifactStore] [GET] 'whisks' finding document: 'id: alftest/hello, rev: sha256-9353f483d838cc5589bbd99cba65bcb4a971e2bfc713bce2c1673c8fe0f2d195' [marker:database_getDocument_start:69]
[2022-04-15T07:05:17.081Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [MemoryArtifactStore]  [marker:database_getDocument_finish:69:0]
[2022-04-15T07:05:17.086Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [MemoryArtifactStore] [ATT_GET] 'whisks' finding attachment 'mem:UEsDBBQAAAAIABBGjVSjRYEHEAIAAHEEAAAfABwAdmlydHVhbGVudi9iaW4vYWN0aXZhdGVfdGhpcy5weVVUCQADX45WYpiXVmJ1eAsAAQQAAAAABAAAAACNU01v2zAMvetXEB4K21jnDOstQA4dMGCHbeihlyEIDMWmE62yJEiKE__7kXKdpEWLzYBt8evxkRSzLPs6wiEoswM8YdMpjUXcq1Dz6RZa1cSiTkJdr86GsoTRHuCotBayiWqQEYGtMCgfD1KjGYBe5a3p0cRKiEe2NtLAFikftnDco0ko_SFEVgEZ8aRCZDIPY9xbA8pE9M4jfW_B2CjiHq9zbJVZuOQqsiwTIvpxKYCembPAU4Muwi_Z4zfvrZ_MXipKeB8C-qisSZYiWfjJfs-0_MFMdWn1hJcO5U7G_SLaxVx8zU6VG_PXLXvfsyyzUqjeWR8hjGE-2iCE1W1tQ82hsCJN9dzKaoexyB_uH79TnjwvxcW0ntSbyZ8jq1Z5Q1UXsyy3gf9nbjTEj7NzQMfCJa_YSmrQ-2D_BqfiOi6sclrGzvoeVivIj8rcfcmnIQRF7XCyeZI7DFe5_lhlCs5PRf5QW66VXT_NrlQ46oD_D6InkOmi3IQcbhKxAX2g4a-Xd5s3UtCtG2pym8eg6WYWqR6SL5OjKMGfSrYt_6kxxQtOpeAgj1LXBNmpE2ElmCSIy5H0zFd8gJ924HWijWhb2hRC6wNEm1QdDZtuSZcEprIUBo_XRNcbQe1OUbQ_r3hPTaPJJDNtFLu8KHV5XoNr3Eo6h6YtOKw8e8ywVF5PnJ-ts3a9_Mz38RpG_AVQSwMEFAAAAAgAgCyOVCTur6iZAAAA6wAAAAsAHABfX21haW5fXy5weVVUCQADwLJXYsCyV2J1eAsAAQQAAAAABAAAAAA9js0KwkAMhO_7FDGnFsUHKHgVH8C7RJtuC_tTsiki4ru727-cMh8zwwx-jKLgSXtjXtG1cIG7TGxMy13GQ6hIbKobA_msi09yUHyzflN6rKGdrfpKLvEMAnnOoNScLWuFBeAJMKlQsCxYL-XCrEOw2Ys3di4CwnFJH_PbSfQwfrSP4YBzQFgnCfDFLYnNXpLryxBstok_8wdQSwECHgMUAAAACAAQRo1Uo0WBBxACAABxBAAAHwAYAAAAAAABAAAApIEAAAAAdmlydHVhbGVudi9iaW4vYWN0aXZhdGVfdGhpcy5weVVUBQADX45WYnV4CwABBAAAAAAEAAAAAFBLAQIeAxQAAAAIAIAsjlQk7q-omQAAAOsAAAALABgAAAAAAAEAAADogWkCAABfX21haW5fXy5weVVUBQADwLJXYnV4CwABBAAAAAAEAAAAAFBLBQYAAAAAAgACALYAAABHAwAAAAA=' of document 'id: alftest/hello, rev: sha256-9353f483d838cc5589bbd99cba65bcb4a971e2bfc713bce2c1673c8fe0f2d195' [marker:database_getDocumentAttachment_start:75]
[2022-04-15T07:05:17.092Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [WhiskAction] write initiated on existing cache entry, invalidating CacheKey(alftest/hello), tid yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ, state WriteInProgress
[2022-04-15T07:05:17.092Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [WhiskAction] write all done, caching CacheKey(alftest/hello) Cached
[2022-04-15T07:05:17.130Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [DockerClientWithFileAccess] running /usr/bin/docker pull openwhisk/python3action:latest (timeout: 10 minutes) [marker:invoker_docker.pull_start:118]
[2022-04-15T07:05:17.132Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [ContainerPool] containerStart containerState: cold container: None activations: 1 of max 1 action: hello namespace: alftest activationId: 7afd09c013014950bd09c01301e950f4 [marker:invoker_containerStart.cold_counter:120]
[2022-04-15T07:05:20.283Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [DockerClientWithFileAccess]  [marker:invoker_docker.pull_finish:3271:3152]
[2022-04-15T07:05:20.283Z] [INFO] [#tid_sid_unknown] [StandaloneDockerContainerFactory] Pulled OpenWhisk provided image openwhisk/python3action:latest
[2022-04-15T07:05:20.284Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [DockerClientWithFileAccess] running /usr/bin/docker run -d --cpu-shares 25 --memory 128m --memory-swap 128m --network bridge -e __OW_API_HOST=http://172.17.0.1:3233 --name wsk0_3_alftest_hello --pids-limit 1024 --cap-drop NET_RAW --cap-drop NET_ADMIN --ulimit nofile=1024:1024 --log-driver json-file openwhisk/python3action:latest (timeout: 1 minute) [marker:invoker_docker.run_start:3273]
[2022-04-15T07:05:20.640Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [DockerClientWithFileAccess]  [marker:invoker_docker.run_finish:3629:355]
[2022-04-15T07:05:20.652Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [DockerClientWithFileAccess] running /usr/bin/docker inspect --format {{.NetworkSettings.Networks.bridge.IPAddress}} 0fb2bf06f4e6e2aeecce307046cc3c4443f77bd6363f4c35e23a00d302fe5f3f (timeout: 1 minute) [marker:invoker_docker.inspect_start:3629]
[2022-04-15T07:05:20.683Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [DockerClientWithFileAccess]  [marker:invoker_docker.inspect_finish:3672:43]
[2022-04-15T07:05:20.687Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [DockerContainer] sending initialization to ContainerId(0fb2bf06f4e6e2aeecce307046cc3c4443f77bd6363f4c35e23a00d302fe5f3f) ContainerAddress(172.17.0.4,8080) [marker:invoker_activationInit_start:3675]
[2022-04-15T07:05:21.143Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [DockerContainer] initialization result: ok [marker:invoker_activationInit_finish:4132:451]
[2022-04-15T07:05:21.145Z] [INFO] [#tid_sid_unknown] [ContainerProxy] resending up to 0 from 0 buffered jobs
[2022-04-15T07:05:21.146Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [DockerContainer] sending arguments to /alftest/hello at ContainerId(0fb2bf06f4e6e2aeecce307046cc3c4443f77bd6363f4c35e23a00d302fe5f3f) ContainerAddress(172.17.0.4,8080) [marker:invoker_activationRun_start:4134]
[2022-04-15T07:05:21.150Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [DockerContainer] running result: ok [marker:invoker_activationRun_finish:4139:4]
[2022-04-15T07:05:21.155Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [ContainerProxy]  [marker:invoker_collectLogs_start:4144]
[2022-04-15T07:05:21.157Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [ExtendedDockerClient] running /usr/bin/docker logs 0fb2bf06f4e6e2aeecce307046cc3c4443f77bd6363f4c35e23a00d302fe5f3f --since 1650006320 --until 1650006322 --timestamps (timeout: 2 seconds) [marker:invoker_docker.logs_start:4145]
[2022-04-15T07:05:21.197Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [ExtendedDockerClient]  [marker:invoker_docker.logs_finish:4186:40]
[2022-04-15T07:05:21.199Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [ContainerProxy]  [marker:invoker_collectLogs_finish:4188:43]
[2022-04-15T07:05:21.203Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [MemoryArtifactStore] [PUT] 'activations' saving document: 'id: alftest/7afd09c013014950bd09c01301e950f4, rev: null' [marker:database_saveDocument_start:4192]
[2022-04-15T07:05:21.272Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [MemoryArtifactStore]  [marker:database_saveDocument_finish:4261:69]
[2022-04-15T07:05:21.276Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [MessagingActiveAck] posted completion of activation 7afd09c013014950bd09c01301e950f4
[2022-04-15T07:05:21.285Z] [INFO] [#tid_yGs2X0bFrFoCcoB8Cui6RPBBKO2sCAxZ] [LeanBalancer] received completion ack for '7afd09c013014950bd09c01301e950f4', system error=false
[2022-04-15T07:05:21.288Z] [INFO] Message [org.apache.openwhisk.core.loadBalancer.InvocationFinishedMessage] to Actor[akka://standalone-actor-system/user/$g#256761085] was unhandled. [1] dead letters encountered. This logging can be turned off or adjusted with configuration settings 'akka.log-dead-letters' and 'akka.log-dead-letters-during-shutdown'.
[2022-04-15T07:05:21.328Z] [INFO] [#tid_sid_invokerNanny] [DockerClientWithFileAccess] running /usr/bin/docker pause 0fb2bf06f4e6e2aeecce307046cc3c4443f77bd6363f4c35e23a00d302fe5f3f (timeout: 10 seconds) [marker:invoker_docker.pause_start:47270]
[2022-04-15T07:05:21.377Z] [INFO] [#tid_sid_invokerNanny] [DockerClientWithFileAccess]  [marker:invoker_docker.pause_finish:47319:49]

```





- 热启动记录

```bash
[2022-04-15T07:08:37.096Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] POST /api/v1/namespaces/alftest/actions/hello blocking=false&result=false
[2022-04-15T07:08:37.099Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [Identity] [GET] serving from cache: CacheKey(babecafe-cafe-babe-cafe-babecafebabe) [marker:database_cacheHit_counter:3]
[2022-04-15T07:08:37.101Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [WhiskActionMetaData] [GET] serving from cache: CacheKey(alftest/hello) [marker:database_cacheHit_counter:6]
[2022-04-15T07:08:37.102Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [ActionsApi]  [marker:controller_activation_start:6]
[2022-04-15T07:08:37.102Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [ActionsApi] action activation id: cb9e791fec2640a19e791fec2600a1e1 [marker:controller_loadbalancer_start:6]
[2022-04-15T07:08:37.103Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [LeanBalancer] posting topic 'invoker0' with activation id 'cb9e791fec2640a19e791fec2600a1e1' [marker:controller_kafka_start:7]
[2022-04-15T07:08:37.104Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [InvokerReactive]  [marker:invoker_activation_start:9]
[2022-04-15T07:08:37.104Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [WhiskAction] [GET] serving from cache: CacheKey(alftest/hello) [marker:database_cacheHit_counter:9]

# 第二次启动时，查询到 warmed container
[2022-04-15T07:08:37.105Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [ContainerPool] containerStart containerState: warmed container: Some(ContainerId(0fb2bf06f4e6e2aeecce307046cc3c4443f77bd6363f4c35e23a00d302fe5f3f)) activations: 1 of max 1 action: hello namespace: alftest activationId: cb9e791fec2640a19e791fec2600a1e1 [marker:invoker_containerStart.warmed_counter:10]
[2022-04-15T07:08:37.105Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [DockerClientWithFileAccess] running /usr/bin/docker unpause 0fb2bf06f4e6e2aeecce307046cc3c4443f77bd6363f4c35e23a00d302fe5f3f (timeout: 10 seconds) [marker:invoker_docker.unpause_start:10]
[2022-04-15T07:08:37.107Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [LeanBalancer] posted to invoker0[0][-1] [marker:controller_kafka_finish:10:3]
[2022-04-15T07:08:37.108Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [ActionsApi]  [marker:controller_loadbalancer_finish:10:4]
[2022-04-15T07:08:37.108Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [ActionsApi]  [marker:controller_activation_finish:10:4]
[2022-04-15T07:08:37.108Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [BasicHttpService] [marker:http_post.202_counter:11:11]
[2022-04-15T07:08:37.168Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [DockerClientWithFileAccess]  [marker:invoker_docker.unpause_finish:70:60]
[2022-04-15T07:08:37.168Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [DockerContainer] sending arguments to /alftest/hello at ContainerId(0fb2bf06f4e6e2aeecce307046cc3c4443f77bd6363f4c35e23a00d302fe5f3f) ContainerAddress(172.17.0.4,8080) [marker:invoker_activationRun_start:71]
[2022-04-15T07:08:37.178Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [DockerContainer] running result: ok [marker:invoker_activationRun_finish:82:10]
[2022-04-15T07:08:37.178Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [ContainerProxy]  [marker:invoker_collectLogs_start:83]
[2022-04-15T07:08:37.178Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [ExtendedDockerClient] running /usr/bin/docker logs 0fb2bf06f4e6e2aeecce307046cc3c4443f77bd6363f4c35e23a00d302fe5f3f --since 1650006517 --until 1650006518 --timestamps (timeout: 2 seconds) [marker:invoker_docker.logs_start:83]
[2022-04-15T07:08:37.227Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [ExtendedDockerClient]  [marker:invoker_docker.logs_finish:131:48]
[2022-04-15T07:08:37.229Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [ContainerProxy]  [marker:invoker_collectLogs_finish:131:48]
[2022-04-15T07:08:37.229Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [MemoryArtifactStore] [PUT] 'activations' saving document: 'id: alftest/cb9e791fec2640a19e791fec2600a1e1, rev: null' [marker:database_saveDocument_start:132]
[2022-04-15T07:08:37.229Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [MemoryArtifactStore]  [marker:database_saveDocument_finish:132:0]
[2022-04-15T07:08:37.229Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [MessagingActiveAck] posted completion of activation cb9e791fec2640a19e791fec2600a1e1
[2022-04-15T07:08:37.229Z] [INFO] [#tid_6QW5BRrxLKt5QSQAJGJQlYtjR3Rh39gF] [LeanBalancer] received completion ack for 'cb9e791fec2640a19e791fec2600a1e1', system error=false
[2022-04-15T07:08:37.229Z] [INFO] Message [org.apache.openwhisk.core.loadBalancer.InvocationFinishedMessage] to Actor[akka://standalone-actor-system/user/$g#256761085] was unhandled. [2] dead letters encountered. This logging can be turned off or adjusted with configuration settings 'akka.log-dead-letters' and 'akka.log-dead-letters-during-shutdown'.
[2022-04-15T07:08:37.296Z] [INFO] [#tid_sid_invokerNanny] [DockerClientWithFileAccess] running /usr/bin/docker pause 0fb2bf06f4e6e2aeecce307046cc3c4443f77bd6363f4c35e23a00d302fe5f3f (timeout: 10 seconds) [marker:invoker_docker.pause_start:243238]
[2022-04-15T07:08:37.348Z] [INFO] [#tid_sid_invokerNanny] [DockerClientWithFileAccess]  [marker:invoker_docker.pause_finish:243290:51]

```



#### 4. OpenWhisk 多节点模式

- OpenWhisk中使用了 Akka 来实现高并发与分布式，Controller 与 Invoker都有各自的 ActorSystem，每个ActorSystem之间的各个actor通过 actorRef 对象进行消息传递。
- Controller 与 Invoker 之间通过 Kafka 进行消息传递



#### 5. OpenWhisk 之 Invoker

##### 1. Containpool 模块

###### 1.1 组织结构

- 容器池管理中共有两个基本文件，一个FaasCache新增代码
  - ContainerPool ：
    - 用于容器池的管理，通过容器代理创建容器并进行管理。
    - 同时ContainerPool自身也是一个Actor
  - ContainerProxy：
    - 容器代理，定义了一个容器生命周期的各个状态
    - 通过抽象工厂创建单个容器，以单个容器为粒度进行容器的使用与管理
      - 也可能不是单个，留后细看
    - 每个ContainerProxy也是一个 Actor，用于与ContainerPool 及其他Actor的通信
  - UpdatablePriorityQueue：FaasCache定义的一个记录单Invoker中Action执行信息的类，用于对Action优先级的排序

###### 1.2 ContainerProxy

- 主要定义了一个 类及一个实例，用于

- Class ContainerProxy：定义了一个容器Actor的生命周期的各个状态，以及与其他Actor信息交互的样例类

  - 容器状态：

    ```scala
    // States
    sealed trait ContainerState
    case object Uninitialized extends ContainerState
    case object Starting extends ContainerState
    case object Started extends ContainerState
    case object Running extends ContainerState
    case object Ready extends ContainerState
    case object Pausing extends ContainerState
    case object Paused extends ContainerState
    case object Removing extends ContainerState
    ```

  - 容器数据抽象信息：

    ```scala
    // Data
    /** Base data type */
    sealed abstract class ContainerData(val lastUsed: Instant, val memoryLimit: ByteSize, val activeActivationCount: Int) {
    
      /** When ContainerProxy in this state is scheduled, it may result in a new state (ContainerData)*/
      def nextRun(r: Run): ContainerData
    
      /**
       *  Return Some(container) (for ContainerStarted instances) or None(for ContainerNotStarted instances)
       *  Useful for cases where all ContainerData instances are handled, vs cases where only ContainerStarted
       *  instances are handled */
      def getContainer: Option[Container]
    
      /** String to indicate the state of this container after scheduling */
      val initingState: String
    
      /** Inidicates whether this container can service additional activations */
      def hasCapacity(): Boolean
    }
    ```

  - 容器使用状态及类型定义：

    ```scala
    /** abstract type to indicate an unstarted container */
    sealed abstract class ContainerNotStarted(override val lastUsed: Instant,
                                              override val memoryLimit: ByteSize,
                                              override val activeActivationCount: Int)
        extends ContainerData(lastUsed, memoryLimit, activeActivationCount) {
      override def getContainer = None
      override val initingState = "cold"
    }
    
    /** abstract type to indicate a started container */
    sealed abstract class ContainerStarted(val container: Container,
                                           override val lastUsed: Instant,
                                           override val memoryLimit: ByteSize,
                                           override val activeActivationCount: Int)
        extends ContainerData(lastUsed, memoryLimit, activeActivationCount) {
      override def getContainer = Some(container)
    }
    
    /** trait representing a container that is in use and (potentially) usable by subsequent or concurrent activations */
    sealed abstract trait ContainerInUse {
      val activeActivationCount: Int
      val action: ExecutableWhiskAction
      def hasCapacity() =
        activeActivationCount < action.limits.concurrency.maxConcurrent
    }
    
    /** trait representing a container that is NOT in use and is usable by subsequent activation(s) */
    sealed abstract trait ContainerNotInUse {
      def hasCapacity() = true
    }
    
    /** type representing a cold (not running) container */
    case class NoData(override val activeActivationCount: Int = 0)
        extends ContainerNotStarted(Instant.EPOCH, 0.B, activeActivationCount)
        with ContainerNotInUse {
      override def nextRun(r: Run) = WarmingColdData(r.msg.user.namespace.name, r.action, Instant.now, 1)
    }
    
    /** type representing a cold (not running) container with specific memory allocation */
    case class MemoryData(override val memoryLimit: ByteSize, override val activeActivationCount: Int = 0)
        extends ContainerNotStarted(Instant.EPOCH, memoryLimit, activeActivationCount)
        with ContainerNotInUse {
      override def nextRun(r: Run) = WarmingColdData(r.msg.user.namespace.name, r.action, Instant.now, 1)
    }
    
    /** type representing a prewarmed (running, but unused) container (with a specific memory allocation) */
    case class PreWarmedData(override val container: Container,
                             kind: String,
                             override val memoryLimit: ByteSize,
                             override val activeActivationCount: Int = 0)
        extends ContainerStarted(container, Instant.EPOCH, memoryLimit, activeActivationCount)
        with ContainerNotInUse {
      override val initingState = "prewarmed"
      override def nextRun(r: Run) =
        WarmingData(container, r.msg.user.namespace.name, r.action, Instant.now, 1)
    }
    
    /** type representing a prewarm (running, but not used) container that is being initialized (for a specific action + invocation namespace) */
    case class WarmingData(override val container: Container,
                           invocationNamespace: EntityName,
                           action: ExecutableWhiskAction,
                           override val lastUsed: Instant,
                           override val activeActivationCount: Int = 0)
        extends ContainerStarted(container, lastUsed, action.limits.memory.megabytes.MB, activeActivationCount)
        with ContainerInUse {
      override val initingState = "warming"
      override def nextRun(r: Run) = copy(lastUsed = Instant.now, activeActivationCount = activeActivationCount + 1)
    }
    
    /** type representing a cold (not yet running) container that is being initialized (for a specific action + invocation namespace) */
    case class WarmingColdData(invocationNamespace: EntityName,
                               action: ExecutableWhiskAction,
                               override val lastUsed: Instant,
                               override val activeActivationCount: Int = 0)
        extends ContainerNotStarted(lastUsed, action.limits.memory.megabytes.MB, activeActivationCount)
        with ContainerInUse {
      override val initingState = "warmingCold"
      override def nextRun(r: Run) = copy(lastUsed = Instant.now, activeActivationCount = activeActivationCount + 1)
    }
    
    /** type representing a warm container that has already been in use (for a specific action + invocation namespace) */
    case class WarmedData(override val container: Container,
                          invocationNamespace: EntityName,
                          action: ExecutableWhiskAction,
                          override val lastUsed: Instant,
                          override val activeActivationCount: Int = 0,
                          resumeRun: Option[Run] = None)
        extends ContainerStarted(container, lastUsed, action.limits.memory.megabytes.MB, activeActivationCount)
        with ContainerInUse {
      override val initingState = "warmed"
      override def nextRun(r: Run) = copy(lastUsed = Instant.now, activeActivationCount = activeActivationCount + 1)
      //track the resuming run for easily referring to the action being resumed (it may fail and be resent)
      def withoutResumeRun() = this.copy(resumeRun = None)
      def withResumeRun(job: Run) = this.copy(resumeRun = Some(job))
    }
    ```

  - 容器代理 Actor 的通信协议

    ```scala
    // Events received by the actor
    case class Start(exec: CodeExec[_], memoryLimit: ByteSize)
    case class Run(action: ExecutableWhiskAction, msg: ActivationMessage, retryLogDeadline: Option[Deadline] = None)
    case object Remove
    case class HealthPingEnabled(enabled: Boolean)
    
    // Events sent by the actor
    case class NeedWork(data: ContainerData)
    case object ContainerPaused
    case object ContainerRemoved // when container is destroyed
    case object RescheduleJob // job is sent back to parent and could not be processed because container is being destroyed
    case class PreWarmCompleted(data: PreWarmedData)
    case class InitCompleted(data: WarmedData)
    case object RunCompleted
    ```

- class ContainerProxy 

  - 继承自 FSM[ContainerState, ContainerData] 与 Trait Stash，FSM 有 Actor的继承，因此ContainerProxy也是一个Actor
  - 它还详细实现了容器的各种管理及使用方法，实现了容器生命周期的管理，接收函数的调用请求并执行

- object ContainerProxy

  - def props 方法实现了创建一个容器代理Actor对象
  - 其余方法留待后续介绍

- class TCPPingClient

  - 一个Actor，用于心跳信息的实现

- object TCPPingClient

  - def props，用于创建一个TCPPingClient Actor对象

- 容器异常信息

  - 用于Activation 请求错误时使用

  - ```scala
    /** Indicates that something went wrong with an activation and the container should be removed */
    trait ActivationError extends Exception {
      val activation: WhiskActivation
    }
    
    /** Indicates an activation with a non-successful response */
    case class ActivationUnsuccessfulError(activation: WhiskActivation) extends ActivationError
    
    /** Indicates reading logs for an activation failed (terminally, truncated) */
    case class ActivationLogReadingError(activation: WhiskActivation) extends ActivationError
    ```

    

###### 1.3 ContainerPool

- 状态定义：

  - 定义了 Worker 的状态，有 Free、Busy 两种

  - ```scala
    sealed trait WorkerState
    case object Busy extends WorkerState
    case object Free extends WorkerState
    
    case class WorkerData(data: ContainerData, state: WorkerState)
    ```
  
- class ContainerPool

  - 对由容器池创建的 容器代理 进行管理，也就是间接的管理容器

  - ```scala
    class ContainerPool(childFactory: ActorRefFactory => ActorRef,
                        feed: ActorRef,
                        prewarmConfig: List[PrewarmingConfig] = List.empty,
                        poolConfig: ContainerPoolConfig)
        extends Actor {
    ```

    - 参数含义：
      - childFactory：创建容器代理Actor的Method
      - feed：一个ActorRef，用来请求工作负载
      - prewarmingConfig：可选设置，容器的预热配置信息
      - poolConfig：容器池的配置信息

  - 

- object ContainerPool

- case class PrewarmingConfig

###### 1.4 UpdatablePriorityQueue

- 定义了一个类及一个实例，用于记录Action的执行信息来计算优先级

- 在 `ContainerPool.scala`中被使用到

  ```scala
  package org.apache.openwhisk.core.containerpool
  
  class TrackedAction() {
      var lastcalled : Double = 0.0;
      var invocations : Long = 0;
      var coldTime : Long = 0;
      var warmTime : Long = 0;
      var memory : Double = 0.0;
      var active : Long = 0;
  
      def priority() : Double = {
          lastcalled + ((invocations * (coldTime - warmTime))/ memory)
      }
  
  }
  
  object ActionOrdering extends Ordering[TrackedAction] {
    def compare(a:TrackedAction, b:TrackedAction) = a.priority() compare b.priority()
  }
  ```

##### 2. Invoker模块

###### 2.1 Invoker

###### 2.2 InvokerReactive

###### 2.3 InstanceIdAssigner

###### 2.4 LogStoreCollector

###### 2.5 NamespaceBlacklist



#### 6. OpenWhisk 之 Controller

##### 1. Controller 模块

##### 2. Entitlement 模块

##### 3. LoadBalancer 模块

###### 3.1 组织结构

- 该文件下一共有七个代码文件

  1. Trait：FeedFactory，未知

  2. Trait：InvokerPoolFactory，工作结点池抽象工厂

  3. InvokerSupervision，用来管理Invoker池及Invoker节点

  4. Trait：LoadBalancer，定义了LoadBalancer、LoadBalancerProvider 两个trait。用于提供负载均衡的抽象定义
     1. CommonLoadBalancer：抽下类，为负载均衡提供了通用的实现
        1. 子类：LeanBalancer：不通过Kafka的调度，直接将Invoker与Controller Build 在一起
        2. 子类：ShardingContainerPoolBalancer：水平切分的容器池调度策略

- 特别注意：

  - InvokerPoolFactor 与 InvokerSupervision 都是在Sharding调度中使用到的
  - LeanBalancer 中并没有关于 工作结点池 的管理

###### 3.2 LoadBalancer.scala

- 该文件中共有四个 类和特质

  - InvokerHealth

  ```scala
  /* InvokerHealth 是一个抽象的Invoker，一个Invoker是一个 local 容器池的管理者
   *  参数含义
   *  @param id：代表了Invoker的独特标识符
   *  @param status：代表了该容器的状态， health、unhealthy、offline三种
  */
  class InvokerHealth(val id: InvokerInstanceId, val status: InvokerState) {
    override def equals(obj: scala.Any): Boolean = obj match {
      case that: InvokerHealth => that.id == this.id && that.status ==this.status
      case _                   => false
    }
  
    override def toString = s"InvokerHealth($id, $status)"
  }
  ```

  - LoadBalancer
    - 主要方法：publish，将一个调用信息发布，用于后续转发到一个invoker中

  ```scala
  // 一个负载均衡的特质，主要是用来定义一个负载均衡策略应该实现的方法
  
  trait LoadBalancer {
  
    /**
     * Publishes activation message on internal bus for an invoker to pick up.
     *
     * @param action the action to invoke
     * @param msg the activation message to publish on an invoker topic
     * @param transid the transaction id for the request
     * @return result a nested Future the outer indicating completion of publishing and
     *         the inner the completion of the action (i.e., the result)
     *         if it is ready before timeout (Right) otherwise the activation id (Left).
     *         The future is guaranteed to complete within the declared action time limit
     *         plus a grace period (see activeAckTimeoutGrace).
     */
    def publish(action: ExecutableWhiskActionMetaData, msg: ActivationMessage)(
      implicit transid: TransactionId): Future[Future[Either[ActivationId, WhiskActivation]]]
  
    /**
     * Returns a message indicating the health of the containers and/or container pool in general.
     *
     * @return a Future[IndexedSeq[InvokerHealth]] representing the health of the pools managed by the loadbalancer.
     */
    def invokerHealth(): Future[IndexedSeq[InvokerHealth]]
  
    /** Gets the number of in-flight activations for a specific user. */
    def activeActivationsFor(namespace: UUID): Future[Int]
  
    /** Gets the number of in-flight activations in the system. */
    def totalActiveActivations: Future[Int]
  
    /** Gets the size of the cluster all loadbalancers are acting in */
    def clusterSize: Int = 1
  }
  ```

  - LoadBalancerProvider

  ```scala
  /** 负载均衡策略的对外实现接口
   * An Spi for providing load balancer implementations.
   */
  trait LoadBalancerProvider extends Spi {
    def requiredProperties: Map[String, String]
  
    def instance(whiskConfig: WhiskConfig, instance: ControllerInstanceId)(implicit actorSystem: ActorSystem,
                                                                           logging: Logging,
                                                                           materializer: ActorMaterializer): LoadBalancer
  
    /** Return default FeedFactory */
    def createFeedFactory(whiskConfig: WhiskConfig, instance: ControllerInstanceId)(implicit actorSystem: ActorSystem,
                                                                                    logging: Logging): FeedFactory = {
  
      val activeAckTopic = s"completed${instance.asString}"
      val maxActiveAcksPerPoll = 128
      val activeAckPollDuration = 1.second
  
      new FeedFactory {
        def createFeed(f: ActorRefFactory, provider: MessagingProvider, acker: Array[Byte] => Future[Unit]) = {
          f.actorOf(Props {
            new MessageFeed(
              "activeack",
              logging,
              provider.getConsumer(whiskConfig, activeAckTopic, activeAckTopic, maxPeek = maxActiveAcksPerPoll),
              maxActiveAcksPerPoll,
              activeAckPollDuration,
              acker)
          })
        }
      }
    }
  }
  ```

  - 样例类：LoadBalancerException

  ```scala
  /** Exception thrown by the loadbalancer 
  * 样例类：抛出负载均衡时会出现的异常信息
  */
  case class LoadBalancerException(msg: String) extends Throwable(msg)
  ```

###### 3.3 CommonLoadBalancer



###### 3.4 LeanBalancer 负载策略

- 继承关系：
  - LeanBalancer 继承自 抽象类CommonLoadBalancer，CLB 继承自 Trait LoadBalancer，
  - LoadBalancerProvider 继承自 Trait Spi。
  - 目前我理解的Spi，是指 Service Provision Interface，服务提供接口。说明LoadBalancer作为一个服务，应该能够被外部作为接口来调用