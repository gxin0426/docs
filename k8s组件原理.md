# 1.API SERVER

- **介绍**

API Server提供了k8s各类资源对象的增删改查及watch等http rest接口 是整个系统的数据总线和数据中心

1. 提供了集群管理的rest api接口（认证授权 数据校验 集群状态变更）
2. 提供其他模块之间的数据交互和通信的枢纽（api server才能直接操作etcd）
3. 是资源配额控制的入口
4. 拥有完备的集群安全机制

# 2. Controller Manager

- controller manager作为集群内部的管理控制中心，负责集群内的node、pod副本、服务端点endpoint、namespace、serviceaccount、resourcequota的管理，当某个node意外宕机时，controller manager会及时发现并执行自动化修复流程。

![](image\controller-manager.png)

1. replication controller
2. node controller
3. resourcequota controller & limitranger
4. namespace controller
5. endpoint controller
6. service controller

# 3. Scheduler

- Scheduler负责pod调度。在整个系统中起“承上启下”作用，承上：负责接收Controller Manager创建的新的pod，为其选择一个合适的node。启下：node上的kubelet接管pod的生命周期。

# 4. kubelet

- 在kubernetes集群中，每个node节点上都会启动kubelet进程，用来处理master节点下到本节点的任务，管理pod和其中的容器。kubelet会在api server上注册节点信息，定期向master汇报节点资源使用情况，并通过cadvisor监控容器和节点资源。

# 5. Kube-Proxy

1. Proxy是为了解决外部网络能够访问跨机器集群中容器提供的应用服务而设计的，运行在每个Minion/Node上。Proxy提供TCP/UDP   sockets的proxy，每创建一种Service，Proxy主要从etcd获取Services和Endpoints的配置信息（也可以从file获取），然后根据配置信息在Minion/Node上启动一个Proxy的进程并监听相应的服务端口，当外部请求发生时，Proxy会根据Load  Balancer将请求分发到后端正确的容器处理。
2. Proxy不但解决了同一主宿机相同服务端口冲突的问题，还提供了Service转发服务端口对外提供服务的能力，Proxy后端使用了随机、轮循负载均衡算法。









