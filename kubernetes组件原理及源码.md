







# 0.client-go源码解析



# 0.kubernetes 源码结构

## 1.代码结构

- pkg目录：主体代码 里面实现了kubernetes的主体逻辑
- cmd目录：kubernetes所有后台进程的代码 主要是各个子模块的启动代码 具体逻辑在pkg下
- plugin目录： 主要是kube-schedule和一些插件

### 1.pkg

- api： api主要包括最新版本的Rest api接口的类 并提供数据格式转换工具类 对应版本号文件夹下的文件描述了特定的版本如何序列化存储和网络
- client： k8s中公用的客户端部分 实现对对象的具体操作crud
- cloudprovider： k8s提供对aws azure gce mesos 等云提供商提供了接口支持 目前包括负载均衡 实例 zone信息 路由信息等
- controller：k8s controller主要包括各个controller实现逻辑 为各类资源如 replication endpoint node 等的增删改查等逻辑提供派发和执行
- credentialprovider： 为docker镜像仓库贡献者提供权威认证
- generated： generated包是所有生成的文件的目标文件 一般这里面的文件日常是不进行改动的
- kubectl： 是命令行工具
- kubelet： 负责node层的pod管理 完成pod及容器的创建 执行pod的删除同步操作
- master： 负责集群中master节点的运行管理 api安装 各个组件的于小宁端口分配 noderegistry podregistry等创建工作
- runtime： 实现不同版本api之间的适配 实现不同api版本之间数据结构的转换

### 2. cmd

- 包括k8s所有后台进程的代码 apiserver controller manager proxy kubelet 等进程

### 3. plugin

- 主要包括调度模块的代码实现 用于执行具体的scheduler的调度工作



# 1.API SERVER原理及源码

- **介绍**

API Server提供了k8s各类资源对象的增删改查及watch等http rest接口 是整个系统的数据总线和数据中心

1. 提供了集群管理的rest api接口（认证授权 数据校验 集群状态变更）
2. 提供其他模块之间的数据交互和通信的枢纽（api server才能直接操作etcd）
3. 是资源配额控制的入口
4. 拥有完备的集群安全机制



# 2. Controller Manager原理及源码

- controller manager作为集群内部的管理控制中心，负责集群内的node、pod副本、服务端点endpoint、namespace、serviceaccount、resourcequota的管理，当某个node意外宕机时，controller manager会及时发现并执行自动化修复流程。

![](image\controller-manager.png)

1. replication controller
2. node controller
3. resourcequota controller & limitranger
4. namespace controller
5. endpoint controller
6. service controller

# 3. Scheduler原理及源码

- Scheduler负责pod调度。在整个系统中起“承上启下”作用，承上：负责接收Controller Manager创建的新的pod，为其选择一个合适的node。启下：node上的kubelet接管pod的生命周期。

  scheduler主要负责整个集群的资源的调度功能 将pod调度到最优的工作节点上去 更加合理的运用资源 

- 调度流程

  根据特定算法和策略将pod调度到合适的node上 是一个独立的二进制程序 启动之后一直监听api-server  获取PodSpec.NodeName为空的pod 对pod都创建一个binding 

  在生产环境下需要考虑的问题：

  1. 如何保证全部节点的调度的公平性 要知道并不是说所有节点资源分配都是一样的
  2. 如何保证每个节点都能被分配资源

  kubernetes的调度器可以采用插件化形式实现 可以方便用户进行定制和二次开发 可以自定义一个调度器并以插件形式和kubernetes进行集成 scheduler的源码位于kubernetes/pkg/scheduler 大体目录为

  ~~~shell
  kubernetes/pkg/scheduler
  -- scheduler.go		//调度相关的具体实现
  |-- algorithm    
  |	|-- predicates   //节点筛选策略
  |	|--priorities    //节点打分策略
  |-- algorithmprovider
  | 	|--defaults	 	//定义默认的调度器
  ~~~

  其中 scheduler创建和运行的核心程序对应的代码在pkg/scheduler/scheduler.go 如果是入口程序 查看 cmd/kube-scheduler/scheduler.go

  调度主要分为三部分

  1. 预选过程 过滤掉不满足条件的节点 这个过程称为predicates
  2. 优选过程 对通过的节点按照优先级排序 称之为priorities
  3. 最后从中选择优先级最高的节点 如果中间任何一步有错误 直接返回错误

  关于筛选算法的部分可以查看：

   k8s.io/kubernetes/pkg/scheduler/algorithm/predicates/predicates.go 

  关于优先级的算法可以查看：

   k8s.io/kubernetes/pkg/scheduler/algorithm/priorities/ 

  参考文章： https://mp.weixin.qq.com/s/zXy5iYDTFxffzI7_IeLU7Q 

  ​		

