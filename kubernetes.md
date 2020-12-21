

### 1.kubernetes部署（tomcat+mysql）

- mysql deployment

  ```sh
  apiVersion: app/v1                  #apiserver的版本
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

```yaml
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

#### 1.理解kubelet认证和授权

- 调用过程kubectl  -- > apiserver  -- > kubelet
- kubelet通过port（默认10250）对外暴露服务 服务需要TLS认证 同时也可以通过readOnlyPort端口（默认10255，0表示关闭）对外暴露只读服务 这个是不需要认证的apiserver通过--kubelet-https 参数指定调用哪个服务 true为前者 

认证过程

- 配置认证方式

1. tls认证

   ~~~yaml
   authentication:
     anonymous:
       enabled: false
     webhook: 
       enabled: false
     x509:
       clientCAFile: xxxx 
   ~~~

2. 允许anonymous 这是可以不配置客户端证书

   ~~~yaml
   authentication:
     anonymous: 
       enabled: true
   ~~~

3. webhook 这时可以不配置客户端证书

   ~~~yaml
   authentication:
     webhook:
       enabled: true
   ~~~

   这时kubelet通过bearer tokens找apiserver认证 如果存在对应的serviceaccount 则认证通过 如果2开启 则忽略x509和webhook认证；否则 如果1 和 3 同时开启 则按照1.3的顺序依次认证 任何一个认证通过则返回通过

- 证书配置

  kubelet对外暴露https服务 必须设置服务端证书 如果通过x509证书认证客户端 那么还需要配置客户端证书 下面说明证书配置的三种方法：

  1. 手工指定证书

     - 假设ca的证书和key：ca.pem ca-key.pem

     - 用上述的ca生成kubelet服务端证书和key：kubelet-server.pem kubelet-server-key.pem

     - 用上述ca生成apiserver使用的客户端证书和key： kubelet-client.pem kubelet-client-key.pem 证书CN为kubelet-client

     - 修改kubelet的配置文件

       ~~~yaml
       tlsCerFile: kubelet-server.pem
       tlsPrivateKeyFile: kubelet-server-key.pem
       authentication:
         x509:
           clientCAFile: ca.pem
       ~~~

     - 修改apiserver参数

       ~~~yaml
       --kubelet-certificate-authority=ca.pem --kubelet-client-certificate=kubelet-client.pem --kubelet-client-key=kubelet-client-key.pem
       ~~~

     - 授权kubelet-client用户：

       ~~~shell
       kubectl create clusterrolebinding kubelet-admin --clusterrole=system:kubelet-api-admin --user=kubelet-client
       ~~~

  2. 自签名证书和key

     实际上是上述过程的特化 不指定tlsCerFile和tlsPriviteKeyFile时，kubelet会自动生成服务端证书保存在--cert-dir指定目录下 文件名分别为kubelet.crt kubelet.key 这个证书是自签名的 所以apiserver不需要指定--kubelet-certificate-authority 其他配置是一样的

  3. 通过TLS bootstrap机制

     建立完整的TLS加密通信，需要有一个CA认证机构，会向客户端下发根证书，服务器端证书以及签名私钥给客户端。ca.pem ca-key.pem ca.csr组成了一个自签名的CA机构
  
    ca.pem： ca根证书文件
  
    ca-key.pem： 服务端私钥 用于对客户端请求的解密和签名
  
    ca.csr： 证书签名请求 用于交叉签名或重新签名
  
    token.csv： 改文件为一个用户的描述文件 基本格式： Token，用户名，UID,用户组；这个文件在apiserver启动时被apiserver加载 然后就相当于在集群中创建一个这个用户 接下来就可以用rbac对他授权
  
    bootstrap.kubeconfig： 该文件内置了token.csv中用户的Token，以及apiserver CA证书 kubelet首次启动时会加载该文件 使用 apiserver CA证书建立与apiserver的TLS通讯 使用其中的用户Token作为身份标识向apiserver发起CSR请求。
  
    kubelet-client-current.pem：这是个软连接文件 当kubelet配置了--feature-gates=RotateKubeletClientCertificate=true 选项后，会在证书总有效期的70%-90%的时间内发起续约请求，请求被批准后会生成kubelet-client-timestamp.pem ;kubelet-client-current.pem 文件则始终软连接到最新的真实证书文件，除首次启动外，kubelet一直会使用这个证书通apiserver通讯。
  
    kubelet-server-current.pem： 同样是一个软连接文件，当 kubelet 配置了 `--feature-gates=RotateKubeletServerCertificate=true` 选项后，会在证书总有效期的 `70%~90%` 的时间内发起续期请求，请求被批准后会生成一个 `kubelet-server-时间戳.pem`；`kubelet-server-current.pem` 文件则始终软连接到最新的真实证书文件，该文件将会一直被用于 kubelet 10250 api 端口鉴权
  
    参考文献：  https://www.jianshu.com/p/bb973ab1029b      
  
    

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

- limitrange 限制pod的内存和cpu

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

### 15.init Container 健康检测 服务依赖  

- 他们仅运行一次，成功后就会退出。
- 每个容器必须在成功执行完成后，系统才能继续执行下一个容器。
- 如果Init Container运行失败，kubernetes 将会重复重启Pod,直到Init Container 成功运行，但是如果  Pod的重启策略（restartPolicy）设置为Never,则Pod不会重启。
- Init Container支持普通应用Container的所有参数，包括资源限制，挂载卷，安全设置等。但是Init Container  在资源的申请和限制上略有不同，同时，由于Init Container必须在Pod ready之前完成并退出，所以它不支持 readiness  探针。
- https://blog.51cto.com/tryingstuff/2130997?source=dra
- https://blog.csdn.net/qq_34463875/article/details/78113382

- 应用方式

1. 处于安全考虑 可以将自定义的代码和工具使用Initcontainer运行 而不必添加到应用的镜像中
2. 用不同的linux命名空间 可以使他们来自应用容器的不同文件系统权限 因此 initcontainer可以获得应用程序容器无法访问的secrets

- 栗子

~~~yaml
#定义一个nginx 在nginx容器启动前更改默认起始页面内容
apiVersion: v1
kind: Pod
metadata: 
  name: myapp-pod
  labels:
    app: myapp
spec:
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh','-c','echo "this is my happy">/local/nginx/html/index.html']
    volumeMounts:
    - name: index-dir
      mountPath: "/local/nginx/html/"
  containers: 
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: index-dir
      mounthPath: /usr/share/nginx/html
  volumes: 
  - name: index-dir
    emptyDir: {}
~~~





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
#去除taint
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





### 19 下载k8s二进制文件并安装集群

~~~shell
1.在https://github.com/kubernetes/kubernetes/releases/tag/v1.14.6 下载文件
2.解压并运行  ./kubernetes/cluster/get-kube-binaries.sh 获取二进制文件 （需要翻墙）
~~~

###20.Kubernetes 针对资源紧缺处理方式的配置

https://www.kubernetes.org.cn/1150.html

1. nodefs : 指node自身的存储 存储daemon的运行日志等 ， 一般指root分区
2. imagefs： 指docker daemon用于存储image和容器可写层（wirite layer）的磁盘

### 21.sealos安装kubernetes集群

~~~shell
#安装docker过程
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
$sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
$ sudo yum install docker-ce-18.09.1 docker-ce-cli-18.09.1 containerd.io -y


#安装sealos
wget https://github.com/fanux/sealos/releases/download/v2.0.7/sealos && chmod +x sealos && mv sealos /usr/bin
#安装
sealos init \
    --master 192.168.1.155 \
    --master 192.168.1.156 \
    --master 192.168.1.157 \
    --node 192.168.1.158 \
    --user root \
    --passwd 123456 \
    --pkg-url /root/kube1.16.0.tar.gz  \
    --version v1.16.0
#清理
sealos clean \
    --master 192.168.1.155 \
    --master 192.168.1.156 \
    --master 192.168.1.157 \
    --node 192.168.1.158 \
    --user root \
    --passwd 123456
#添加节点
sealos join \
    --master 192.168.1.155 \
    --master 192.168.1.156 \
    --master 192.168.1.157 \
    --node 192.168.1.164 \
    --vip 10.103.97.2 \
    --user root \
    --passwd 123456 \
    --pkg-url /root/kube1.16.0.tar.gz  
#教程
https://www.kubernetes.org.cn/5904.html
https://www.jianshu.com/p/0ce1b53478ce
~~~

### 22.kubernetes nfs 外部存储

#### 安装nfs

- NFS是network file system的缩写，即网络文件系统。功能是让客户端通过网络访问不同宿主机磁盘上的数据，主要用在类Unix系统上实现文件共享和存储的一种方法。

- 安装环境
  - server 192.168.1.156
  - client 192.168.1.164
- 服务器端安装nfs

~~~shell
$ yum install -y nfs-utils #无需安装rpcbind rpcbind是他的一个依赖
#开启nfs 和 rpcbind
$ systemctl enable rpcbind
$ systemctl enable nfs
$ systemctl start rpcbind 
$ systemctl start rpcbind
#当开启防火墙时 需打开rpcbind和nfs服务
$ firewall-cmd --zone=public --permanent --add-service={rpc-bind,mounted,nfs}
success
$ firewall-cmd --reload
success

#配置共享目录
$ mkdir /data
$ chmod 777 /data
$ vim /etc/exports
#添加配置
/data/ 192.168.1.0/24(rw,sync,no_root_squash,no_all_squash)

# /data : 共享目录位置
# 192.168.1.0/24 : 客户端ip范围
# rw : 权限设置，可读可写
# sync : 同步共享目录
# no_root_squash : 可以使用root授权
# no_all_squash : 可以使用普通用户授权

#重启nfs
$ systemctl restart nfs

#查看本地共享目录
$ showmount -e localhost
Export list for localhost:
/data 192.168.1.0/24
~~~

- 配置client

~~~shell
#安装nfs
$ yum install nfs-utils -y
#设置开机启动并start
$ systemctl enable rpcbind
$ systemctl restart rpcbind
#client不需要打开防火墙 client是请求方 网络连接到server即可 client不需要开启nfs 因为不共享目录
# 查看server共享目录
$ showmount -e 192.168.1.156
Export list for 192.168.1.156:
/data 192.168.1.0/24
#创建共享目录
$ mkdir /data
$ chmod 777 /data
#在客户端执行
$ mount -t nfs 192.168.1.156:/data /data

$mount 
192.168.1.156:/data on /data type nfs4 (rw,relatime,vers=4.1,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=192.168.1.164,local_lock=none,addr=192.168.1.156)

#client自动挂载
$ vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Thu May 25 13:11:52 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=414ee961-c1cb-4715-b321-241dbe2e9a32 /boot                   xfs     defaults        0 0
/dev/mapper/cl-home     /home                   xfs     defaults        0 0
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
192.168.1.156:/data     /data                   nfs     defaults        0 0
#由于修改了 /etc/fstab，需要重新加载 systemctl
$systemctl daemon-reload
~~~

#### kubernetes使用nfs作为外部存储

~~~shell
#拉取代码
$ git clone https://github.com/kubernetes-incubator/external-storage.git
#进入到nfs-client目录下 修改deploy/deployment.yml文件
#需要将nfs的ip和目录修改
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-client-provisioner
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-client-provisioner
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: 192.168.1.156
            - name: NFS_PATH
              value: /data
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.1.156
            path: /data
