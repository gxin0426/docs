### 1.kubernetes部署javaweb（tomcat+mysql）

- mysql deployment

  ```sh
  apiVersion: extensions/v1beta1                  #apiserver的版本
  kind: Deployment                                      #副本控制器deployment，管理pod和RS
  metadata:
    name: mysql                                            #deployment的名称，全局唯一
  spec:
    replicas: 1                                                #Pod副本期待数量
    selector:
      matchLabels:                                         #定义RS的标签
        app: mysql                                          #符合目标的Pod拥有此标签
    strategy:                                                  #定义升级的策略
      type: RollingUpdate                               #滚动升级，逐步替换的策略
    template:                                                #根据此模板创建Pod的副本（实例）
      metadata:
        labels:
          app: mysql                                        #Pod副本的标签，对应RS的Selector
      spec:
        containers:                                          #Pod里容器的定义部分
        - name: mysql                                     #容器的名称
          image: mysql:5.7                               #容器对应的docker镜像
          volumeMounts:                                #容器内挂载点的定义部分
          - name: time-zone                            #容器内挂载点名称
            mountPath: /etc/localtime              #容器内挂载点路径，可以是文件或目录
          - name: mysql-data
            mountPath: /var/lib/mysql               #容器内mysql的数据目录
          - name: mysql-logs
            mountPath: /var/log/mysql              #容器内mysql的日志目录
          ports:
          - containerPort: 3306                         #容器暴露的端口号
          env:                                                   #写入到容器内的环境容量
          - name: MYSQL_ROOT_PASSWORD   #定义了一个mysql的root密码的变量
            value: "123456"
        volumes:                                             #本地需要挂载到容器里的数据卷定义部分
        - name: time-zone                              #数据卷名称，需要与容器内挂载点名称一致
          hostPath:
            path: /etc/localtime                        #挂载到容器里的路径，将localtime文件挂载到容器里，可让容器使用本地的时区
        - name: mysql-data
          hostPath:
            path: /data/mysql/data                   #本地存放mysql数据的目录
        - name: mysql-logs
          hostPath:
            path: /data/mysql/logs                    #本地存入mysql日志的目录
  ```

- mysql service 

```shell
apiVersion: v1
kind: Service     #表示Kubernetes Service
metadata:
  name: mysql   #Service的名称
spec:
  ports:
    - port: 3306   #Service提供服务的端口号
  selector:
    app: mysql    #Service对应的Pod的标签
```

- tomcat deployment

```shell
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: myweb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myweb
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
      - name: myweb
        image: kubeguide/tomcat-app:v1
        volumeMounts:
        - name: time-zone
          mountPath: /etc/localtime
        - name: tomcat-logs
          mountPath: /usr/local/tomcat/logs
        ports:
        - containerPort: 8080
        env:
        - name: MYSQL_SERVICE_HOST
          value: '10.254.144.64'               #此处为mysql服务的Cluster IP
        - name: MYSQL_SERVICE_PORT
          value: '3306'
      volumes:
      - name: time-zone
        hostPath:
          path: /etc/localtime
      - name: tomcat-logs
        hostPath:
          path: /data/tomcat/logs
```

- tomcat service 

```shell
apiVersion: v1
kind: Service
metadata:
  name: myweb
spec:
  type: NodePort
  ports:
    - port: 8080
      nodePort: 30001
  selector:
    app: myweb
```

- 文章链接

<https://blog.51cto.com/andyxu/2309187> 

### 2.k8s网络模式

- 外部访问集群内部的五种网络模式：hostNetwork hostPort LoadBalancer Ingress
- 当pod设置hostNetwork：true时候，Pod中的所有容器就直接暴露在宿主机的网络环境中，这时候，Pod的Podip就是其所在的node的IP。
- 对于同Deployment下的hostNetwork: true启动的Pod，每个node上只能启动一个。也就是说，Host模式的Pod启动副本数不可以多于“目标node”的数量，“目标node”指的是在启动Pod时选定的node，若未选定（没有指定nodeSelector），“目标node”的数量就是集群中全部的可用的node的数量。当副本数大于“目标node”的数量时，多出来的Pod会一直处于Pending状态，因为schedule已经找不到可以调度的node了。 

### 3.从harbor私有仓库拉去镜像