# 4. kubelet原理及源码

- 在kubernetes集群中，每个node节点上都会启动kubelet进程，用来处理master节点下到本节点的任务，管理pod和其中的容器。kubelet会在api server上注册节点信息，定期向master汇报节点资源使用情况，并通过cadvisor监控容器和节点资源。

# 5. Kube-Proxy原理及源码

1. Proxy是为了解决外部网络能够访问跨机器集群中容器提供的应用服务而设计的，运行在每个Minion/Node上。Proxy提供TCP/UDP   sockets的proxy，每创建一种Service，Proxy主要从etcd获取Services和Endpoints的配置信息（也可以从file获取），然后根据配置信息在Minion/Node上启动一个Proxy的进程并监听相应的服务端口，当外部请求发生时，Proxy会根据Load  Balancer将请求分发到后端正确的容器处理。
2. Proxy不但解决了同一主宿机相同服务端口冲突的问题，还提供了Service转发服务端口对外提供服务的能力，Proxy后端使用了随机、轮循负载均衡算法。

- 原理：

1. kube-proxy 其实就是管理service的访问入口，包括集群内pod到service访问和集群外访问service
2. kube-proxy管理service的Endpoints，service暴露一个vip（cluster ip），集群内通过访问这个Cluster ip：port就能访问到集群对应的service下的pod
3. service是通过Selector选择的一组pods的服务抽象，其实就是一个微服务，提供了服务的LB和反向代理能力 而kube-proxy的主要作用就是负责service的实现

- 服务发现

1. 环境变量 ： 在创建pod的时候kubelet会在该pod中注入集群内所有service的环境变量（不通用）
2. DNS：使用CoreDNS进行服务发现（ https://blog.csdn.net/waltonwang/article/details/54317082 ）

- 暴露服务

1. ClusterIP： 默认的ServiceType 通过集群内的ClusterIP在内部发布服务
2. NodePort： 对集群外暴露服务
3. LoadBalancer： 需要云的支持
4. ExternalName： 集群内发布服务 借助于CoreDns

- 内部原理（iptables 和 ipvs）

  - iptables模式
    - iptables模式使用nat完成转发， 以prometheus nodeport为例

  1. 首先，通过node的32018端口访问 会进入到以下链中

  ```shell 
  # iptables -S -t nat
  -A KUBE-NODEPORTS -p tcp -m comment --comment "monitoring/prometheus-server:webui" -m tcp --dport 32018 -j KUBE-MARK-MASQ
  -A KUBE-NODEPORTS -p tcp -m comment --comment "monitoring/prometheus-server:webui" -m tcp --dport 32018 -j KUBE-SVC-B2LATO2XNXKAW5B6
```
  
2. 然后转到KUBE-SVC-B2LATO2XNXKAW5B6的链，三个链分别对应三个pod --probability 是概率 
  
  ```
  -A KUBE-SVC-B2LATO2XNXKAW5B6 -m comment --comment "monitoring/prometheus-server:webui" -m statistic --mode random --probability 0.33332999982 -j KUBE-SEP-6LNVVXXZ6N5KISWB
  -A KUBE-SVC-B2LATO2XNXKAW5B6 -m comment --comment "monitoring/prometheus-server:webui" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-7JGOJU5MHN5IX7QC
  -A KUBE-SVC-B2LATO2XNXKAW5B6 -m comment --comment "monitoring/prometheus-server:webui" -j KUBE-SEP-56HYIA4V5KXKNGTU
```
  
3. 进入KUBE-SEP-6LNVVXXZ6N5KISWB链中，具体作用是将请求 DNAT到10.253.186.195：9090中
  
  ```
  -A KUBE-SEP-6LNVVXXZ6N5KISWB -s 10.253.186.195/32 -m comment --comment "monitoring/prometheus-server:webui" -j KUBE-MARK-MASQ
  -A KUBE-SEP-6LNVVXXZ6N5KISWB -p tcp -m comment --comment "monitoring/prometheus-server:webui" -m tcp -j DNAT --to-destination 10.253.186.195:9090
```
  
4. 其他两个链是同样的道理
  
  ```
  -A KUBE-SEP-56HYIA4V5KXKNGTU -s 10.253.41.131/32 -m comment --comment "monitoring/prometheus-server:webui" -j KUBE-MARK-MASQ
  -A KUBE-SEP-56HYIA4V5KXKNGTU -p tcp -m comment --comment "monitoring/prometheus-server:webui" -m tcp -j DNAT --to-destination 10.253.41.131:9090
```
  
  ```
  -A KUBE-SEP-7JGOJU5MHN5IX7QC -s 10.253.239.130/32 -m comment --comment "monitoring/prometheus-server:webui" -j KUBE-MARK-MASQ
  -A KUBE-SEP-7JGOJU5MHN5IX7QC -p tcp -m comment --comment "monitoring/prometheus-server:webui" -m tcp -j DNAT --to-destination 10.253.239.130:9090
```
  
