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
3. https://cloud.tencent.com/developer/article/1557568
4. https://juejin.cn/post/6844904083057295374

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

###2.workqueue example 另一篇文章

![](image\kubebuilder\workqueueexample.png)

下半部分是需要自己写的

**client-go中包含以下部分：**

- **Reflector：**通过调用ListWatch接口与ApiServer通信，监听特定资源，并把资源的更新动态（event）存入DeltaFIFO队列
- **Informer：**从Delta中拿出对象（key）， 完成此操作的函数是processLoop （informer在client-go是controller）
- **Indexer：** 本地缓存 提供索引 方便快速查找

**custom controller包含以下部分**

- **Informer reference：** informer 对象引用
- **Indexer reference：** Indexer 对象引用
- **Resource Event Handlers：** 被Informer调用的回调函数，这些函数的作用通常是获取对象的key，并把key放入workqueue中，以进一步做处理
- **Workqueue：**工作队列，用于将对象的交付与处理分离，编写Resource event handler Functions 以提取传递的对象key并将其添加到工作队列。此处可以过滤掉我们不关系的信息
- **Process Item：** 用于处理workqueue中的对象， 可以有一个或多个其他函数一起处理；这些函数通常使用Indexer reference 或Listing wrapper 来检索与该键对应的对象。==**这里就是我们要自定义的业务逻辑**==（reconcile）

**workqueue example主要是用于监听pod创建， 删除信息，并将信息打印**

~~~go
package main

import (
	"flag"
	"fmt"
	v1 "k8s.io/api/core/v1"
	"k8s.io/apimachinery/pkg/fields"
	"k8s.io/apimachinery/pkg/util/runtime"
	"k8s.io/apimachinery/pkg/util/wait"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/cache"
	"k8s.io/client-go/tools/clientcmd"
	"k8s.io/client-go/util/workqueue"
	"k8s.io/klog"
	"time"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)


/*
此示例演示如何编写 跟踪监视资源状态的控制器

它演示了如何:
	1.将工作队列（workqueue）与缓存（cache）合并到完整的控制器
	2.启动时同步控制器
这个示例基于： https://git.k8s.io/community/contributors/devel/sig-api-machinery/controllers.md

一个kubernetes controller 是一个主动调谐过程（process），controller观测着系统中某个资源的真实状态，同时也观测着系统中某个资源的期望状态，
然后controller发出指令 试图使系统某个资源的真实状态 更接近期望状态

 */




// Controller demonstrates how to implement a controller with client-go.
//自己实现的controller （用户侧）


//1.首先我们定义一个Controller结构体
type Controller struct {
	// indexer的引用
	indexer  cache.Indexer
	//workqueue
	queue    workqueue.RateLimitingInterface
	//informer的引用
	informer cache.Controller
}

// NewController creates a new Controller.
func NewController(queue workqueue.RateLimitingInterface, indexer cache.Indexer, informer cache.Controller) *Controller {
	return &Controller{
		informer: informer,
		indexer:  indexer,
		queue:    queue,
	}
}

//定义controller工作流
// Run begins watching and syncing.
func (c *Controller) Run(threadiness int, stopCh chan struct{}) {
	defer runtime.HandleCrash()

	// Let the workers stop when we are done
	defer c.queue.ShutDown()
	klog.Info("Starting Pod controller")
	//启动informer
	go c.informer.Run(stopCh)

	// Wait for all involved caches to be synced, before processing items from the queue is started
	if !cache.WaitForCacheSync(stopCh, c.informer.HasSynced) {
		runtime.HandleError(fmt.Errorf("Timed out waiting for caches to sync"))
		return
	}
	//启动多个worker 处理workqueue中的对象
	for i := 0; i < threadiness; i++ {
		go wait.Until(c.runWorker, time.Second, stopCh)
	}

	<-stopCh
	klog.Info("Stopping Pod controller")
}