#storageclass不做修改 或者修改provisioner的名字 需要与deployment中PROVISIONER_NAME一直
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"
#rbac
$ kubectl apply -f rbac.yml
#test
$ kubectl create -f deploy/test-claim.yaml
#test pod
$ kubectl create -f deploy/test-pod.yaml
#在NFS服务器上的共享目录下的卷子目录中检查创建的NFS PV卷下是否有"SUCCESS" 文件
#删除测试POD
$ kubectl delete -f deploy/test-pod.yaml
#删除测试PVC
$ kubectl delete -f deploy/test-claim.yaml 
~~~

- 遇到的问题

~~~shell
#解决mount挂载问题：wrong fs type, bad option, bad superblock on
#在集群的各个节点安装ntp-utils
$ yum install -y ntp-utils
~~~

### 23.kuboard

~~~shell
#获取token
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}')
~~~

### 24.go语言开发k8s命令行管理工具

#### 1.k8s资源管理的权限控制 

- 创建自己的sa

~~~shell
[root@master1 ~]# kb create sa test-admin
[root@master1 ~]# kb describe sa test-admin
Name:                test-admin
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   test-admin-token-62ncj
Tokens:              test-admin-token-62ncj
Events:              <none>
[root@master1 ~]# kb describe secret test-admin-token-62ncj
Name:         test-admin-token-62ncj
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: test-admin
              kubernetes.io/service-account.uid: 0ed0d881-28a1-48a1-8cd6-ac54ed84195b
Type:  kubernetes.io/service-account-token
Data
====
namespace:  7 bytes
token: ....
ca.crt:     1029 bytes
~~~

- 上面已经获得了sa的鉴权信息（token） 通过token测试接口

~~~shell
[root@master1 ~]# curl 'https://192.168.1.155:6443/healthz/ping' -k -H 'Authorization: eyJhbGciOiJSUzI1NiIsImtpZCI6InNjZkVYem5JSDdQRkR5b0JaR285S0d5ZUVfeWNuT1dkRlVRNl91dWF2QVEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InRlc3QtYWRtaW4tdG9rZW4tNjJuY2oiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoidGVzdC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBlZDBkODgxLTI4YTEtNDhhMS04Y2Q2LWFjNTRlZDg0MTk1YiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OnRlc3QtYWRtaW4ifQ.SyR6Uo_ghu-CDuhxpIMb1hpM--alwKdnoLsX0MXW9smpudVs836qD8HhZuyRH4D4TxO6iFhwdYyzl3pp3DjPKIbk1szuzgmKaRUSx4sguYzXEA_gtgfCG-7b-mGsTznsu4VWIAOVLSm9aV0A-NMc2VeSnnAMbIDZJLuAHsxSVPxYnaffU6UJkaHueu2OKMHjuSVBwu5GLYo7fs42GNm4d-wFXVjIyth1dNsiz0IFIZHjIZB3aXsY0cY82waepJa8Zns0wN6BQJvZwWcvm4KN_oeuxjasD-4lZ4F6IhhSzMoLgeKEduyjeFgRRRhRZwNHhrRGo6iLjOVOu8CcyE5EgQ'

{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/healthz/ping\"",
  "reason": "Forbidden",
  "details": {

  },
  "code": 403
~~~

没有得到我们想要的结果是因为test-admin这个sa并没有被授权，接下来我们将创建一个role并将其授权（rolebinding）给test-admin这个sa。鉴权是指sa的用户密码或者token是否正确，授权是在鉴权成功后，看这个sa是否拥有访问集群资源的权限（看一下rbac）。对资源的操作可以细分为create，update，delete，patch，get，list，watch（前四个对应写，后三个对应读）。

- sa访问集群资源方法

~~~shell
#设置一个账号鉴权信息
[root@master1 .kube]# kubectl config set-credentials test-admin --token eyJhbGciOiJSUzI1NiIsImtpZCI6InNjZkVYem5JSDdQRkR5b0JaR285S0d5ZUVfeWNuT1dkRlVRNl91dWF2QVEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InRlc3QtYWRtaW4tdG9rZW4tNjJuY2oiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoidGVzdC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBlZDBkODgxLTI4YTEtNDhhMS04Y2Q2LWFjNTRlZDg0MTk1YiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OnRlc3QtYWRtaW4ifQ.SyR6Uo_ghu-CDuhxpIMb1hpM--alwKdnoLsX0MXW9smpudVs836qD8HhZuyRH4D4TxO6iFhwdYyzl3pp3DjPKIbk1szuzgmKaRUSx4sguYzXEA_gtgfCG-7b-mGsTznsu4VWIAOVLSm9aV0A-NMc2VeSnnAMbIDZJLuAHsxSVPxYnaffU6UJkaHueu2OKMHjuSVBwu5GLYo7fs42GNm4d-wFXVjIyth1dNsiz0IFIZHjIZB3aXsY0cY82waepJa8Zns0wN6BQJvZwWcvm4KN_oeuxjasD-4lZ4F6IhhSzMoLgeKEduyjeFgRRRhRZwNHhrRGo6iLjOVOu8CcyE5EgQ
User "test-admin" set.
#设置集群的访问信息
[root@master1 .kube]# kubectl config set-cluster kubernetes --server https://192.168.1.155:6443 --certificate-authority /opt/ca.pem --embed-certs=true
Cluster "kubernetes" set.
#创建Context 连接集群和鉴权信息
[root@master1 .kube]# kubectl config set-context kubernetes-ctx --cluster kubernetes --user test-admin
Context "kubernetes-ctx" created.
#使用这个context
[root@master1 .kube]# kubectl config use-context kubernetes-ctx
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:test-admin" cannot list pods in the namespace "default"
~~~

报错的原因是test-admin没有role进行绑定，接下来将创建一个role并且与sa绑定（授权）

- 授权过程

~~~shell
#创建一个role
[root@master1 .kube]# kb get role test-admin-role -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2019-10-25T07:38:24Z"
  name: test-admin-role
  namespace: default
  resourceVersion: "1583602"
  selfLink: /apis/rbac.authorization.k8s.io/v1/namespaces/default/roles/test-admin-role
  uid: 0491fd33-893a-4edc-9edf-87af5a0614d5
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - services
  - secrets
  verbs:
  - create
  - update
  - delete
  - patch
  - get
  - list
  - watch
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - create
  - update
  - delete
  - patch
  - get
  - list
  - watch
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs:
  - create
  - update
  - delete
  - patch
  - get
  - list
  - watch
#创建一个rolebinding
[root@master1 .kube]# kb get rolebinding test-admin-role-binding -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: "2019-10-25T07:43:58Z"
  name: test-admin-role-binding
  namespace: default
  resourceVersion: "1584392"
  selfLink: /apis/rbac.authorization.k8s.io/v1/namespaces/default/rolebindings/test-admin-role-binding
  uid: b87f6cf7-608c-4a36-b946-c6401437c2bc
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: test-admin-role
subjects:
- kind: ServiceAccount
  name: test-admin
  namespace: default
#使用kubernetes-ctx
[root@master1 .kube]# kubectl config use-context kubernetes-ctx
Switched to context "kubernetes-ctx".
#测试
[root@master1 .kube]# kb get po
NAME                                      READY   STATUS    RESTARTS   AGE
nfs-client-provisioner-7b84c4f769-x5jxb   1/1     Running   1          6d5h
nginx                                     1/1     Running   1          23h
~~~

#### 2.开发基于http的应用

~~~go
package main

import (
    "bytes"
    "flag"
    "fmt"
    "io/ioutil"
    "net/http"
    "strings"
)

func main() {
    var host string
    var port int
    flag.StringVar(&host, "host", "0.0.0.0", "host to listen")
    flag.IntVar(&port, "port", 9090, "port to listen")
    flag.Parse()

    http.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
        defer req.Body.Close()

        buffer := bytes.NewBuffer(nil)
        //print request method and URI
        buffer.WriteString(fmt.Sprintln(req.Method, req.RequestURI))

        buffer.WriteString(fmt.Sprintln())
        //print headers
        for k, v := range req.Header {
            buffer.WriteString(fmt.Sprintln(k+":", strings.Join(v, ";")))
        }
        buffer.WriteString(fmt.Sprintln())

        //print request body
        reqBody, readErr := ioutil.ReadAll(req.Body)
        if readErr != nil {
            buffer.WriteString(fmt.Sprintln(readErr))
        } else {
            buffer.WriteString(fmt.Sprintln(string(reqBody)))
        }

        outputData := buffer.Bytes()
        fmt.Println(string(outputData))
        w.Write(outputData)
    })

    //listen and serve
    err := http.ListenAndServe(fmt.Sprintf("%s:%d", host, port), nil)
    fmt.Println("listen err,", err)
}
~~~

- 制作镜像

~~~go
//dockerFile
FROM alpine:latest
RUN mkdir -p /home/app/
ADD ./echo-go /home/app/
CMD /home/app/echo-go -host 0.0.0.0 -port 9090
//在打包之前，我们来关注一个小的细节，这个细节将会影响我们的二进制文件是否能够正常启动。在上面的编译过程中，我们直接使用了 go build 的编译方式，不过由于我们的应用里面使用了 net/http 这个库，这个库的编译是需要依赖 CGO 的，由于我们的 alpine:latest 镜像很简洁，所以为了能够让我们的应用能够在这个镜像里面运行起来，我们需要在编译应用的时候，加上一些编译选项
CGO_ENABLED=0 go build -a -ldflags '-extldflags "-static"' -o echo-go echo-go.go
//打包
$ tar cvf echo-go.tar Dockerfile echo-go
//tar tf 
$ tar tf echo-go.tar
echo-go
Dockerfile
//image-build.go
package main
import (
    "context"
    "flag"
    "fmt"
    "io"
    "os"

    "github.com/docker/docker/api/types"
    "github.com/docker/docker/client"
)

const (
    DockerClientVersion = "1.37"
)

// 为了突出 Docker 镜像构建的部分功能，我们这里的 Tar 文件使用手动打好的指定的 Tar 包。
func main() {
    var tarFile string
    var imageName string
    flag.StringVar(&tarFile, "tar-file", "", "tar file to build docker image")
    flag.StringVar(&imageName, "image-name", "", "dest image name")
    flag.Parse()
    if tarFile == "" || imageName == "" {
        fmt.Println("Err: no tar file or dest image name specified")
        return
    }

    // 创建镜像构建的 Client 对象
    imageBuildClient, err := client.NewClientWithOpts(client.WithVersion(DockerClientVersion))
    if err != nil {
        fmt.Println("Err: create docker build client error,", err.Error())
        return
    }

    // 打开 Tar 文件
    tarFileFp, err := os.Open(tarFile)
    if err != nil {
        fmt.Println("Err: open tar file error,", err.Error())
        return
    }

    defer tarFileFp.Close()

    // 发送构建请求
    ctx := context.Background()

    imageBuildResp, err := imageBuildClient.ImageBuild(ctx, tarFileFp, types.ImageBuildOptions{
        Tags:       []string{imageName},
        Dockerfile: "./Dockerfile",
    })
    if err != nil {
        fmt.Println("Err: send image build request error,", err.Error())
        return
    }
    defer imageBuildResp.Body.Close()

    // 打印构建输出
    _, err = io.Copy(os.Stdout, imageBuildResp.Body)
    if err != nil {
        fmt.Println("Err: read image build response error,", err.Error())
        return
    }
}
//编译
$ go build image-build.go
$ ./image-build -tar-file 'echo-go.tar' -image-name 'gaoxin2020/echo-test/echo-go:1.0'
//image-push.go
package main