- 用docker登录harbor之后，会在~/.docker/config.json中存入用户名和密码

```shell
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "Z2FveGluMjAyMDpkamRnX3Jzamg="
    }
  }
}
```

- 在对上面的内容进行base64加密

```shell
cat ~/.docker/config.json|base64 -w 0

ewoJImF1dGhzIjogewoJCSJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOiB7CgkJCSJhdXRoIjogIloyRnZlR2x1TWpBeU1EcGthbVJuWDNKemFtZz0iCgkJfQoJfQp9
```

- 创建secret

```yaml
apiVersion: v1
kind: Secret
metadata: 
  name: tomsecret
data: 
  .dockerconfigjson:  ewoJImF1dGhzIjogewoJCSJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOiB7CgkJCSJhdXRoIjogIloyRnZlR2x1TWpBeU1EcGthbVJuWDNKemFtZz0iCgkJfQoJfQp9
type: kubernetes.io/dockerconfigjson
```

- deployment 文件

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
  name: myweb 
spec: 
  replicas: 1
  template: 
    metadata: 
      labels: 
        app: myweb
    spec: 
      containers: 
      - name: myweb
        image: 10.2.20.194:32032/test02/mytomcat:v1
        ports: 
        - containerPort: 8080
      imagePullSecret: 
      - name: tomsecret
```

### 4.rbac

- Role表示一组规则权限，权限只会增加。Role适用于在一个namespace中，如果想要夸namespace需要使用ClusterRole。
- 以下是一个对default空间Pods具有访问权限的样例：

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata: 
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","watch","list"]
```

- ClusterRole可用于
  -  集群级别的资源控制
  - 非资源型endpoints（/healthz）
  - 所有命名空间资源控制

以下是ClusterRole对secret的访问样例

```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata: 
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get","watch","list"]
```

- role利用rolebinding绑定一个user

```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata: 
  name: read-pod
  namespace: default
subjects: 
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleref: 
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

- **RoleBinding 同样可以引用 ClusterRole 来对当前 namespace 内用户、用户组或 ServiceAccount 进行授权，这种操作允许集群管理员在整个集群内定义一些通用的 ClusterRole，然后在不同的 namespace 中使用 RoleBinding 来引用** 
- 例如，以下 RoleBinding 引用了一个 ClusterRole，这个 ClusterRole 具有整个集群内对 secrets 的访问权限；但是其授权用户 `dave` 只能访问 development 空间中的 secrets(因为 RoleBinding 定义在 development 命名空间) 

```yaml
apiVersion: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata: 
  name: read-secrets
  namespace: development
subjects: 
- kind: User
  name: dave
  apiGroup: rbac.authorization.k8s.io
roleRef: 
  kind:ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

- ClusterRoleBinding 

```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: read-secret-global
subjects: 
- kind: Group
  name: manager
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

#### Referring to Resources

kubernetes集群内一些资源一般以其名称字符串表示，这些字符串一般会在API的URL地址中出现；同时某些资源也会包含子资源，例如logs资源就属于pods的子资源 API中URL样例如下

```shell
GET /api/v1/namespaces/{namespace}/pods/{name}/log
```

如果在RBAC授权模型中控制这些子资源的访问权限，可以通过/分隔符来实现，以下是一个定义pods子资源logs访问权限的Role定义样例

```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata: 
  namespace: default
  name: pod-and-pod-logs-reader
rules: 
- apiGroups: [""]
  resources: ["pods","pods/log"]
  verbs: ["get","list"]
```

- 具体的资源引用可以通过 `resourceNames` 来定义，当指定 `get`、`delete`、`update`、`patch` 四个动词时，可以控制对其目标资源的相应动作；以下为限制一个 subject 对名称为 my-configmap 的 configmap 只能具有 `get` 和 `update` 权限的样例

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  namespace: default
  name: configmap-updater
rules:
- apiGroups: [""]
  resources: ["configmap"]
  resourceNames: ["my-configmap"]
  verbs: ["update", "get"]
```

**当设定了 resourceNames 后，verbs 动词不能指定为 list、watch、create 和 deletecollection；因为这个具体的资源名称不在上面四个动词限定的请求 URL 地址中匹配到，最终会因为 URL 地址不匹配导致 Role 无法创建成功**