//从workqueue中取出对象，并打印消息
func (c *Controller) processNextItem() bool {
	// Wait until there is a new item in the working queue
	key, quit := c.queue.Get()
	if quit {
		return false
	}
	// Tell the queue that we are done with processing this key. This unblocks the key for other workers
	// This allows safe parallel processing because two pods with the same key are never processed in
	// parallel.
	defer c.queue.Done(key)


	// Invoke the method containing the business logic
	//包含了处理逻辑 好像reconcile就在这个里面？
	// 将key对应的 object 的信息进行打印
	err := c.syncToStdout(key.(string))
	// Handle the error if something went wrong during the execution of the business logic
	c.handleErr(err, key)
	return true
}

// syncToStdout is the business logic of the controller. In this controller it simply prints
// information about the pod to stdout. In case an error happened, it has to simply return the error.
// The retry logic should not be part of the business logic.
func (c *Controller) syncToStdout(key string) error {
	obj, exists, err := c.indexer.GetByKey(key)
	if err != nil {
		klog.Errorf("Fetching object with key %s from store failed with %v", key, err)
		return err
	}

	if !exists {
		// Below we will warm up our cache with a Pod, so that we will see a delete for one pod
		fmt.Printf("Pod %s does not exist anymore\n", key)
	} else {
		// Note that you also have to check the uid if you have a local controlled resource, which
		// is dependent on the actual instance, to detect that a Pod was recreated with the same name
		fmt.Printf("Sync/Add/Update for Pod %s\n", obj.(*v1.Pod).GetName())
	}
	return nil
}

// handleErr checks if an error happened and makes sure we will retry later.
func (c *Controller) handleErr(err error, key interface{}) {
	if err == nil {
		// Forget about the #AddRateLimited history of the key on every successful synchronization.
		// This ensures that future processing of updates for this key is not delayed because of
		// an outdated error history.
		c.queue.Forget(key)
		return
	}

	// This controller retries 5 times if something goes wrong. After that, it stops trying.
	if c.queue.NumRequeues(key) < 5 {
		klog.Infof("Error syncing pod %v: %v", key, err)

		// Re-enqueue the key rate limited. Based on the rate limiter on the
		// queue and the re-enqueue history, the key will be processed later again.
		c.queue.AddRateLimited(key)
		return
	}

	c.queue.Forget(key)
	// Report to an external entity that, even after several retries, we could not successfully process this key
	runtime.HandleError(err)
	klog.Infof("Dropping pod %q out of the queue: %v", key, err)
}


/***********具体处理workqueue中对象的流程**************/

func (c *Controller) runWorker() {
	//启动无线循环 接受并处消息
	for c.processNextItem() {
	}
}

func main() {
	var kubeconfig string
	var master string

	flag.StringVar(&kubeconfig, "kubeconfig", "", "absolute path to the kubeconfig file")
	flag.StringVar(&master, "master", "", "master url")
	flag.Parse()

	// creates the connection
	config, err := clientcmd.BuildConfigFromFlags("", "config")
	if err != nil {
		klog.Fatal(err)
	}

	// creates the clientset
	clientset, err := kubernetes.NewForConfig(config)
	if err != nil {
		klog.Fatal(err)
	}
	// 指定ListerWatcher 在default ns下监听 pod资源
	// create the pod watcher
	podListWatcher := cache.NewListWatchFromClient(clientset.CoreV1().RESTClient(), "pods", v1.NamespaceDefault, fields.Everything())

	// create the workqueue
	queue := workqueue.NewRateLimitingQueue(workqueue.DefaultControllerRateLimiter())

	// Bind the workqueue to a cache with the help of an informer. This way we make sure that
	// whenever the cache is updated, the pod key is added to the workqueue.
	// Note that when we finally process the item from the workqueue, we might see a newer version
	// of the Pod than the version which was responsible for triggering the update.

	//client-go tool/cache/controller.go文件 中的controller 也就是informer
	indexer, informer := cache.NewIndexerInformer(podListWatcher, &v1.Pod{}, 0, cache.ResourceEventHandlerFuncs{
		//当有pod创建时，根据delta FIFO 弹出的object生成对应的key，并加入到workqueue中。此处可以根据Object的一些属性进行过滤
		AddFunc: func(obj interface{}) {
			key, err := cache.MetaNamespaceKeyFunc(obj)
			if err == nil {
				queue.Add(key)
			}
		},
		UpdateFunc: func(old interface{}, new interface{}) {
			key, err := cache.MetaNamespaceKeyFunc(new)
			if err == nil {
				queue.Add(key)
			}
		},
		//DeletionHandlingMetaNamespaceKeyFunc 会在生成key之前检查，因为资源删除后有可能会进行重建等操作
		//监听时错过了删除信息，从而导致该条记录是陈旧的
		DeleteFunc: func(obj interface{}) {
			// IndexerInformer uses a delta queue, therefore for deletes we have to use this
			// key function.
			key, err := cache.DeletionHandlingMetaNamespaceKeyFunc(obj)
			if err == nil {
				queue.Add(key)
			}
		},
	}, cache.Indexers{})

	controller := NewController(queue, indexer, informer)

	// We can now warm up the cache for initial synchronization.
	// Let's suppose that we knew about a pod "mypod" on our last run, therefore add it to the cache.
	// If this pod is not there anymore, the controller will be notified about the removal after the
	// cache has synchronized.
	indexer.Add(&v1.Pod{
		ObjectMeta: metav1.ObjectMeta{
			Name:      "etcd-sample-0",
			Namespace: v1.NamespaceDefault,
		},
	})

	// Now let's start the controller
	stop := make(chan struct{})
	defer close(stop)
	go controller.Run(1, stop)

	// Wait forever
	select {}
}