import (
    "context"
    "encoding/base64"
    "encoding/json"
    "flag"
    "fmt"
    "io"
    "os"

    "github.com/docker/docker/api/types"
    "github.com/docker/docker/client"
)

const (
    DockerClientVersion = "1.37"
)

func main() {
    var imageName string
    flag.StringVar(&imageName, "image-name", "", "image name to push")
    flag.Parse()
    if imageName == "" {
        fmt.Println("Err: no image name specified")
        return
    }

    // 创建镜像推送的 Client 对象
    imagePushClient, err := client.NewClientWithOpts(client.WithVersion(DockerClientVersion))
    if err != nil {
        fmt.Println("Err: create docker push client error,", err.Error())
        return
    }

    // 构建镜像推送的鉴权信息
    imagePushAuthConfig := types.AuthConfig{
        Username: "xxx",
        Password: "xxx",
    }
    imagePushAuth, _ := json.Marshal(&imagePushAuthConfig)

    // 发送镜像推送的请求
    ctx := context.Background()
    imagePushResp, err := imagePushClient.ImagePush(ctx, imageName, types.ImagePushOptions{
        RegistryAuth: base64.URLEncoding.EncodeToString(imagePushAuth),
    }) 
    if err != nil {  
        fmt.Println("Err: send image push request error,", err.Error())
        return 
    }
    defer imagePushResp.Close() 

    // 打印镜像推送的输出
    _, err = io.Copy(os.Stdout, imagePushResp)
    if err != nil {
        fmt.Println("Err: read image push response error,", err.Error())
        return
    }
}
~~~

#### 3.cobra框架

~~~go
// cobra.Command 的定义
type Command struct {
    // Use 表示用一句话来描述这个命令作用，这段话的第一个单词会被作为这个命令的名称
    // 这个设置在子命令中生效，对于根命令则没有意义
    Use string

    // Alias 可以用来给子命令定义别名，除了使用 Use 中的第一个单词作为子命令外，你还可以使用这个 Alias 
    // 里面定义的任何一个名称作为子命令名称
    Aliases []string

    // SuggestFor 定义一组提示命令，当输入匹配其中任何一个命令的时候，会提示是否希望输入的为 echo 命令 
    SuggestFor []string

    // Short 是用来在帮助信息位置显示的简短命令帮助
    Short string

    // Long 是用来在使用命令 'help <this-command>' 显示帮助信息时显示的长文字
    Long string

    // Example 用来定义子命令使用的具体示例，可以在里面定义多行不同的命令使用样例，供用户参考，这一点在
    // Kubectl 命令中体现的非常明显，因为 Kubectl 命令很复杂，参数也很多，所有样例会极大方便用户
    Example string

    // ValidArgs 是一组可用在 Bash 补全中的合法的非选项参数
    ValidArgs []string

    // Args 表示期望的参数
    Args PositionalArgs

    // ArgAliases 是 ValidArgs 的一组别名
    // 这些参数不会在 Bash 补全中提示给用户，但是如果手动输入的话，也是允许的
    ArgAliases []string

    // BashCompletionFunction 是 Bash 自动补全生成器使用的自定义函数
    BashCompletionFunction string

    // Deprecated 不为空的时候，在命令执行时都会提示命令已废弃，并且输出这段文字
    Deprecated string

    // Hidden 参数设置为 true 的时候，将无法在命令帮助列表中看到这个命令，但是实际这个命令仍然是可用的，一般用于
    // 对命令做向下兼容的处理，在未来的版本中如果这个命令会废弃，那么先让它隐藏起来会比直接删除较好
    Hidden bool

    // Annotations 定义一些键值对，应用可以用这些注解来分组命令，主要用于标注上面的分组
    Annotations map[string]string

    // Version 定义这个命令的版本。当 Version 值不为空，且命令没有定义 version 选项的时候，会自动给这个命令增加一个
    // boolean 类型，名称为 version 的选项。如果指定这个选项，就会输出这里 Version 的值。
    Version string

    //  下面的这组 Run 函数执行顺序为：
    //   * PersistentPreRun()
    //   * PreRun()
    //   * Run()
    //   * PostRun()
    //   * PersistentPostRun()
    // 所有的函数传入的参数都相同，都是命令名称之后的参数

    // PersistentPreRun 这个命令的子命令都将继承并执行这个函数
    PersistentPreRun func(cmd *Command, args []string)

    // PersistentPreRunE 和 PersistentPreRun 一样，但是遇到错误时可以返回一个错误
    // 一旦这个函数返回的 error 不为 nil，那么执行就中断了。所以你可以在这个函数里面
    // 做诸如权限验证等等全局性的工作
    PersistentPreRunE func(cmd *Command, args []string) error

    // PreRun 这个命令的子命令不会继承和运行这个函数
    PreRun func(cmd *Command, args []string)

    // PreRunE 和 PreRun 一样，但是遇到错误时可以返回一个错误
    // 一旦这个函数返回的 error 不为 nil，那么执行就中断了。所以你可以在这个函数里面
    // 做一些和该命令相关的输入参数检测之类的工作
    PreRunE func(cmd *Command, args []string) error

    // Run 命令核心工作所在的函数，大多数情况下只实现这个命令即可
    Run func(cmd *Command, args []string)

    // RunE 和 Run 一样，但是遇到错误时可以返回一个错误
    // 一旦这个函数返回的 error 不为 nil，那么执行就中断了。
    RunE func(cmd *Command, args []string) error

    // PostRun 在 Run 函数执行之后执行
    PostRun func(cmd *Command, args []string)

    // PostRunE 在 PostRun 之后执行，但是可以返回一个错误      
    // 一旦这个函数返回的 error 不为 nil，那么执行就中断了。    
    PostRunE func(cmd *Command, args []string) error      

    // PersistentPostRun 在 PostRun 之后执行，这个命令的子命令都将继承并执行这个函数    
    PersistentPostRun func(cmd *Command, args []string)     

    // PersistentPostRunE 和 PersistentPostRun 一样，但是可以返回一个错误   
    // 一旦这个函数返回的 error 不为 nil，那么执行就中断了。   
    PersistentPostRunE func(cmd *Command, args []string) error   

    // SilenceErrors 设置为 true 时可以在命令执行过程中遇到任何错误时，不显示错误   
    SilenceErrors bool   

    // SilenceUsage 设置为 true 时可以在命令执行遇到输入错误时，不显示使用方法   
    SilenceUsage bool   

    // DisableFlagParsing 设置为 true 时将禁用选项解析功能，这样命令之后所有的内容  
    // 都将作为参数传递给命令   
    DisableFlagParsing bool   

    // DisableAutoGenTag 在生成命令文档的时候是否显示 gen tag
    DisableAutoGenTag bool

    // DisableFlagsInUseLine 设置为 true 的时候，将不会在命令帮助信息或者文档中显示命令支持的选项
    DisableFlagsInUseLine bool

    // DisableSuggestions 禁用命令提示
    DisableSuggestions bool

    // SuggestionsMinimumDistance 定义显示命令提示的最小的 Levenshtein 距离
    SuggestionsMinimumDistance int

    // TraverseChildren 在执行该命令子命令前，解析所有父命令的选项
    TraverseChildren bool

    //FParseErrWhitelist 定义可以被忽略的解析错误
    FParseErrWhitelist FParseErrWhitelist
}
~~~

- 简单模板

~~~go
//模板1
package main
import (
    "fmt"
    "strings"

    "github.com/spf13/cobra"
)
func main() {
    // 定义一个命令，直接输出命令行参数
    echoCmd := cobra.Command{
        // 命令名称
        Use: "echo",
        // 命令执行过程
        Run: func(cmd *cobra.Command, args []string) { 
            fmt.Println(strings.Join(args, " "))
        }, 
    }

    // 执行命令 
    echoCmd.Execute() 
}
//模板2 常用的命令模板
package main
import (
    "fmt"
    "strings"
    "github.com/spf13/cobra"
)
func main() {
    rootCmd := cobra.Command{
        //定义这个工具默认输出帮助
        Run: func(cmd *cobra.Command, args []string){
            cmd.Help()
        },
    }
    //定义一个子命令 直接输出命令行参数
    echoCmd := cobra.Command{
        //子命令描述，第一个单词作为子命令名称
        Use: "echo inputs",
        //子命令别名
        Aliases: []string{
            "copy",
            "repeat",
        },
        // 命令提示，默认情况如果输入的子命令不存在会提示 unknown cmd，
        // 但是如果定义了 SuggestFor 的情况下，如果输入的命令不存在，会去 SuggestFor
        // 里面查找是否有匹配的字符串，如果有，则提示是否期望输入的是 echo 命令
        SuggestFor: []string{
            "ech","cp",
        },
        DisableSuggestions: true,
        SuggestionsMinimumDistance: 2,
        DisableFlagsInUseLine: true,
        // 简单扼要概括下命令的用途
        Short: "echo is a command",
        // 想说什么都在这里说吧，越详细越好，可以使用 `` 来跨行输入
        Long: `echo is a command`,
        // 当你在新版本废弃这个命令的时候，可以先隐藏，让用户优先使用替代品或者看不到，
        // 但是处于向下兼容目的，这个命令仍然是可用的，只是在帮助列表里面看不到
        //Hidden: true,

        // 当你需要废弃这个命令的时候设置。废弃的意思意味着未来版本可能删除这个命令。
        // 标注为废弃的命令在执行的时候，都会打印命令已废弃的提示信息以及这个设置的提示信息。
        //Deprecated: "will be deleted in version 2.0",

        //注解，用于代码层面的命令分组 不会显示在命令行输入中
        Annotations: map[string]string{
            "group": "user",
            "require-auth": "none",
        },
        SilenceErrors: true,
        //SilenceUsage:  true,
        // Version 定义版本
        Version: "1.0.0",
        // 是否禁用选项解析
        //DisableFlagParsing: true,

        //DisableAutoGenTag: true,

        // PersistentPreRun: func(cmd *cobra.Command, args []string) {
        //     fmt.Println("hahha! let me check echk")
        // },
        PersistentPreRunE: func(cmd *cobra.Command, args []string) error {
            //    return errors.New("invalid parameter")
            return nil
        },
        
        // 子命令执行过程
        // Run: func(cmd *cobra.Command, args []string) {
        //     fmt.Println(strings.Join(args, " "))
        // },
        RunE: func(cmd *cobra.Command, args []string) error{
            fmt.Println(strings.Join(args," "))
            return nil
        },
        PostRun: func(cmd *cobra.Command, args []string){
            fmt.Println(" i am a post run")
        },
    }
    //添加子命令根命令
    rootCmd.AddCommand(&echoCmd)
    //执行根命令
    rootCmd.Execute()
}
~~~

cobra支持两种选项 一种是命令自身的选项 另一种是父命令继承过来的选项 命令自身的选项可以通过函数Flags来添加 而继承过来的选项则是父命令通过PersistentFlags设置的选项  例如

~~~shell
$ kb -n default get pods -l "label" -o json
~~~

上面的命令中 -n是命令kb的选项 这个命令不仅提供给get 也提供给其他命令使用 每个子命令应该继承-n这个选项

-o 和 -l 是get 自身的选项 代码如下

~~~go
package main