5.  分析完nodePort的工作方式，接下里说一下clusterIP的访问方式。 对于直接访问cluster IP(10.254.162.44)的3306端口会直接跳转到KUBE-SVC-B2LATO2XNXKAW5B6
  
  ```
  -A KUBE-SERVICES ! -s 10.254.0.0/16 -d 10.254.248.39/32 -p tcp -m comment --comment "monitoring/prometheus-server:webui cluster IP" -m tcp --dport 9090 -j KUBE-MARK-MASQ
  -A KUBE-SERVICES -d 10.254.248.39/32 -p tcp -m comment --comment "monitoring/prometheus-server:webui cluster IP" -m tcp --dport 9090 -j KUBE-SVC-B2LATO2XNXKAW5B6
```
  
![](image\kube-proxy-iptables.png)
  
  - ipvs模式 
    - https://blog.51cto.com/blief/1745134
    - ipvs三种模式



# 6.Calico组件原理

参考文章：https://blog.csdn.net/u010771890/article/details/103224004

- 名词解释

~~~shell
endpoint： 接入到calico网络中的网卡成为endpoint
AS: 网络自治系统 通过BGP协议与其他AS网络交换路由信息
ibgp： AS内部的BGP Speaker，与同一个AS内部的ibgp、ebgp交换路由信息
ebgp： AS边界的BGP Speaker， 与同一个AS内部的ibgp、其他AS的ebgp交换路由信息
workloadEndpoint： 虚拟机 容器使用的endpoint
hostEndpoints: 物理机的地址
BGP Speaker： 一般是网络中的物理路由器 calico将node改造成了一个软路由器（通过软路由软件bird) 
~~~

- 组织和架构

~~~shell
#分配和管理IP
#配置容器的veth pair和容器内默认路由
#根据集群网络情况实时更新节点的路由表
#除etcd 容器中还运行着几个组件
#runsv： 是一个minimal的init系统提供的命令 用来管理多个进程 可以看到他运行的进程包括：felix bird bird6 confd libnetwork
#libnetwork plugin： calico提供的docker网络插件 主要提供的是ip管理和网络管理功能 默认情况 当网络中第一个容器出现时 calico会为容器分配一个网段 后续出现在该节点的容器都从这个子网分配IP地址
#BIRD： 一个常用的网络路由组件 支持很多路由协议
#confd： 因为bird的配置文件会根据用户设置的变化而变化 因此需要一种动态的机制来实时维护配置文件并通知bird最新的配置 这就是confd的工作 confd监听etcd的数据 用来更新bird的配置文件 confd的工作目录是 /etc/calico/confd 里面有三个目录 （conf.d需要读取的配置文件 config生成的配置文件最终放的目录 template模板文件） 当发现etcd更新时 就会更新bird.cfg.mesh.template 把新生成的文件放在/etc/calico/confd/config/bird.cfg 
#felix： 负责最终网络相关的配置 1.更新节点的路由表项 2.更新节点上的iptables表项
#etcd: 保存calico网络的元数据
~~~

- 组件的作用

~~~shell
Felix, Calico agent: 跑在每台node上 作用是负责配置路由和ACLs等信息来确保endpoint的连接状态
etcd 负责网络元数据一致性 确保calico网络状态准确性
BGP client（bird） 主要负责吧Felix写入kernel路由信息分发到当前calico网络 确保workload间的联通性
BGP Route Reflector 大规模部署时使用 并且所有节点的mesh模式 通过一个或者多个bgp route reflector 来完成集中式的路由分发
~~~

- calico有基于隧道（ipip）和BGP（路由）两种方式
- 获取calico方法

```shell
curl -O -L  https://github.com/projectcalico/calicoctl/releases/download/v3.9.5/calicoctl
```

- 获取calico部署模板

```shell
curl https://docs.projectcalico.org/v3.9/manifests/calico.yaml -O
```



# 7.BGP协议

- 概念

1. AS 自治系统
2. IGP： RIP OSPF
3. BGP： IBGP EBGP
4. 小规模私有网络使用IGP 大规模私有网络使用IBGP 互联网使用EBGP

·

#8.关于iptables和calico一些笔记

1. 在每个node节点上calico会创建一个tunl0的网桥 （网段）（目前使用calico主要还是以ipip方式）这个节点的所有pod都会被分配这个网段的ip

# 9.BPF