- **一个ServiceAccount的栗子**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata: 
  name: ingress
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata: 
  name: ingress
subjects: 
  - kind: ServiceAccount
    name: ingress
    namespace: kube-system
roleRef: 
  kind: ClusterRole
  name: cluster-admin
  apiVersion: rbac.authorization.k8s.io
```



### 5.kubernetes中的本地存储方式

- emptyDir hostPath local volume

**1. emptyDir**

emptyDir类型的Volume在Pod分配到Node上时被创建，Kubernetes会在Node上自动分配一个目录，因此无需指定宿主机Node上对应的目录文件。 这个目录的初始内容为空，当Pod从Node上移除时，emptyDir中的数据会被永久删除。 

```shell
apiVersion: v1
kind: Pod
metadata: 
  name: test-pod
spec: 
  containers: 
  - image: busybox
    name: test-emptydir
    command: ["sleep","3600"]
    volumeMounts: 
    - mountPath: /data
      name: data-volume
  volumes: 
  - name: data-volume
    emptyDir: {}
```

**2.hostPath**

hostPath类型则是映射node文件系统中的文件或者目录到pod里。在使用hostPath类型的存储卷时，也可以设置type字段，支持的类型有文件、目录、File、Socket、CharDevice和BlockDevice。 



### 6.构建和管理容器的技巧

- 使用最新的kubernetes模式
- 复用基础镜像以节省时间

在 Kubernetes 集群中创建应用容器时，用户需要构建一个 Docker 基础镜像，然后在此镜像基础上构建部分或全部应用容器。有很多应用共享依赖项、库和配置，因此可以在基础镜像完成对共享部分进行配置，从而实现复用。

Docker Hub 和 Google Container 注册中心有数千个可供下载的基础镜像，这些镜像已经预先完成应用配置，随时可以投入使用，这可以节省大量时间。

![](dockerimage\镜像.png)

- 不要轻易相信任何镜像

尽管使用预先构建的镜像很方便，但要格外小心并确保对其运行特定漏洞扫描。

一些开发人员会从 Docker Hub 中获取一个其他用户创建的基础镜像，然后将这个容器推送到生产环境，而这一切只是因为乍一看这个镜像包含了所需要的包。

这里有很多错误：镜像中的代码版本可能不正确；这些代码可能有漏洞；或者更糟糕的情形是该项目可能已经被故意绑定了恶意软件。

为了缓解上述问题，用户可以通过 Snyder 或 Twistlock 来运行静态分析，然后将其整合到 CI/CD（持续集成和持续交付）管道中，进而扫描所有容器漏洞。

一般来说，一旦在基础镜像中发现漏洞，用户就应该重新构建整个镜像，而不是仅仅修复漏洞。容器应该是不变的，因此，需要引入补丁重新构建和部署镜像。

- 优化基础镜像
- 确保容器只运行一个进程

按照 Google Cloud 的说法，把容器当作虚拟机并同时运行多个进程是一个常见的错误。虽然容器可以实现这种方式，但这样就无法使用 Kubernetes 的自我修复属性。

通常，容器和应用应该同时启动；同样，当应用停止时，容器也应该停止。如果在一个容器中有多个进程，可能会出现应用程序状态混杂的情形，这将导致 Kubernetes 无法确定一个容器是否健康。

- 正确处理Linux信号

容器通过 Linux 信号来控制其内部进程的生命周期。为了将应用的生命周期与容器联系起来，需要确保应用能够正确处理 Linux 信号。

Linux 内核使用了诸如 SIGTERM、SIGKILL 和 SIGINIT 等信号来终止进程。但是，容器内的 Linux 会使用不同的方式来执行这些常见信号，如果执行结果同信号默认结果不符，将会导致错误和中断发生。

创建专门的 init 系统有助于解决此问题，比如专门针对容器的Linux Tini系统。这个工具正确注册了信号处理程序（比如 PID），容器化应用可以正确执行 Linux 信号，从而正常关闭孤立进程和僵尸进程，完成内存回收。

- 充分利用 Docker 的缓存构建机制 

容器镜像由一系列镜像层组成，这些镜像层通过模板或 Dockerfile 中的指令生成。这些层以及构建顺序通常被容器平台缓存。例如，Docker 就有一个可以被不同层复用的构建缓存。这个缓存可以使构建更快，但是要确保当前层的所有父节点都保存了构建缓存，并且这些缓存没有被改变过。简单来讲，需要把不变的层放在前面，而把频繁改变的层放在后面。

例如，假设有一个包含步骤 X、Y 和 Z 的构建文件，对步骤 Z 进行了更改，构建文件可以在缓存中重用步骤 X 和 Y，因为这些层在更改 Z 之前就已经存在，这样可以加速构建过程。但是，如果改变了步骤 X，缓存中的层就不能再被复用。

虽然这是一种方便的行为，可以节省时间，但是必须确保所有镜像层都是最新的，而不是从旧的、过时的缓存构建而成。

- 使用类似helm的包管理器
- 使用标签和语义化版本号
- 安全

在很多情况下，当构建 Docker 镜像时，需要让容器内的应用程序访问敏感数据，例如 API 令牌、私钥和数据库连接字符串等。

将这些秘密信息嵌入到容器中并不是一个安全的解决方案，即使只是保存到一个私有容器镜像中。将未加密的隐私数据作为 Docker 镜像的一部分进行处理会面临无数额外的安全风险，包括网络和镜像注册表的安全性，而 Docker 架构本身也决定了无法对容器中未加密的敏感数据进行优化。

相反，用户可以通过使用 Kubernetes Secrets 对象将隐私信息存储在容器外面，这样更简单、安全。

Kubernetes 提供了一个 Secrets 抽象，允许在 Docker 镜像或 Pod 定义之外存储隐私数据。用户可以通过挂载卷或环境变量的方式把这些信息加载到容器中。更新时，只需更换相关服务的 Pod 并使用新的证书即可。用户也可以通过 Hashicorp Vault 以及Bitnami Sealed Secrets来保存隐私数据。



### 7.kubernetes中的认证机制

1. server account 是由kubernetes API管理的账户。绑定到了特定的namespace，并由api server在自动创建 server account关联一套凭证 存储在secret中，这些凭证同时被挂载到pod中。从而允许kubernetes api之间的调用

### 8.kubernetes中各组件的作用

- API Server： 提供资源对象的唯一操作入口，其他组件必须通过他提供的API来操作资源对象。
- Controller Manager : 集群的内部管理控制中心。主要目的是实现kubernetes集群的故障检测和自动恢复等工作。包括两个核心组件： Node Controller Replication Controller 。其中，其中Node Controller负责计算节点的加入和退出，可以通过Node  Controller实现计算节点的扩容和缩容。Replication  Controller用于Kubernetes资源对象RC的管理，应用的扩容、缩容以及滚动升级都是有Replication  Controller来实现 

​        Controller Manager包含Replication Controller、Node Controller、ResourceQuota Controller、      Namespace Controller、ServiceAccount Controller、Token  Controller、Server Controller以及Endpoint Controller等多个控制器，Controller  Manager是这些Controller的管理者 

- *Scheduler*: 集群中的调度器，负责Pod在集群的中的调度和分配 
- *Kubelet*: 负责本Node节点上的Pod的创建、修改、监控、删除等Pod的全生命周期管理，Kubelet实时向API Server发送所在计算节点（Node）的信息 

节点管理：kubelet可以自动向API Server注册自己，它可以采集所在计算节点的资源信息和使用情况并提交给API Server，通过启动/停止kubelet进程来实现计算节点的扩容、缩容。

Pod管理：kubelet通过API Server监听ETCD目录，同步Pod清单，当发现有新的Pod绑定到所在的节点，则按照Pod清单的要求创建改清单。如果发现本地的Pod被删除，则kubelet通过docker client删除该容器。

健康检查：Pod通过两类探针来检查容器的健康状态。一个是LivenessProbe探针，用于判断容器是否健康，如果LivenessProbe探针探测到容器不健康，则kubelet将删除该容器，并根据容器的重启策略做相应的处理。另一类是ReadnessProbe探针，用于判断容器是否启动完成，且准备接受请求，如果ReadnessProbe探针检测到失败，则Pod的状态被修改。Enpoint   Controller将从Service的Endpoint中删除包含该容器的IP地址的Endpoint条目。kubelet定期调用LivenessProbe探针来诊断容器的健康状况，它目前支持三种探测：HTTP的方式发送GET请求;  TCP方式执行Connect目的端口; Exec的方式，执行一个脚本。

cAdvisor资源监控:  在Kubernetes集群中，应用程序的执行情况可以在不同的级别上检测到，这些级别包含Container，Pod，Service和整个集群。作为Kubernetes集群的一部分，Kubernetes希望提供给用户各个级别的资源使用信息，这将使用户能够更加深入地了解应用的执行情况，并找到可能的瓶颈。Heapster项目为Kubernetes提供了一个基本的监控平台，他是集群级别的监控和事件数据集成器。Heapster通过收集所有节点的资源使用情况，将监控信息实时推送至一个可配置的后端，用于存储和可视化展示

- *Kube-Proxy*: 实现Service的抽象，为一组Pod抽象的服务（Service）提供统一接口并提供负载均衡功能 

### 9.Kubernetes中的ResourceQuota和LimitRange配置资源限

- 两种控制策略的作用范围都是对于某一 namespace，`ResourceQuota` 用来限制 namespace 中所有的 Pod 占用的总的资源 request 和 limit，而 `LimitRange` 是用来设置 namespace 中 Pod 的默认的资源 request 和 limit 值。 
- 配置计算资源分配

```yaml
apiVersion: v1
kind: ResourceQuota
metadata: 
  name: test
  namespace: dev