import(
    "fmt"
    "github.com/spf13/cobra"
)
var namespace string
func main(){
 
    rootCmd := cobra.Command{
        Use : "kb",
    }
    //添加全局的选项 所有的子命令都可以继承
    rootCmd.PersistentFlags().StringVarP(&namespace,"namespace","n","","if present, the namespace scope for this hahahaha")
    
    //一级子命令 get
    var outputFormat string
    var outputLabel string
    getCmd := cobra.Command{
        Use : "get",
        Run : func(cmd *cobra.Command, args []string){
            fmt.Println("print flags")
            fmt.Printf("Flags: namespace=[%s], selector=[%s], outputformat=[%s]\n", namespace, outputLabel, outputFormat)
            fmt.Println("Print args...")
            for _, arg := range args {
                fmt.Println("args:",arg )
            }
        },
    }
        //添加命令选项 仅子命令可用
    //func StringVarP(p *string, name, shorthand string, value string, usage string)
    getCmd.PersistentFlags().StringVarP(&outputFormat,"output","o","","qqqqqq")
    getCmd.PersistentFlags().StringVarP(&outputLabel,"label","l","","eeeeee")

    var teststr string
    testCmd := cobra.Command{
        Use : "test",
        Run : func(cmd *cobra.Command, args []string){
            fmt.Println("print flags")
            fmt.Printf("Flags: namespace=[%s], selector=[%s], outputformat=[%s], teststr=[%s]\n", namespace, outputLabel, outputFormat, teststr)
            fmt.Println("Print args...")
            for _, arg := range args {
                fmt.Println("args:",arg )
            }
        
        },
    }
    testCmd.Flags().StringVarP(&teststr,"test","t","","")
    //组装命令
    rootCmd.AddCommand(&getCmd)
    getCmd.AddCommand(&testCmd)
    rootCmd.Execute()
}
~~~

#### 4.pod源码

1. 定义一个pod 一般需要：pod的名称和标签 镜像 对外暴露的端口 启动命令和工作目录