~~~









###3.核心概念

#### 1.GVK and GVR

GVK = GroupVersionKind

GVR = GroupVersionResource

API Group and version

API Group是相关API功能的集合，每个Group拥有一个或多个Version 用于接口的演进

每个GV包含多个API类型，成为Kinds  Reource是kind的对象标识

apiVersion : GV

kind : K

 根据 GVK K8s 就能找到你到底要创建什么类型的资源，根据你定义的 Spec 创建好资源之后就成为了 Resource，也就是 GVR。GVK/GVR 就是 K8s 资源的坐标，是我们创建/删除/修改/读取资源的基础。 

#### 2.scheme

每一组Controllers都需要一个Scheme，提供kind与对应Go types 的映射，也就是说给定Go type就知道他的GVK，给定GVK就知道他的Go type

#### 3.manager

负责运行所有的COntrollers

初始化共享caches 包括listAndWatch功能

初始化clients用于与Api server通信

#### 4.Cache

负责在Controller进程里面根据Scheme同步Api server中所有该Controllers关心GVK的GVR，其核心是GVK -> Informer 的映射，Informer会负责监听对应的GVK的GVR的创建、删除、更新，以触发Controller的Reconcile逻辑

#### 5.Finalizer

在一般情况下，如果资源被删除后，我们虽然能被触发删除事件，但是这个时候从Cache里面无法读取任何被删除对象的信息，这样一来，导致很多垃圾清理工作因为信息不足无法进行。在kubernetes中，只要ObjMeta里面的finalizer不为空，对该对象的delete操作就会转变为update操作，具体说就是 update deletionTimestap字段，其意义就是告诉kubernetes的GC “在deletionTImestap这个时刻以后， 只要finalizer为空，就立马删除该对象”

### 4.几个重要的问题

1. 如何同步自定义资源以及 K8s build-in 资源？（reconcile是如何工作的）
2. Controller 的 Reconcile 方法是如何被触发的？（reconcile的触发条件）
3. Cache 的工作原理是什么？（Informer机制的原理）

源码开始：



# 3.2017谷歌工程师分享

- **spec and status**

大部分的kubernetes API都有Spec和Status两个field。Spec是让用户写入期望的状态，系统可以通过Spec读出用户的期望，Status是系统写入观察到的状态，用户可以从中读出系统当前是什么状态

![](image\kubebuilder\specandStatus.png)

- **Reconcile**

这两个field对于Reconciliation Loop非常重要， Reconcile中文意思是调谐，简单说就是不断使系统当前的状态向用户期望的状态移动。如图，用户期望的Replica是3个，Controller通过watch发现期望的状态是3个，但实际观察到的Replica是2个，所以他会Create一个新的pod，然后controller会继续watch这些pod，当他发现Create完成了，就会更新Status到3个，使Status和Spec达到一致的状态。

![](image\kubebuilder\reconcile2.png)