spec: 
  hard:
    pod: "20"
    requests.cpu: "20"
    requests.memory: 100Gi
    limits.cpu: "40"
    limits.memory: 200Gi
    
```

- 配置对象数量限制

```yaml
apiVersion: v1
kind: ResourceQuota
metadata: 
  name: test2
  namespace: dev
spec: 
  hard: 
    configmaps: "20"
    persistentvolumesclaims: "4"
    replicationsets: "8"
    secrets: "3"
    services: "3"
    services.loadbalancers: "2"
```

- limitrange 限制pod的没存和cpu

```yaml
apiVersion: v1
kind: LimitRange
metadata: 
  name: test3
  namespace: dev
spec: 
  limits: 
  - default:
      memory: 50Gi
      cpu: "8"
    defaultRequest:
      memory: 1Gi
      cpu: "4"
    type: Container
    
```

### 10.kubernetes架构

![](image\k8s-arch.jpg)

1.master节点工作原理

![](image\master.png)

```shell

```

1. Kubecfg将特定的请求，比如创建Pod，发送给Kubernetes Client。
2. Kubernetes Client将请求发送给API server。
3. API Server根据请求的类型，比如创建Pod时storage类型是pods，然后依此选择何种REST Storage API对请求作出处理。
4. REST Storage API对的请求作相应的处理。
5. 将处理的结果存入高可用键值存储系统Etcd中。
6. 在API Server响应Kubecfg的请求后，Scheduler会根据Kubernetes Client获取集群中运行Pod及Minion/Node信息。
7. 依据从Kubernetes Client获取的信息，Scheduler将未分发的Pod分发到可用的Minion/Node节点上。

**master**

- **api server**
- **controller manager**
- **scheduler**

**node**

- **kubelet**

- **proxy**

  

### 11.kubernetes中hostport port targetport nodeport 区别

- **hostport：** 这是一种直接定义Pod网络的方式。hostPort是直接将容器的端口与所调度的节点上的端口路由，这样用户就可以通过宿主机的IP加上来访问Pod了
- **port：** port是在Service IP中使用的，使用Service IP +Port就可以访问到服务
- **nodePort：** nodePort在kubenretes里是一个广泛应用的服务暴露方式。Kubernetes中的service默认情况下都是使用的ClusterIP这种类型，这样的service会产生一个ClusterIP，这个IP只能在集群内部访问，要想让外部能够直接访问service，需要将service type修改为 nodePort
- **targetPort：** targetPort 说的是Pod内的应用暴露的服务端口，Service IP+Port的访问会被代理到这个Target Port
- **containerPort:** 在容器上，用于和pod绑定

1、NodePort和port

前者是将服务暴露给外部用户使用并在node上、后者则是为内部组件相互通信提供服务的，是在service上的端口。

2、targetPort

targetPort是pod上的端口，用来将pod内的container与外部进行通信的端口

 3、port、NodePort、ContainerPort和targetPort在哪儿？

 port在service上，负责处理对内的通信，clusterIP:port

NodePort在node上，负责对外通信，NodeIP:NodePort

 ContainerPort在容器上，用于被pod绑定

 targetPort在pod上、负责与kube-proxy代理的port和Nodeport数据进行通信

### 12.k8s集群和host同步时间

```yaml
    #映射主机/etc/localtime
    spec:
      containers:
      - name: myweb
        image: harbor/tomcat:8.5-jre8
        volumeMounts:
        - name: host-time
          mountPath: /etc/localtime
        ports:
        - containerPort: 80
      volumes:
      - name: host-time
        hostPath:
          path: /etc/localtime
