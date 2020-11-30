# <font color=red size= 10>Operator</font>

## 1.概念篇

### 1.什么是SRE

SRE是用开发软件的方式来进行应用运维的人。他们是工程师、开发者、通晓如何进行软件开发、尤其特定应用域的软件开发。他们做出来的东西，就是包括这一应用的运维领域技能的软件

### 2.什么是Operator

**operator**在基础的k8s资源和控制器之上，加入了相关的知识和配置，让operator能够执行特定软件的常用任务。例如当手动对etcd集群进行伸缩的时候，用户必须执行几个步骤：为新的etcd示例创建DNS名称，加载新的etcd示例，使用etcd管理工具（etcdctl member add）来告知现有集群加入新成员。etcd-operator的用户就只需要简单的把etcd的集群规模字段加一即可。

Operator是跟应用紧密相关的，所以其中最重要的工作就是把应用自身的运维方法编码成为资源和控制逻辑。

**FAQ**

- Operator和statefulset的区别

有的应用需要集群提供“有状态服务”，例如静态ip或者存储。然而有的应用需要更多有状态部署模型的支持，例如故障的告警和应对、备份、重新配置等

- Operator和helm的区别

helm是一个把多个kubernetes资源包装为一个单独软件包的工具，把多个应用集成在一起的概念。



# 2.kubebuilder原理

**本文参考一下文章：**

1. https://www.jianshu.com/p/33ddd445b468
2. https://juejin.cn/post/6844903952241131534

### 1. workqueue example讲解

![](E:\devopsdocs\image\kubebuilder\reconcile原理.png)

1. Reflector通过ListAndWatch方法去监听指定的Object

~~~go
func (r *Reflector) Run(stopCh <-chan struct{}) {
    klog.V(3).Infof("Starting reflector %v (%s) from %s", r.expectedTypeName, r.resyncPeriod, r.name)
    wait.Until(func() {
        if err := r.ListAndWatch(stopCh); err != nil {
            utilruntime.HandleError(err)
        }
    }, r.period, stopCh)
}
~~~

2. Reflector会将监听到的event，包括object的Add、Update、Delete的操作push到DeltaFIFO中这个queue中
3. Informer首先会解析event中的action和object
4. Informer将解析的object更新到localstore
5. 然后Informer会执行Controller在初始化Informer时注册的ResourceEventHandler（这部分可以自己修改）
6. ResourceEventHandler中注册的callback会将对应变化的object的key存储到其初始化的一个workqueue中
7. 最终controller会循环执行reconcile，就是workqueue不停的pop key， 然后去local store中取对应的object，然后进行处理，最终多数情况会再通过client去更新这个object

**kubebuilder还帮助我们做了其他的一些工作：**

1. kubebuilder引入了manager这个概念，一个manager可以管理多个controller，这些controller会共享manager的client
2. 如果manager挂掉，所有的controller也会随之停止工作
3. kubebuilder使用一个```map[GroupVersionKind]informer```来管理这些controller，所以每个controller还是拥有独立的workqueue，deltaFIFO，并且kubebuilder也可以帮我们实现这部分代码
4. 我们主要是写reconcile部分就可以了

