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