```

### 13. kube api

```yaml
kubectl api-versions
```

### 14. 镜像加速

```shell
#如果我们在docker官方仓库拉取的镜像是以下形式：
docker pull xxx/yyy:zz
#那么使用Azure中国镜像，应该是这样拉取：
docker pull dockerhub.azk8s.cn/xxx/yyy:zz
#mysql5.7
docker pull dockerhub.azk8s.cn/library/mysql:5.7


#如果我们拉取的google镜像是以下形式：
docker pull gcr.io/xxx/yyy:zzz
#那么使用Azure中国镜像，应该是这样拉取：
docker pull gcr.mirrors.ustc.edu.cn/xxx/yyy:zzz
#以拉取gcr.io/kubernetes-helm/tiller:v2.9.1为例，如下：
docker pull gcr.azk8s.cn/kubernetes-helm/tiller:v2.9.1

#如果我们拉取的kubernetes google镜像是以下形式：
docker pull k8s.gcr.io/xxx:yyy
#相当于docker pull gcr.io/google-containers/xxx:yyy
#那么使用Azure中国镜像，应该是这样拉取：
docker pull gcr.azk8s.cn/google-containers/xxx:yyy
#以拉取k8s.gcr.io/addon-resizer:1.8.3为例，如下：
docker pull gcr.azk8s.cn/google-containers/addon-resizer:1.8.3

