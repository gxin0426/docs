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