~~~go
// $GOPATH/src/k8s.io/api/core/v1/types.go
// Pod 是一组可以在集群中运行的容器。通过 Kubernetes 的客户端创建，然后调度到集群中的机器上面运行。
type Pod struct {
    // 资源 API 元数据
    metav1.TypeMeta `json:",inline"`
    // 资源标准的元数据
    metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

    // Pod 的期望规格
    Spec PodSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

    // Pod 的状态，该状态由 Kubernetes 系统自动更新，为只读信息
    Status PodStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
~~~

 我们看到，Pod 的组成部分有四块。其中 metav1.TypeMeta 和 metav1.ObjectMeta 保存了 Pod  的基础元信息，这两个结构体是所有的 Kubernetes 对象都会有的。Pod 自身的规格信息需要通过 Spec 字段来设置。而 Status  字段则保存着系统最近一次更新的 Pod 的状态信息，这个信息是只读信息，所以我们无法设置它。 

 在研究 Pod 的 Spec 之前，先看一下所有 Kubernetes 对象都会有的两个属性。首先看下 metav1.TypeMeta 的结构： 

~~~go
// $GOPATH/src/k8s.io/apimachinery/pkg/apis/meta/v1/types.go
// TypeMeta 描述了请求和回复中该 Kubernetes 对象的类型和 API 版本
type TypeMeta struct {
    // Kind 用来表示资源的类型，比如 Pod、Service、Ingress、Deployment 等
    Kind string `json:"kind,omitempty" protobuf:"bytes,1,opt,name=kind"`

    // APIVersion 定义了操作该资源的 API 的版本
    APIVersion string `json:"apiVersion,omitempty" protobuf:"bytes,2,opt,name=apiVersion"`
}
~~~

 再看下 metav1.ObjectMeta 的结构，这个结构的内容会比较多，基本上用来描述资源自身的一些属性： 

~~~go
// $GOPATH/src/k8s.io/api/core/v1/types.go

// ObjectMeta 是所有的持久化的 Kubernetes 资源都拥有的属性
type ObjectMeta struct {
    // Name 在命名空间中必须是唯一的，在创建资源时必须提供。
    Name string `json:"name,omitempty" protobuf:"bytes,1,opt,name=name"`

    // 仅当没有指定 Name 的时候生效。
    // 在没有指定 Name 的情况下，服务端使用这个参数作为生成名称的前缀。
    GenerateName string `json:"    ,omitempty" protobuf:"bytes,2,opt,name=generateName"`

    // 资源所在命名空间，无法更新
    Namespace string `json:"namespace,omitempty" protobuf:"bytes,3,opt,name=namespace"`

    // SelfLink 表示该资源的 URL 只读  
    SelfLink string `json:"selfLink,omitempty" protobuf:"bytes,4,opt,name=selfLink"`

    // UID 表示该对象基于时空的唯一性ID，由服务端在对象生成成功后创建，只读属性  
    UID types.UID `json:"uid,omitempty"  protobuf:"bytes,5,opt,name=uid,casttype=k8s.io/kubernetes/pkg/types.UID"`

    // 该值表示资源在系统内部的版本，属于不可修改的隐藏值。    
    ResourceVersion string `json:"resourceVersion,omitempty"  protobuf:"bytes,6,opt,name=resourceVersion"`      

    // Generation 是一个序列号，表示期望状态的某个具体的版本，由系统填充，只读属性
    Generation int64 `json:"generation,omitempty" protobuf:"varint,7,opt,name=generation"` 

    // CreationTimestamp 是资源创建成功的服务端时间，只读属性
    CreationTimestamp Time `json:"creationTimestamp,omitempty" protobuf:"bytes,8,opt,name=creationTimestamp"`

    // DeletionTimestamp 是系统设置的资源将被删除的时间。当用户请求以优雅的方式删除资源的时候，服务端才会去设置这个值。
    DeletionTimestamp *Time `json:"deletionTimestamp,omitempty" protobuf:"bytes,9,opt,name=deletionTimestamp"`

    // 只读属性。当设置了 DeletionTimestamp 时生效，表示资源从系统删除所允许的宽限时间。
    DeletionGracePeriodSeconds *int64 `json:"deletionGracePeriodSeconds,omitempty" protobuf:"varint,10,opt,name=deletionGracePeriodSeconds"`

    // 资源的标签，用来当作资源的选择器使用。
    Labels map[string]string `json:"labels,omitempty" protobuf:"bytes,11,rep,name=labels"`

    // 资源的注解，提供给外部的系统使用。
    Annotations map[string]string `json:"annotations,omitempty" protobuf:"bytes,12,rep,name=annotations"`

    // 该资源所依赖的对象列表。如果这个列表里面所有的对象都被删除了，那么这个对象也会被回收掉。
    OwnerReferences []OwnerReference `json:"ownerReferences,omitempty" patchStrategy:"merge" patchMergeKey:"uid" protobuf:"bytes,13,rep,name=ownerReferences"`

    // 在对象创建的时候，用来设置一些系统参数的控制器列表。这个列表里面的条目表示还没有对该对象进行应用的控制器。如果列表为空，
    // 则表示该对象已经被初始化了。
    Initializers *Initializers `json:"initializers,omitempty" protobuf:"bytes,16,opt,name=initializers"`

    // 在ZZ+资源删除之前，必须为空。每个条目表示一个负责从列表中删除资源的组件。当 deletionTimestamp 不为空的时候，该列表的条目无法删除。
    Finalizers []string `json:"finalizers,omitempty" patchStrategy:"merge" protobuf:"bytes,14,rep,name=finalizers"`

    // 资源所属的集群名称，暂时无用
    ClusterName string `json:"clusterName,omitempty" protobuf:"bytes,15,opt,name=clusterName"`
}
~~~

 在 metav1.ObjectMeta 中，我们目前主要需要关注的是 Name 和 Labels。因为第一个表示了该对象的唯一性名称，第二个表示了该对象的标签属性，这个标签属性在其它资源对象引用该对象时使用。 

 接下来，让我们详细地看一下定义 Pod 的 PodSpec 的内容： 

~~~go
// $GOPATH/src/k8s.io/api/core/v1/types.go

// PodSpec 描述了生成一个 Pod 所需要的参数
type PodSpec struct {
    // 可以挂载到 Pod 中容器的卷
    Volumes []Volume `json:"volumes,omitempty" patchStrategy:"merge,retainKeys" patchMergeKey:"name" protobuf:"bytes,1,rep,name=volumes"`

    // Pod 的初始化容器。初始化容器是在主容器启动之前依次执行的。如果任何一个初始化容器执行失败了，那么 Pod 的启动就被认为是失败了。
    // 如果 Pod 启动失败，会按照 RestartPolicy 所指定的策略进行下一步处理。初始化容器的名称和主容器的名称在所有的容器中必须都是唯一的。
    // 初始化容器可以没有生命循环的操作 Readiness 探针或者 Liveness 探针。
    // 初始化容器的资源需求在调度期间是通过查找每个资源的最大的需求/限制，然后使用最大值或者正常主容器所需值的总和。
    // 初始化容器目前无法更新或者删除。
    InitContainers []Container `json:"initContainers,omitempty" patchStrategy:"merge" patchMergeKey:"name" protobuf:"bytes,20,rep,name=initContainers"`

    // Pod 中的一组容器，当前不支持删除或者更新。
    Containers []Container `json:"containers" patchStrategy:"merge" patchMergeKey:"name" protobuf:"bytes,2,rep,name=containers"`

    // Pod 中容器的重启策略。可选值为 Always、OnFailure、Never。默认为 Always。
    RestartPolicy RestartPolicy `json:"restartPolicy,omitempty" protobuf:"bytes,3,opt,name=restartPolicy,casttype=RestartPolicy"`

    // Pod 用来优雅地终止执行所需要的时间。可以通过 DELETE 请求来缩短这个时间。
    // 这个值必须是非负整数。当设置为 0 的时候，表示立即删除 Pod。
    // 如果这个值设置为 nil，那么就使用默认的优雅退出时间。默认值为 30 秒。
    // 优雅退出的时间指的是 Pod 中运行的进程收到终止执行信号后到最终被 KILL 信号强制终止时所经历的时间。
    // 把这个值设置的比你的进程清理时间稍微长一点。
    TerminationGracePeriodSeconds *int64 `json:"terminationGracePeriodSeconds,omitempty" protobuf:"varint,4,opt,name=terminationGracePeriodSeconds"`

    // 这个值是相对于 Pod 的 StartTime 到最终被系统主动标记为失败并且 KILL 相关容器所经历的时间。必须为正整数。
    ActiveDeadlineSeconds *int64 `json:"activeDeadlineSeconds,omitempty" protobuf:"varint,5,opt,name=activeDeadlineSeconds"`

    // Pod 中的 DNS 策略。默认为 ClusterFirst。
    // 可选值为 ClusterFirstWithHostNet，ClusterFirst，Default 或者 None。
    DNSPolicy DNSPolicy `json:"dnsPolicy,omitempty" protobuf:"bytes,6,opt,name=dnsPolicy,casttype=DNSPolicy"`

    // 当需要把 Pod 指定分配到某些 Node 上时设置的 Node 选择标签。
    NodeSelector map[string]string `json:"nodeSelector,omitempty" protobuf:"bytes,7,rep,name=nodeSelector"`

    // 该 Pod 中所使用的 ServiceAccount 的名称。
    ServiceAccountName string `json:"serviceAccountName,omitempty" protobuf:"bytes,8,opt,name=serviceAccountName"`

    // 设置是否应该自动挂载 ServiceAccount 的 Secret Token 到容器中。
    AutomountServiceAccountToken *bool `json:"automountServiceAccountToken,omitempty" protobuf:"varint,21,opt,name=automountServiceAccountToken"`

    // 该参数指定将 Pod 调度到某个 Node 上。如果不为空，那么这个 Pod 就会直接被调度到该 Node，假定该 Node 满足资源要求。
    NodeName string `json:"nodeName,omitempty" protobuf:"bytes,10,opt,name=nodeName"`

    // 设置该 Pod 是否需要共享 Host 的网络空间，默认为 false。
    // 如果该参数设置为 true，那么所有使用到的端口必须显式指定。
    HostNetwork bool `json:"hostNetwork,omitempty" protobuf:"varint,11,opt,name=hostNetwork"`

    // 设置 Pod 是否共享 Host 的 PID 空间，默认为 false。
    HostPID bool `json:"hostPID,omitempty" protobuf:"varint,12,opt,name=hostPID"`

    // 设置 Pod 是否共享 Host 的 IPC 空间，默认为 false。
    HostIPC bool `json:"hostIPC,omitempty" protobuf:"varint,13,opt,name=hostIPC"`

    // 在 Pod 中的所有容器间共享一个进程空间。当指定这个参数时，Pod 中的所有容器将可以查看同 Pod 中其它容器的进程或者向它们发信号。
    // 另外在指定该参数时，每个容器中的第一个进程的 PID 将不再是1。HostPID 和 ShareProcessNamespace 不可以同时指定。
    ShareProcessNamespace *bool `json:"shareProcessNamespace,omitempty" protobuf:"varint,27,opt,name=shareProcessNamespace"`

    // 保存 Pod 级别的安全属性和一些常见的容器设置。默认为空。
    SecurityContext *PodSecurityContext `json:"securityContext,omitempty" protobuf:"bytes,14,opt,name=securityContext"`

    // 指定拉取镜像所需要的 Secret。当镜像所在的镜像空间为私有空间的时候，需要指定拉取的 Secret，否则无法拉取镜像。
    ImagePullSecrets []LocalObjectReference `json:"imagePullSecrets,omitempty" patchStrategy:"merge" patchMergeKey:"name" protobuf:"bytes,15,rep,name=imagePullSecrets"`

    // 指定 Pod 的主机名，如果不指定，Pod 将采用系统定义的默认的主机名。
    Hostname string `json:"hostname,omitempty" protobuf:"bytes,16,opt,name=hostname"`

    // 如果指定，那么 Pod 的完整主机名是 <hostname>.<subdomain>.<pod namespace>.svc.<cluster domain> 。
    // 如果不指定，则 Pod 将不会有 Domain Name。
    Subdomain string `json:"subdomain,omitempty" protobuf:"bytes,17,opt,name=subdomain"`

    // 指定 Pod 的调度约束
    Affinity *Affinity `json:"affinity,omitempty" protobuf:"bytes,18,opt,name=affinity"`

    // 指定 Pod 的调度器名称，如果不指定，则使用默认的调度器。
    SchedulerName string `json:"schedulerName,omitempty" protobuf:"bytes,19,opt,name=schedulerName"`

    // 用来指定一组 Pod 可容忍的污点
    Tolerations []Toleration `json:"tolerations,omitempty" protobuf:"bytes,22,opt,name=tolerations"`

    // HostAliases 指定一组 Host 和 IP 的映射关系，这些映射会被注入到 Pod 的 /etc/hosts 文件中。
    // 这个设置只对非 hostNetwork 的 Pod 生效。
    HostAliases []HostAlias `json:"hostAliases,omitempty" patchStrategy:"merge" patchMergeKey:"ip" protobuf:"bytes,23,rep,name=hostAliases"`

    // 指定 Pod 的优先级。其中 system-node-critical 和 system-cluster-critical 是两个特殊的关键字用来表示最高的优先级。
    // 如果需要指定其它的优先级，必须定义一个指定名称的 PriorityClass 的对象。如果不指定，那么 Pod 采用默认的优先级或者如果
    // 没有默认的优先级，那么就是 0。
    PriorityClassName string `json:"priorityClassName,omitempty" protobuf:"bytes,24,opt,name=priorityClassName"`

    // 指定 Pod 的优先级。有多个系统组件会使用到这个参数来决定 Pod 的优先级。
    Priority *int32 `json:"priority,omitempty" protobuf:"bytes,25,opt,name=priority"`

    // 指定 Pod 的 DNS 参数。这里指定的 DNS 参数会和 DNSPolicy 生成的 DNS 配置合并在一起。
    DNSConfig *PodDNSConfig `json:"dnsConfig,omitempty" protobuf:"bytes,26,opt,name=dnsConfig"`

    // 如果指定，所有的 ReadinessGate 都会被检查来确认 Pod 是否就绪。Pod 就绪必须是在所有的容器都已就绪，并且所有 Readiness Gate
    // 中所指定的条件都为 true 的情况下才算就绪。
    ReadinessGates []PodReadinessGate `json:"readinessGates,omitempty" protobuf:"bytes,28,opt,name=readinessGates"`

    // RuntimeClassName 指 node.k8s.io 组中的一个 RuntimeClass 的对象。必须使用这个对象来运行 Pod。如果指定的值没有对应的
    // RuntimeClass，那么这个 Pod 就不会运行。如果这个值没有设置或者设置为空字符串，那么会采用“遗留”的 RuntimeClass。这个对象是
    // 一个隐式的，使用默认的 Runtime Handler 定义的空对象。
    RuntimeClassName *string `json:"runtimeClassName,omitempty" protobuf:"bytes,29,opt,name=runtimeClassName"`

    // 设置是否关于 Service 的信息应该被注入到 Pod 的环境变量中。
    EnableServiceLinks *bool `json:"enableServiceLinks,omitempty" protobuf:"varint,30,opt,name=enableServiceLinks"`
}
~~~

 在上面的内容中，我们目前可以关注其中的一部分，剩余的部分大家可以参考注释，然后在课后做练习的时候自行设置，调试看看效果。我们目前关注一个 Pod 的最小组成部分，即名称、标签和容器。名称和标签都属于 metadata，在上面已经看到过了，我们需要关注下 Pod 里面的容器即  Container 的定义。 

~~~go
// $GOPATH/src/k8s.io/api/core/v1/types.go

// 在 Pod 中运行的一个容器的定义
type Container struct {
    // 容器的名称，Pod 中每个容器的名称必须是在 Pod 层面为唯一的，无法更新
    Name string `json:"name" protobuf:"bytes,1,opt,name=name"`

    // Docker 镜像的名称
    Image string `json:"image,omitempty" protobuf:"bytes,2,opt,name=image"`

    // 容器的启动命令。这些命令不是在 shell 中执行的。如果这个参数没有指定，则使用 Docker 镜像的 ENTRYPOINT。
    Command []string `json:"command,omitempty" protobuf:"bytes,3,rep,name=command"`

    // 容器启动命令的参数列表。如果该参数没有指定，则使用镜像的 CMD 指定的值。
    Args []string `json:"args,omitempty" protobuf:"bytes,4,rep,name=args"`

    // 容器启动命令的工作目录
    WorkingDir string `json:"workingDir,omitempty" protobuf:"bytes,5,opt,name=workingDir"`

    // 设置容器允许暴露的端口，如果该参数不指定，在容器内部监听 0.0.0.0 地址的端口是默认暴露的。
    Ports []ContainerPort `json:"ports,omitempty" patchStrategy:"merge" patchMergeKey:"containerPort" protobuf:"bytes,6,rep,name=ports"`

    // 容器内获取环境变量的来源
    EnvFrom []EnvFromSource `json:"envFrom,omitempty" protobuf:"bytes,19,rep,name=envFrom"`

    // 容器中设置的一组环境变量
    Env []EnvVar `json:"env,omitempty" patchStrategy:"merge" patchMergeKey:"name" protobuf:"bytes,7,rep,name=env"`

    // 该容器所需要的资源
    Resources ResourceRequirements `json:"resources,omitempty" protobuf:"bytes,8,opt,name=resources"`

    // 挂载到容器文件系统上的卷
    VolumeMounts []VolumeMount `json:"volumeMounts,omitempty" patchStrategy:"merge" patchMergeKey:"mountPath" protobuf:"bytes,9,rep,name=volumeMounts"`

    // 定义容器使用的一组块设备
    VolumeDevices []VolumeDevice `json:"volumeDevices,omitempty" patchStrategy:"merge" patchMergeKey:"devicePath" protobuf:"bytes,21,rep,name=volumeDevices"`

    // 容器是否存活的周期性检查探针
    LivenessProbe *Probe `json:"livenessProbe,omitempty" protobuf:"bytes,10,opt,name=livenessProbe"`

    // 容器服务是否就绪的周期性检查探针
    ReadinessProbe *Probe `json:"readinessProbe,omitempty" protobuf:"bytes,11,opt,name=readinessProbe"`

    // 响应容器生命周期时间的动作
    Lifecycle *Lifecycle `json:"lifecycle,omitempty" protobuf:"bytes,12,opt,name=lifecycle"`

    // 容器终止执行的消息存储文件路径，默认为 /dev/termination-log
    TerminationMessagePath string `json:"terminationMessagePath,omitempty" protobuf:"bytes,13,opt,name=terminationMessagePath"`

    // 定义从容器中获取容器终止执行的消息策略。如果定义为 File，那么会使用 terminationMessagePath 的内容来填充容器的状态信息；
    // 如果指定为 FallbackToLogsOnError 则会在容器终止执行消息为空并且容器因执行失败而退出的情况下，使用容器的最后一块日志来填充
    // 容器的状态信息
    TerminationMessagePolicy TerminationMessagePolicy `json:"terminationMessagePolicy,omitempty" protobuf:"bytes,20,opt,name=terminationMessagePolicy,casttype=TerminationMessagePolicy"`

    // 镜像的拉取策略，可选值为 Always、Never 和 IfNotPresent。
    ImagePullPolicy PullPolicy `json:"imagePullPolicy,omitempty" protobuf:"bytes,14,opt,name=imagePullPolicy,casttype=PullPolicy"`

    // 容器执行时所采用的安全策略选项
    SecurityContext *SecurityContext `json:"securityContext,omitempty" protobuf:"bytes,15,opt,name=securityContext"`

    // 是否开启交互式的模式。在开启交互式的模式下，容器在启动的时候会给 Stdin 分配一块缓冲区。如果该参数设置为 false，
    // 那么在容器中从 Stdin 读取数据是读取不到的。
    Stdin bool `json:"stdin,omitempty" protobuf:"varint,16,opt,name=stdin"`

    // 当上面的参数 Stdin 设置为 true 的情况下，Stdin 输入流将在所有打开的会话中有效。如果设置 StdinOnce 为 true，
    // 那么 Stdin 会在容器启动的时候打开，在第一个连接进来的会话后清空，直到容器重新启动后才会被再次打开。
    // 当这个参数设置为 false 的情况下，容器进行从 Stdin 读取时永远不会返回 EOF 错误。
    StdinOnce bool `json:"stdinOnce,omitempty" protobuf:"varint,17,opt,name=stdinOnce"`

    // 容器是否需要为自己分配一个 TTY。如果需要设置为 true，那么要求上面的 Stdin 也设置为 true
    TTY bool `json:"tty,omitempty" protobuf:"varint,18,opt,name=tty"`
}
~~~

#### 5.service源码

 在上一节实验中，通过对 Pod 结构体的学习，我们了解了 Kubernetes 中的资源的基本结构包括 metadata、spec 和 status。所以 Service 的基本结构也是如此： 

~~~go
// $GOPATH/src/k8s.io/api/core/v1/types.go

type Service struct {
    metav1.TypeMeta `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`
    Spec ServiceSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`
    Status ServiceStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
~~~

 在上面的结构中，metadata 部分我们已经熟悉了。Status 部分是只读的内容，我们先看看 Service 的 Spec 的定义。 

```go
// $GOPATH/src/k8s.io/api/core/v1/types.go

// ServiceSpec 描述了 Service 资源的属性
type ServiceSpec struct {
    // Service 暴露的所有端口
    Ports []ServicePort `json:"ports,omitempty" patchStrategy:"merge" patchMergeKey:"port" protobuf:"bytes,1,rep,name=ports"`

    // 将 Service 的流量路由给拥有标签匹配 Selector 中定义的标签的 Pod。如果不指定或者为空，那么就假设这个
    // 服务有外部的进程来管理 Service 对应的 Endpoints，Kubernetes 在这种情况下不会去修改这个 Endpoints。
    // Selector 仅在 ServiceType 为 ClusterIP、NodePort 和 LoadBalancer 时生效。如果 ServiceType
    // 为 ExternalName，那么 Selector 值会被忽略。
    Selector map[string]string `json:"selector,omitempty" protobuf:"bytes,2,rep,name=selector"`

    // ClusterIP 是集群分配给 Service 的一个随机的 IP 地址。如果这个 IP 地址是手动指定的，那么要看这个 IP 是否已经
    // 被其它服务所占用。如果没有占用，那么就会分配这个 Service，否则 Service 的创建会失败。
    // 这个值不可以通过 Update 请求来更新。可选值为 None、空字符串和合法的 IP 地址。
    // 可以为无头 Service 指定 None，这个时候不需要转发流量。
    // ClusterIP 仅在 ServiceType 为 ClusterIP、NodePort 和 LoadBalancer 时生效。如果 ServiceType
    // 为 ExternalName，那么 ClusterIP 值会被忽略。
    ClusterIP string `json:"clusterIP,omitempty" protobuf:"bytes,3,opt,name=clusterIP"`

    // Type 参数定义如何暴露这个 Service。 默认设置为 ClusterIP。可选值为 ExternalName、ClusterIP、NodePort 和
    // LoadBalancer。其中 ExternalName 表示使用指定的 externalName。
    // ClusterIP 表示为 Service 分配一个集群内部的 IP 并且将流量负载到 Service 对应的 Endpoints 去。如果 ClusterIP 设置为 None，
    // 将不会给 Service 分配一个虚拟 IP，Endpoints 就将作为一组端点暴露出来，而不是一个稳定的 IP。
    // NodePort 表示除了给 Service 分配一个集群内的 IP 之外，还会在每个 Node 上面为这个 Service 暴露一个映射端口。通过这个暴露出来的
    // 端口，Node 上的集群外的服务可以访问集群内的 Service。
    // LoadBalancer 基于 NodePort 之上，通过创建一个外部的负载均衡器，然后通过 NodePort 的端口路由到集群内部的 Service。
    Type ServiceType `json:"type,omitempty" protobuf:"bytes,4,opt,name=type,casttype=ServiceType"`

    // 指定一组外部允许访问集群内服务的 IP。这些 IP 本身不属于 Kubernetes 系统。
    // 用户需要自己确保这些 IP 是允许访问集群内服务的。一个常见的例子是外部的负载均衡器。
    ExternalIPs []string `json:"externalIPs,omitempty" protobuf:"bytes,5,rep,name=externalIPs"`

    // 用来设置 Session 的亲和性。可选值为 ClientIP 和 None。默认为 None。
    SessionAffinity ServiceAffinity `json:"sessionAffinity,omitempty" protobuf:"bytes,7,opt,name=sessionAffinity,casttype=ServiceAffinity"`

    // 只适用于 Service Type 为 LoadBalancer 的情况。用来指定负载均衡器的 IP 地址。
    LoadBalancerIP string `json:"loadBalancerIP,omitempty" protobuf:"bytes,8,opt,name=loadBalancerIP"`

    // 如果指定的参数被平台所支持，将控制只允许指定的 Client IP 的流量经过云厂商负载均衡器。
    LoadBalancerSourceRanges []string `json:"loadBalancerSourceRanges,omitempty" protobuf:"bytes,9,opt,name=loadBalancerSourceRanges"`

    // 该参数指定 kubedns 或者其他 DNS 返回的一个该 Service 的 CNAME 记录。这个中间不存在任何代理。
    ExternalName string `json:"externalName,omitempty" protobuf:"bytes,10,opt,name=externalName"`

    // 该参数表示 Service 期望如何路由外部的流量到 Node 或者是 Cluster 层面的 Endpoints。
    // 如果指定为 Local，那么会保留客户端的 IP 地址，对 LoadBalancer 和 NodePort 类型的服务来说，会避免第二跳，
    // 潜在的风险是导致流量负载的不均衡。
    // 如果指定为 Cluster，那么客户端的 IP 地址将被忽略，有可能导致流量负载的第二跳，但是可以保证整体流量的负载均衡。
    ExternalTrafficPolicy ServiceExternalTrafficPolicyType `json:"externalTrafficPolicy,omitempty" protobuf:"bytes,11,opt,name=externalTrafficPolicy"`

    // 该参数指定 Service 健康检查的端口。如果没有指定的话，将自动创建一个关联到分配的 NodePort 的检查端口。
    // 只在 Service Type 为 LoadBalancer 和 ExternalTrafficPolicy 设置为 Local 的时候生效。
    HealthCheckNodePort int32 `json:"healthCheckNodePort,omitempty" protobuf:"bytes,12,opt,name=healthCheckNodePort"`

    // 当设置为 true 时，表示 Endpoints 中的 notReadyAddresses 对应的端点也需要发布出来。默认为 false。
    PublishNotReadyAddresses bool `json:"publishNotReadyAddresses,omitempty" protobuf:"varint,13,opt,name=publishNotReadyAddresses"`

    // 包含 Session 亲和性相关配置
    SessionAffinityConfig *SessionAffinityConfig `json:"sessionAffinityConfig,omitempty" protobuf:"bytes,14,opt,name=sessionAffinityConfig"`
}
```

 我们在之前的 YAML 配置文件中，重点关注了 Ports 和 ServiceType 。其中前者定义了 Service 对外暴露的端口，以及对内映射到 Pod 的端口 

 其中 ServicePort 的定义如下 

```go
// $GOPATH/src/k8s.io/api/core/v1/types.go

// ServicePort 定义了 Service 暴露的端口。
type ServicePort struct {
    // Service 内部的端口名称，ServiceSpec 内部的端口名称必须唯一。
    // 这个名称和 EndpointPort 对象名称一致。
    // 如果这个 Service 只暴露了一个端口，那么这个值是可选的。
    Name string `json:"name,omitempty" protobuf:"bytes,1,opt,name=name"`

    // 该端口的 IP 协议。支持 TCP、UDP 和 SCTP。默认为 TCP 。
    Protocol Protocol `json:"protocol,omitempty" protobuf:"bytes,2,opt,name=protocol,casttype=Protocol"`

    // Service 暴露的端口
    Port int32 `json:"port" protobuf:"varint,3,opt,name=port"`

    // Service 所指向的 Pod 的端口，这个端口可以是一个整型的端口号，也可以是端口的名称。
    // 端口的范围为 1 ～ 65535。 如果这个值是一个字符串，那么这个值会被当作目标 Port 的容器端口名称。
    // 如果没有指定，则使用和上面的 Port 一样的值。
    // 当 ClusterIP = None 的时候，这个值会被忽略。
    TargetPort intstr.IntOrString `json:"targetPort,omitempty" protobuf:"bytes,4,opt,name=targetPort"`

    // 当 ServiceType 为 NodePort 或者 LoadBalancer 的时候，这个端口表示 Service 暴露在 Node 上面的端口。
    // 这个值通常是由系统分配的。如果是手动指定的话，当这个端口可用时会被分配给 Service，如果不可用那么 Service
    // 创建会失败。
    // 默认情况下，如果 Service 的 ServiceType 需要这样的一个端口的话，系统会自动分配一个。
    NodePort int32 `json:"nodePort,omitempty" protobuf:"varint,5,opt,name=nodePort"`
}
```

这个结构体的值和我们之前学习过的 YAML 配置中的值可以对应起来了。

好了，现在总结一下 Service 资源的作用：

1. Service 为一组 Pod 提供了集群内的访问接口，其它集群内的服务可以通过这个 Service 来访问到后端 Pod 中的服务。
2. Service 中支持定义一个或者多个端口映射，当有多个端口映射的时候，每个映射必须有一个唯一的名称。
3. Service 暴露的端口一般和后端 Pod 中容器暴露的端口保持一致，这样方便系统的维护。
4. Service 暴露的端口在向后端 Pod 中端口映射的时候，支持指定后端端口的端口号或者是端口名称。
5. Service 可以在 ServiceType 为 NodePort 的时候，在 Node 节点上也暴露一个端口，可以通过这个端口提供对集群外的访问。
6. Service 本身在集群内部通常是有一个 ClusterIP 的，这个 IP 由系统自动分配，供系统内其它服务访问。
7. Service 向后端 Pod 的请求路由是通过匹配 Service 中的 Selector 和 Pod 中定义的标签来完成的。

我们已经知道了 Service 可以向 Pod 进行路由请求，也知道了 Pod 的任何 Pod IP 变动、所在宿主 Node  的变动等信息都是通过和 Service 同名的 Endpoints 对象来动态维护的。比如后端 Pod 增加了一个实例，那么  Kubernetes 就会更新 Endpoints 对象，把这个 Pod 的实例信息加到 Endpoints 中，这样 Service  路由请求的时候就能动态发现这个 Pod 实例了。当然，Pod 的销毁或者是迁移遵循的流程是一样的。Kubernetes 就是通过这种方式来维护  Service 和 Pods 之间的关系的。

虽然这个 Endpoints 不需要我们手动维护，但是为了加深印象，还是看下 Endpoints 在 Go SDK 中的定义：

```go
// $GOPATH/src/k8s.io/api/core/v1/types.go

type Endpoints struct {
    metav1.TypeMeta `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

    // Subsets 表示所有端点的集合。Subset 中的地址是根据 Pod IP 来聚合的。一个地址对应多个端口。
    // 其中的端口有些是可用的，有些不是(因为来自其它的容器)。这样 EndpointSubset 里面对不同的端口就会有
    // Addresses 和 NotReadyAddresses。一个端点 Address 要么出现在 Addresses 里面，要么出现在
    // NotReadyAddresses 里面。
    // 这些 Addresses 和 Ports 一同维护着 Service 的路由状态。
    Subsets []EndpointSubset `json:"subsets,omitempty" protobuf:"bytes,2,rep,name=subsets"`
}
```

 我们再看下 EndpointSubset 的定义 

```go
继续深入 EndpointAddress 和 EndpointPoint 的定义// $GOPATH/src/k8s.io/api/core/v1/types.go

type EndpointSubset struct {
    // 端点可用的地址，即 Pod IP 和对应端口可用。
    Addresses []EndpointAddress `json:"addresses,omitempty" protobuf:"bytes,1,rep,name=addresses"`

    // 目前因为各种原因暂不可用的端点，比如 Pod 还没有完成启动，或者最近一次 Readiness 探针检查失败，或者最近一次 Liveness 检查失败
    NotReadyAddresses []EndpointAddress `json:"notReadyAddresses,omitempty" protobuf:"bytes,2,rep,name=notReadyAddresses"`

    // 在相关 Pod IP 上面可用的端口
    Ports []EndpointPort `json:"ports,omitempty" protobuf:"bytes,3,rep,name=ports"`
}
```

 继续深入 EndpointAddress 和 EndpointPoint 的定义 

```go
// $GOPATH/src/k8s.io/api/core/v1/types.go

type EndpointAddress struct {
    // 端点的 Pod IP
    IP string `json:"ip" protobuf:"bytes,1,opt,name=ip"`
    // 端点的 Hostname
    Hostname string `json:"hostname,omitempty" protobuf:"bytes,3,opt,name=hostname"`
    // 端点所在的 Node 的名称
    NodeName *string `json:"nodeName,omitempty" protobuf:"bytes,4,opt,name=nodeName"`
    // 指向提供该端点的 Pod 对象
    TargetRef *ObjectReference `json:"targetRef,omitempty" protobuf:"bytes,2,opt,name=targetRef"`
}
```

```go
// $GOPATH/src/k8s.io/api/core/v1/types.go

type EndpointPort struct {
    // 端点(服务)端口的名称
    Name string `json:"name,omitempty" protobuf:"bytes,1,opt,name=name"`
    // 端点(服务)端口号
    Port int32 `json:"port" protobuf:"varint,2,opt,name=port"`
    // 端点(服务)端口协议
    Protocol Protocol `json:"protocol,omitempty" protobuf:"bytes,3,opt,name=protocol,casttype=Protocol"`
}
```

 我们从这里看出 Endpoints 维护了 Pod IP 和 Pod 暴露的端口信息。另外还暴露了 Pod 所在 Node 信息，这样就为 Service 的请求路由提供了支持。 

#### 6.ingress源码

 Kubernetes 提供了一个名称叫做 Ingress 的资源来将集群外部的 HTTP 协议的流量路由给集群内部的  Service，从而将内部服务暴露给集群外部访问。Ingress 的中文意思本身就是入口。可以认为 Ingress 是 Kubernetes  七层(应用层)协议的入口。 

 从上图可以看出，Ingress 将请求路由给后端的 Service，然后由 Service 再转发给 Pod。之前我们已经了解到 Service 是通过 Selector 来选择需要转发的 Pod 的，而这里 Ingress 是通过指定 Service 的名称来进行路由的。另外  Ingress 还提供了负载均衡、HTTPS 证书支持和基于域名的虚拟主机功能。 

 我们先看下 Ingress 的组成部分 

```go
// $GOPATH/src/k8s.io/api/extensions/v1beta1/types.go

type Ingress struct {
    metav1.TypeMeta `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

    // Spec 是 Ingress 期望的状态
    Spec IngressSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

    // Status 是 Ingress 当前的状态
    Status IngressStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

 其中最主要研究下 Ingress 的期望状态 Spec 的定义： 

```go
// $GOPATH/src/k8s.io/api/extensions/v1beta1/types.go

type IngressSpec struct {
    // 定义一个默认的后端用来响应不匹配任何规则的请求。至少指定 backend 或者 rules。
    Backend *IngressBackend `json:"backend,omitempty" protobuf:"bytes,1,opt,name=backend"`

    // 定义 HTTPS 的证书。目前 Ingres 只支持一个 HTTPS 的端口，即 443。
    // 如果这个列表的成员指定了不同的 Host，那么将根据通过 SNI TLS 指定的域名把请求复用到相同的端口。
    // 上面的功能需要 Ingress Controller 支持 SNI。
    TLS []IngressTLS `json:"tls,omitempty" protobuf:"bytes,2,rep,name=tls"`

    // Ingress 中定义的一组基于 Host 的规则。如果没有指定任何规则，所有的流量都会发送给默认的后端。
    Rules []IngressRule `json:"rules,omitempty" protobuf:"bytes,3,rep,name=rules"`
}
```

 其中 IngressBackend 定义了一个后端的 Service 和 Service 暴露的端口。这个类型定义如下： 

```GO
// $GOPATH/src/k8s.io/api/extensions/v1beta1/types.go

type IngressBackend struct {
    // 定义了引用的 Service 的名称
    ServiceName string

    // 定义了引用的 Service 暴露的端口
    ServicePort intstr.IntOrString
}
```

 而 IngressTLS 则是定义了允许的 Host 列表和对应的保存 HTTPS 证书的 Secret 名称。这个类型定义如下： 

```go
// $GOPATH/src/k8s.io/api/extensions/v1beta1/types.go

type IngressTLS struct {
    // Hosts 指定了 HTTPS 证书中包含的 Host 列表。
    Hosts []string
    // SecretName 是保存 HTTPS 证书内容的 Secret 名称。
    // 我们需要把 HTTPS 的证书事先保存到 Secret 中，才能使用 HTTPS。
    SecretName string
}
```

 最后，最重要的是在常规情况下，请求转发到后端 Service 的定义。主要是转发规则和对应的 Service。这个类型定义如下： 

```go
// $GOPATH/src/k8s.io/api/extensions/v1beta1/types.go

type IngressRule struct {
    // Host 定义转发的请求对应的 Host，简单来讲就是从哪个域名请求服务的。
    Host string
    // IngressRuleValue 定义了上面的 Host 的路由规则，可以根据不同的请求路径
    // 请求路由到不同的后端 Service。
    IngressRuleValue
}
```

 我们看下基于 Path 的路由定义： 

```go
// $GOPATH/src/k8s.io/api/extensions/v1beta1/types.go

type IngressRuleValue struct {
    HTTP *HTTPIngressRuleValue
}

type HTTPIngressRuleValue struct {
    Paths []HTTPIngressPath
}

type HTTPIngressPath struct {
    // Path 定义了请求的路径
    Path string
    // Backend 定义了路由到的后端 Service
    Backend IngressBackend
}
```

 我们看下基于 Path 的路由定义： 

```go
// $GOPATH/src/k8s.io/api/extensions/v1beta1/types.go

type IngressRuleValue struct {
    HTTP *HTTPIngressRuleValue
}

type HTTPIngressRuleValue struct {
    Paths []HTTPIngressPath
}

type HTTPIngressPath struct {
    // Path 定义了请求的路径
    Path string
    // Backend 定义了路由到的后端 Service
    Backend IngressBackend
}
```

我们通过一层层查看资源的定义方式，最终看到了 Ingress 是在 HTTPIngresPath 中基于请求的 Path 转发到不同的 Backend 对应的 Service。

所以简单来讲，Ingress 的主要功能就是基于不同的 Host、不同的 Path，将请求路由到不同的后端 Service。

现在来讲解一下，外部的流量是如何通过这个 Service 根据我们指定的 Ingress 的规则路由到对应集群内部的 Service 中去。

首先，上面的 Ingress Nginx 的 Service 将流量转发给其根据 Selector 匹配的 Pod，这些 Pod 也就是 Ingress Nginx Controller 对应的 Pod：

 这些 Ingress Nginx Controller 的 Pod 会监听 Kubernetes API Server 的接口  /ingress，从这个接口获取集群中 Ingress 定义的内容和更新，然后根据这些 Ingress 规则来将流量转发到这些 Ingress  规则中匹配到的 Service 中。从而形成了一整条流量转发链： 

![](image\ingressOfpic.png)

#### 7.secret源码

secret struct

```go
// $GOPATH/src/k8s.io/api/core/v1/types.go

type Secret struct {
    metav1.TypeMeta `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

    // Data 字段包含 Secret 的数据，每个 Key 必须由数字，字符字符, -, _ 或者 . 组成。
    // 对应的值为 Secret 数据 base64 编码后的值，可以用来表示任意类型的数据(也可以是非字符串的二进制数据)。
    Data map[string][]byte `json:"data,omitempty" protobuf:"bytes,2,rep,name=data"`

    // StringData 用来存储字符串类型的数据。这个字段只用来方便写入。写入的数据会合并到上面的 Data 字段中。
    StringData map[string]string `json:"stringData,omitempty" protobuf:"bytes,4,rep,name=stringData"`

    // 预定义一些方便程序处理的 Secret 数据类型，比如 TLS、SSH Key、ServiceAccount 等。
    Type SecretType `json:"type,omitempty" protobuf:"bytes,3,opt,name=type,casttype=SecretType"`
}
```

从上面 Secret 的结构定义就可以看出 Secret 用来存储键值对数据，而且可以是任意类型的数据，无论是二进制还是字符串类型的数据都可以存储。如果是二进制的数据，则使用 base64 编码之后再存入 Data 中。

另外对于常见的一些类型的 Secret 数据，还额外定义了一些常用的 Secret 类型字符串，以及它们对应的需要填充的 Key 名称来方便程序处理。我们可以看看预定义了哪些类型的 Secret：

```go
// $GOPATH/src/k8s.io/api/core/v1/types.go

type SecretType string

const (
    // SecretTypeOpaque 默认的类型，表示任意数据
    SecretTypeOpaque SecretType = "Opaque"

    // SecretTypeServiceAccountToken 包含一个可以用来代表 ServiceAccount 的 Token，可以使用该 Token 来调用 API
    // 必填字段:
    // - Secret.Annotations["kubernetes.io/service-account.name"] - ServiceAccount 的名字
    // - Secret.Annotations["kubernetes.io/service-account.uid"] - SerivceAccout 的 UID
    // - Secret.Data["token"] - 用来代表 ServiceAccount 的 Token，可以使用该 Token 来调用 API
    SecretTypeServiceAccountToken SecretType = "kubernetes.io/service-account-token"

    // SecretTypeDockercfg 包含了一个 dockercfg 文件的内容，格式和 ~/.dockercfg 一样。
    // 必填字段:
    // - Secret.Data[".dockercfg"] - 一个序列化的 ~/.dockercfg 文件
    SecretTypeDockercfg SecretType = "kubernetes.io/dockercfg"


    // SecretTypeDockerConfigJson 包含了 dockercfg 文件的内容，格式和 ~/.docker/config.json 一样。
    // 必填字段:
    // - Secret.Data[".dockerconfigjson"] - a serialized ~/.docker/config.json file
    SecretTypeDockerConfigJson SecretType = "kubernetes.io/dockerconfigjson"


    // SecretTypeBasicAuth 包含了 Basic 鉴权所需要的用户名和密码。
    // 必填字段:
    // - Secret.Data["username"] - 用于鉴权的用户名
    // - Secret.Data["password"] - 用于鉴权的密码或令牌
    SecretTypeBasicAuth SecretType = "kubernetes.io/basic-auth"

    // SecretTypeSSHAuth 包含了 SSH 验证所需要的数据。
    // 必填字段:
    // - Secret.Data["ssh-privatekey"] - SSH 私钥
    SecretTypeSSHAuth SecretType = "kubernetes.io/ssh-auth"

    // SecretTypeTLS 包含了 HTTPS 的证书和私钥
    // 必填字段:
    // - Secret.Data["tls.key"] - 私钥内容
    //   Secret.Data["tls.crt"] - 证书内容
    SecretTypeTLS SecretType = "kubernetes.io/tls"

    // SecretTypeBootstrapToken 包含用于自启动所需的 Token。
    SecretTypeBootstrapToken SecretType = "bootstrap.kubernetes.io/token"
)
```

 上面这些以 SecretType 开头的常量就是预定义的常用的 Secret 数据类型。另外对于这些类型的 Secret  需要设置哪些键的值，也是预先定义为常量了，避免编程的时候手误写错。例如 SecretTypeServiceAccountToken 类型的  Secret 就定义好了如下的需要填充的键： 

```GO
// $GOPATH/src/k8s.io/api/core/v1/types.go

ServiceAccountNameKey = "kubernetes.io/service-account.name"
ServiceAccountUIDKey = "kubernetes.io/service-account.uid"
ServiceAccountTokenKey = "token"
ServiceAccountKubeconfigKey = "kubernetes.kubeconfig"
ServiceAccountRootCAKey = "ca.crt"
ServiceAccountNamespaceKey = "namespace"
```

 上面的这些字符串常量就是 SecretTypeServiceAccountToken 这个类型的 Secret 需要填充的键。 

 我们从上面了解到可以使用 Secret 来存储 HTTPS 的证书和私钥。其实就是填充 Secret 结构中的 Data 或者 StringData，需要填充的两个键如下： 

```GO
// $GOPATH/src/k8s.io/api/core/v1/types.go

TLSCertKey = "tls.crt"
TLSPrivateKeyKey = "tls.key"
```

 而这个 Secret 对应的预定义类型为 SecretTypeTLS 

#### 8.deployment源码

Deployment 提供了一种机制，方便用户通过自动的方式来管理 Pod 的生命周期，提供一种便捷的方式来解决应用的升级、回滚和扩容操作。

当我们期望对应用进行升级、回滚和扩容的时候，都会涉及到 Pod 的销毁和重建。在 Deployment 中可以描述我们期望 Pod  升级到的版本、回滚到的版本或者是期望扩容到的 Pod 的数量。然后 Kubernetes 中的 Replication Controller  会执行我们的预期，让生产环境中的 Pod 状态和预期的状态一致。

```go
// $GOPATH/src/k8s.io/api/apps/v1beta1/types.go

type Deployment struct {
    metav1.TypeMeta `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

    // Deployment 的期望状态
    Spec DeploymentSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

    // Deployment 的最新状态
    Status DeploymentStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

```go
// $GOPATH/src/k8s.io/api/apps/v1beta1/types.go

type DeploymentSpec struct {
    // 期望的 Pod 数量。
    Replicas *int32 `json:"replicas,omitempty" protobuf:"varint,1,opt,name=replicas"`

    // 选择 Pod 的标签。必须和 Template 定义的标签匹配。
    Selector *metav1.LabelSelector `json:"selector,omitempty" protobuf:"bytes,2,opt,name=selector"`

    // Pod 的创建模板，这里面包含了 Pod 的定义。所以可以自动创建 Pod。
    Template v1.PodTemplateSpec `json:"template" protobuf:"bytes,3,opt,name=template"`

    // Deployment 的执行策略，在升级多副本的服务时很有用。
    Strategy DeploymentStrategy `json:"strategy,omitempty" patchStrategy:"retainKeys" protobuf:"bytes,4,opt,name=strategy"`

    // 最短可用时间，即 Pod 创建完成之后多久即认为 Pod 可用。默认为 0，表示创建后即可用。
    MinReadySeconds int32 `json:"minReadySeconds,omitempty" protobuf:"varint,5,opt,name=minReadySeconds"`

    // 保留的旧的 ReplicaSets 的数量用于回滚
    RevisionHistoryLimit *int32 `json:"revisionHistoryLimit,omitempty" protobuf:"varint,6,opt,name=revisionHistoryLimit"`

    // 标记 Deployment 是否被暂停，暂停之后 Deployment Controller 不会再处理这个 Deployment。
    Paused bool `json:"paused,omitempty" protobuf:"varint,7,opt,name=paused"`

    // Deployment 被标记为失败之前的最大允许执行时间。Deployment Controller 会继续处理失败的 Deployment
    // 并且会把失败的原因写到 Deployment 的 Status 中。这个默认设置为 int32 类型的最大值（即 2147483647），
    // 表示没有截止时间。
    ProgressDeadlineSeconds *int32 `json:"progressDeadlineSeconds,omitempty" protobuf:"varint,9,opt,name=progressDeadlineSeconds"`
}
```

从上面的 Deployment 的 Spec 的定义中，我们可以看到 Deployment 可以把之前通过手动管理 Pod  的操作管理起来，然后通过 Replication Controller  来自动执行这些过程。由于整个过程中，不再需要人为介入，所以可以避免很多的工作量，同时减少人为操作的失误。

在 Deployment 的 Spec 定义中，有个配置项叫做 Strategy，这是一个在多副本（Pod）应用场景下很有用的升级或者回滚的配置项，我们可以从它的可选设置里面了解它的具体功能：

```go
// $GOPATH/src/k8s.io/api/apps/v1beta1/types.go

// 部署策略
type DeploymentStrategyType string

const (
    // 创建新的 Pod 之前，销毁所有旧的 Pod。
    RecreateDeploymentStrategyType DeploymentStrategyType = "Recreate"

    // 通过滚动升级方式逐步创建新的 Pod，同时销毁旧的 Pod。
    RollingUpdateDeploymentStrategyType DeploymentStrategyType = "RollingUpdate"
)

type DeploymentStrategy struct {
    // 部署的类型，可以是 Recreate 或者是 RollingUpdate。默认是 RollingUpdate
    Type DeploymentStrategyType

    // 滚动升级的配置参数，当上面的 Type 是 RollingUpdate 的时候生效
    RollingUpdate *RollingUpdateDeployment
}
```

 通过上面的配置，我们发现 Deployment 可以支持滚动升级的方式，这样可以在不中断服务的情况下，完成升级或者回滚过程。 

### 25.几个模糊的概念

1. pause容器的作用

kubernetes中的pause容器主要为每个业务容器提供以下功能

PID命名空间：Pod中的不同应用程序可以看到其他应用程序的进程ID

网络命名空间： Pod中的多个容器能够访问同一个IP和端口范围

IPC命名空间：Pod中的多个容器能够使用SystemV IPC或POSIX消息队列进行通信

UTS命名空间： Pod中的多个容器共享一个主机名；Volumes（共享存储卷）

2. kubernetes之pod健康检测

分类： LivenessProbe和ReadinessProbe

探针实现方式：ExecAction HTTPGetAction TCPSocketAction

3. kubernetes安全机制步骤

- Authentication：API Server认证管理
- Authorization：API Server授权管理
- Admission Control 准入控制
- Service Account
- Secret

### 26.kubernetes常见报错

#### 1.证书过期

- 问题描述

所有节点全部处于notready状态 kubelet输出日志

![](image\k8s\kubeletmistake.png)

~~~shell
#查看证书的有效日期 openssl
$ openssl x509 -in /etc/kubernetes/ssl/kubelet.crt -noout -dates
# cfssl查看方法
$ cfssl-certinfo -cert /etc/kubernetes/ssl/kubernetes.pem 

# kubelet证书默认有效期为一年 会申请自动续约 发起csr请求 所以在集群中可以通过命令查看和通过申请
$ kubectl get csr
$ kubectl certificate approve {csr_name}

#查看几个组件日志的命令
$ journalctl -u kube-scheduler
$ journalctl -xefu kubelet
$ journalctl -u kube-apiserver
$ journalctl -u kubelet |tail
$ journalctl -xe

$ kubectl logs --tail 200 -f kube-apiserver -n kube-system |more
$ kubectl logs --tail 200 -f podname -n jenkins
$ cat /var/log/messages
~~~

### 27.安装minikube

~~~shell
minikube start --vm-driver=none

#阿里云安装
minikube start --image-mirror-country cn \
    --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.7.3.iso \
    --registry-mirror=https://xxxxxx.mirror.aliyuncs.com \
    --driver=none
    
#troubleshooting：Kubernetes v1.18.0 requires conntrack to be installed in root's path
#解决： yum install -y conntrack


~~~

### 28.kubernetes operator

~~~go
//文章
//https://my.oschina.net/u/1464083/blog/3196723
//https://blog.csdn.net/boling_cavalry/article/details/88917818
~~~

### 29查看kubelet的证书有效时间

~~~shell
openssl x509 -in /etc/kubernetes/ssl/kubelet.crt -noout -dates
#解决方案
kubectl get csr
kubectl certificate approve
~~~

### 30.filebeat发生oom-kill

 https://www.jianshu.com/p/389702465461 

### 31.创建用户并配置kubeconfig

#### 1.预备知识

kubernetes中有两种用户：服务账号（serviceaccount）和普通意义的用户（user），sa是由k8s管理，User通常是外部管理。User通常是人使用，而sa是某个服务、资源、程序使用的

#### 2.用户验证方式

尽管k8s认知用户靠的只是用户的名字，但是只需要用户名字就能请求k8s的api显然是不合理的，所以依然需要验证用户的身份，在k8s中有三种验证方式：

- x509客户端证书

  客户端证书验证通过为api server指定 --client-ca-file=xxx 选项启用，api server通过此ca文件来验证api请求携带的客户端证书的有效性，一旦成功，api server就会将客户端subject里的CN属性作为此次请求的用户名

- 静态token文件

  通过指定 --token-auth-file=SOMEFILE 选项来开启bearer token验证方式， 引用的文件时一个包括token，用户名，用户id的csv文件

- 静态密码文件

  通过指定basic-auth-file=SOMEFILE 选项开启密码验证

#### 3.创建用户并绑定权限

本文使用的是minikube环境，ca.crt文件的目录保存在 /var/lib/minikube/certs 目录下

~~~shell
#创建文件夹
mkdir /root/gx
#首先为用户创建一个私钥
openssl genrsa -out gx.key 2048
#接着用此私钥创建一个csr（证书签名请求）文件，其中我们需要在subject里带上用户信息（CN为用户名，0为用户组）
openssl req -new -key gx.key -out gx.csr -subj "/CN=gx/O=MGM"
#通过集群的CA证书和之前创建的csr文件，来为用户颁发证书
openssl x509 -req -in gx.csr -CA /var/lib/minikube/certs/ca.crt -CAkey /var/lib/minikube/certs/ca.key -CAcreateserial -out gx.crt -days 365
~~~

至此，我们的用户就创建完成了，接下来开始配置kubeconfig

~~~shell
# 设置集群参数
export KUBE_APISERVER="https://47.107.238.89:8443"
kubectl config set-cluster kubernetes \
--certificate-authority=/var/lib/minikube/certs/ca.crt \
--embed-certs=true \
--server=${KUBE_APISERVER} \
--kubeconfig=gx.kubeconfig

# 设置客户端认证参数
kubectl config set-credentials gx \
--client-certificate=./gx.crt \
--client-key=./gx.key \
--embed-certs=true \
--kubeconfig=gx.kubeconfig

# 设置上下文参数
kubectl config set-context kubernetes \
--cluster=kubernetes \
--user=gx \
--kubeconfig=gx.kubeconfig

# 设置默认上下文
kubectl config use-context kubernetes --kubeconfig=gx.kubeconfig

# 给用户授权 rbac
kubectl create clusterrolebinding gx-cluster --clusterrole=cluster-admin --user=gx
~~~