#如果我们拉取的quay.io镜像是以下形式：
docker pull quay.io/xxx/yyy:zzz
#那么使用Azure中国镜像，应该是这样拉取：
docker pull quay.azk8s.cn/xxx/yyy:zzz
#以拉取quay.io/coreos/kube-state-metrics:v1.5.0为例，如下：
docker pull quay.azk8s.cn/coreos/kube-state-metrics:v1.5.0

```

### 15.init Container

- 他们仅运行一次，成功后就会退出。
- 每个容器必须在成功执行完成后，系统才能继续执行下一个容器。
- 如果Init Container运行失败，kubernetes 将会重复重启Pod,直到Init Container 成功运行，但是如果  Pod的重启策略（restartPolicy）设置为Never,则Pod不会重启。
- Init Container支持普通应用Container的所有参数，包括资源限制，挂载卷，安全设置等。但是Init Container  在资源的申请和限制上略有不同，同时，由于Init Container必须在Pod ready之前完成并退出，所以它不支持 readiness  探针。

- https://blog.51cto.com/tryingstuff/2130997?source=dra

### 16.node 调度和隔离 亲和性 

- node selector

```yaml
#得到node的名字
kubectl get nodes 
#给node打标签
kubectl label nodes <node-name> <label-key>=<label-value>
#删除标签
kubectl label nodes <node-name> <label-key>-
#查看node的label
kubectl get nodes --show-labels 

#给pod设置 nodeselector

apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  labels: 
    env: test
spec:
  containers:
  - name: test-nginx
    image: nginx
  nodeSelector:
    nodetype: prom 
```

- node taint

```yaml
#给node设置为taint
kubectl taint nodes <node-name> nodetype=clustermaster:PreferNoSchedule
kubectl taint nodes <node-name> nodetype=clustermaster:NoSchedule
kubectl taint nodes <node-name> nodetype=clustermaster:NoExecute

kubectl taint nodes <node-name> nodetype:PreferNoSchedule-
kubectl taint nodes <node-name> nodetype:NoSchedule-
kubectl taint nodes <node-name> nodetype:NoExecute-

#查看node上的taint
kubectl describe nodes node1

#可以为pod设置toleration 使pod可以调度到taint node上

#只要在 pod 的 spec 中设置 tolerations 字段即可，可以有多个 key
#如果将 toleration 应用于 pod 上，则表示这些 pod 可以（但不要求）被调度到具有相应 taint 的节点上
tolerations:
- key: "nodetype"
  operator: "Equal"
  value: "clustermaster"
  effect: "PreferNoSchedule"
  tolerationSeconds: 6000
  


```

### 17.利用kubectl提高工作效率

- **命令完成**

命令完成是提高你的kubectl生产力的最有用但经常被忽视的技巧之一

命令完成的工作原理：通常，命令完成是一个shell功能，它通过completion script(完成脚本)的方式工作。完成脚本是一个shell脚本，用于定义特定命令的完成行为。获取完成脚本可以完成相应的命令。

Kubectl可以使用以下命令自动生成并打印出Bash和Zsh的完成脚本

~~~shell
kubectl completion bash
or
kubectl completion zsh
~~~

Bash的完成脚本取决于bash-completion项目，因此您必须先安装它

~~~shell
yum install -y bash-completion

#您可以使用以下命令测试是否正确安装了bash-completion
type _init_completion

#如果这输出shell函数的代码，则已正确安装bash-completion。如果该命令输出not found错误，则必须将以下行添加到您的~/.bashrc文件中
#是否必须将此行添加到您的~/.bashrc文件中，取决于您用于安装bash-completion的包管理器。对于APT来说，这是必要的，对于yum，则无需。
source /usr/share/bash-completion/bash_completion

#安装bash-completion后，您必须进行设置，以便在所有shell会话中获取kubectl 完成脚本。一种方法是将以下行添加到您的~/.bashrc文件中
source <(kubectl completion bash)

#另一种可能性是将kubectl完成脚本添加到/etc/bash_completion.d目录中（如果它不存在则创建它）：
kubectl completion bash >/etc/bash_completion.d/kubectl

#使用自定义列输出格式
kubectl get pods -o custom-columns='NAME:metadata.name'
~~~

- ### JSONPath表达式

选择资源字段的表达式基于JSONPath。JSONPath是一种从JSON文档中提取数据的语言（类似于XPath for XML）。选择单个字段只是JSONPath的最基本用法。它有很多功能，如列表选择器，过滤器等。但是，kubectl explain仅支持JSONPath功能的一部分。以下通过示例用法总结了这些支持的功能：

### 18.kubernetes 中HPA的应用

~~~yaml
 1 apiVersion: v1
 2 kind: Service
 3 metadata:
 4   name: svc-hpa
 5   namespace: default
 6 spec:
 7   selector:
 8     app: myapp
 9   type: NodePort  ##注意这里是NodePort，下面压力测试要用到。
10   ports:
11   - name: http
12     port: 80
13 ---
14 apiVersion: apps/v1
15 kind: Deployment
16 metadata:
17   name: myapp
18   namespace: default
19 spec:
20   replicas: 1
21   selector:
22     matchLabels:
23       app: myapp
24   template:
25     metadata:
26       name: myapp-demo
27       namespace: default
28       labels:
29         app: myapp
30     spec:
31       containers:
32       - name: myapp
33         image: ikubernetes/myapp:v1
34         imagePullPolicy: IfNotPresent
35         ports:
36         - name: http
37           containerPort: 80
38         resources:
39           requests:
40             cpu: 50m
41             memory: 50Mi
42           limits:
43             cpu: 50m
44             memory: 50Mi
~~~

~~~yaml
 1 apiVersion: autoscaling/v2beta1
 2 kind: HorizontalPodAutoscaler
 3 metadata:
 4   name: myapp-hpa-v2
 5   namespace: default
 6 spec:
 7   minReplicas: 1         ##至少1个副本
 8   maxReplicas: 8         ##最多8个副本
 9   scaleTargetRef:
10     apiVersion: apps/v1
11     kind: Deployment
12     name: myapp
13   metrics:
14   - type: Resource
15     resource:
16       name: cpu
17       targetAverageUtilization: 50  ##注意此时是根据使用率，也可以根据使用量：targetAverageValue
18   - type: Resource
19     resource:
20       name: memory
21       targetAverageUtilization: 50  ##注意此时是根据使用率，也可以根据使用量：targetAverageValue

#使用ab工具模拟压力测试
ab -c 1000 -n 5000000 http://192.168.1.103:31727/index.html
~~~

