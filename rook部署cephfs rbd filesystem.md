# rook部署cephfs rbd filesystem

## 环境信息

| kubernetes版本 | 系统版本  | 内核                       | rook版本   | docker版本 |
| -------------- | --------- | -------------------------- | ---------- | ---------- |
| 1.12           | Centos7.6 | 3.10.0-957.21.3.el7.x86_64 | release0.9 | 1.13.1     |

使用rook分别部署ceph rbd、rgw、cephfs

拉去代码:

`git clone -b release-0.9 https://github.com/rook/rook.git`

进入到目录：

`rook/cluster/examples/kubernetes/ceph`

可以看到如下一些文件	

```shell
-rwxr-x--- 1 root root  8132 Jul 20 17:09 cluster.yaml
-rw-r----- 1 root root   363 Jul 20 15:38 dashboard-external-https.yaml
-rw-r----- 1 root root   362 Jul 20 15:38 dashboard-external-http.yaml
-rw-r----- 1 root root  1487 Jul 20 15:38 ec-filesystem.yaml
-rw-r----- 1 root root  1538 Jul 20 15:38 ec-storageclass.yaml
-rw-r----- 1 root root  1375 Jul 20 15:38 filesystem.yaml
-rw-r----- 1 root root  1923 Jul 20 15:38 kube-registry.yaml
drwxr-x--- 2 root root   104 Jul 20 15:38 monitoring
-rw-r----- 1 root root   160 Jul 20 15:38 object-user.yaml
-rw-r----- 1 root root  1813 Jul 20 15:38 object.yaml
-rwxr-x--- 1 root root 12690 Jul 20 15:38 operator.yaml
-rw-r----- 1 root root   742 Jul 20 15:38 pool.yaml
-rw-r----- 1 root root   410 Jul 20 15:38 rgw-external.yaml
-rw-r----- 1 root root  1216 Jul 20 15:38 scc.yaml
-rw-r----- 1 root root   991 Jul 20 16:35 storageclass.yaml
-rw-r----- 1 root root  1544 Jul 20 15:38 toolbox.yaml
-rw-r----- 1 root root  6492 Jul 20 15:38 upgrade-from-v0.8-create.yaml
-rw-r----- 1 root root   874 Jul 20 15:38 upgrade-from-v0.8-replace.yaml
```

## 部署组件

## 部署operator

环境和代码准备好之后，首先要部署rook-ceph-operator。包括crd以及角色等信息。

```shell
$ kubectl apply -f operator.yml
$ kubectl get crd
NAME                                    CREATED AT
cephblockpools.ceph.rook.io             2021-07-20T09:07:52Z
cephclusters.ceph.rook.io               2021-07-20T09:07:52Z
cephfilesystems.ceph.rook.io            2021-07-20T09:07:52Z
cephobjectstores.ceph.rook.io           2021-07-20T09:07:52Z
cephobjectstoreusers.ceph.rook.io       2021-07-20T09:07:52Z
podgroups.scheduling.incubator.k8s.io   2021-07-20T08:19:36Z
podgroups.scheduling.sigs.dev           2021-07-20T08:19:36Z
queues.scheduling.incubator.k8s.io      2021-07-20T08:19:36Z
queues.scheduling.sigs.dev              2021-07-20T08:19:36Z
volumes.rook.io                         2021-07-20T09:07:52Z


$ kubectl get pods -n rook-ceph-system
NAME                                  READY   STATUS    RESTARTS   AGE
rook-ceph-agent-fkg9z                 1/1     Running   0          3h20m
rook-ceph-agent-s6k7z                 1/1     Running   0          3h20m
rook-ceph-agent-zwb8n                 1/1     Running   0          3h20m
rook-ceph-operator-7dffb4dcb5-p7wg6   1/1     Running   0          3h20m
rook-discover-4ddjf                   1/1     Running   0          3h20m
rook-discover-9wdkn                   1/1     Running   0          3h20m
rook-discover-sfm4w                   1/1     Running   0          3h20m
```

## 部署ceph cluster 

接下来部署ceph集群。可以通过修改`cluster.yml`文件来配置符合自己环境的集群，因为是测试环境所以用目录创建osd。

```yaml
$ cat cluster.yml
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  cephVersion:
    image: ceph/ceph:v13.2.4-20190109
    allowUnsupported: false

  # In Minikube, the '/data' directory is configured to persist across reboots. Use "/data/rook" in Minikube environment.
  #目录可以更改为自己的目录
  dataDirHostPath: /export/rook/config
  # set the amount of mons to be started
  mon:
    count: 3
    allowMultiplePerNode: true
  # enable the ceph dashboard for viewing cluster status
  dashboard:
    enabled: true
    # serve the dashboard under a subpath (useful when you are accessing the dashboard via a reverse proxy)
    # urlPrefix: /ceph-dashboard
    # serve the dashboard at the given port.
    port: 7000
    # serve the dashboard using SSL
    #dashboard 不使用https
    ssl: false
  network:
    # toggle to use hostNetwork
    hostNetwork: true
  rbdMirroring:
    # The number of daemons that will perform the rbd mirroring.
  storage: # cluster level storage configuration and selection
    useAllNodes: true
    useAllDevices: false
    deviceFilter:
    location:
		#因为是测试环境所以用目录创建osd
    directories:
    - path: /export/rook/data
```



```shell
$ kubectl apply -f cluster.yml
$ kubectl get pods -n rook-ceph-system

[root@A01-R20-I103-9-5YP8GM2 ~]# kubectl get pods -n rook-ceph
NAME                                         READY   STATUS      RESTARTS   AGE
rook-ceph-mds-myfs-a-568dbd6cd9-xtppw        1/1     Running     0          3h59m
rook-ceph-mds-myfs-b-766f5c9545-s4dz8        1/1     Running     0          3h59m
rook-ceph-mgr-a-57f6f88497-hxw26             1/1     Running     0          4h12m
rook-ceph-mon-a-77bb4dd6f-wjh54              1/1     Running     0          4h13m
rook-ceph-mon-b-c58b98f48-d4v6r              1/1     Running     0          4h12m
rook-ceph-mon-c-994fd745b-xvffj              1/1     Running     0          4h12m
rook-ceph-osd-0-67d98d8796-z7rv9             1/1     Running     0          4h12m
rook-ceph-osd-1-7fd8f95b64-n6mlq             1/1     Running     0          4h12m
rook-ceph-osd-2-79d756fb8d-75ztc             1/1     Running     0          4h12m
rook-ceph-osd-prepare-10.196.100.134-rddhj   0/2     Completed   0          4h12m
rook-ceph-osd-prepare-10.196.100.196-z2kh6   0/2     Completed   0          4h12m
rook-ceph-osd-prepare-10.196.103.9-wq4bw     0/2     Completed   0          4h12m
rook-ceph-rbd-mirror-a-c4966c96b-t7sjx       1/1     Running     0          4h12m
rook-ceph-rbd-mirror-b-5cd69d68cb-2gq28      1/1     Running     0          4h12m
rook-ceph-rbd-mirror-c-6bbd88b5f8-k9wmv      1/1     Running     0          4h12m
rook-ceph-rgw-my-store-557db5b975-4g7zd      1/1     Running     0          4h1m
rook-ceph-tools-cb5655595-j94nq              1/1     Running     0          3h59m
```

## 部署dashboard nodeport

在cluser.yml中配置`ssl: false`不使用ssl，随意这里执行`dashboard-external-http.yaml`就可以将dashboard服务暴露到集群外部

```shell
$ kubectl apply -f dashboard-external-http.yaml

$ kubectl get svc -n rook-ceph |grep mgr-dashboard
rook-ceph-mgr-dashboard                 ClusterIP   10.254.201.138   <none>        7000/TCP         4h17m
rook-ceph-mgr-dashboard-external-http   NodePort    10.254.61.101    <none>        7000:32356/TCP   4h14m

#浏览器访问masterip:32356 就可以访问dashboard

#用户名为admin 密码用下面的命令
$ kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath='{.data.password}'  |  base64 --decode
UvJVjtZqEK
```

## 部署toolbox用来执行ceph命令

```shell
$ kubectl apply -f toolbox.yaml
$ kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') bash

$ kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') bash
bash: warning: setlocale: LC_CTYPE: cannot change locale (en_US.UTF-8): No such file or directory
bash: warning: setlocale: LC_COLLATE: cannot change locale (en_US.UTF-8): No such file or directory
bash: warning: setlocale: LC_MESSAGES: cannot change locale (en_US.UTF-8): No such file or directory
bash: warning: setlocale: LC_NUMERIC: cannot change locale (en_US.UTF-8): No such file or directory
bash: warning: setlocale: LC_TIME: cannot change locale (en_US.UTF-8): No such file or directory

#执行ceph命令查看集群状态

$ ceph status
$ ceph df
$ rados df
$ ceph osd status
$ ceph fs ls
$ ceph mds stat
$ ceph osd status

#重启dashboard
$ ceph mgr module disable dashboard
$ ceph mgr module enable dashboard

#查看dashboard地址
$ ceph mgr services
```

## 部署block storage

执行命令`kubectl create -f storageclass.yaml`创建一个rbd pool和storageclass。`replicated.size: 3`需要至少三个osd在三个不同的node上。

```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 3
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-block
provisioner: ceph.rook.io/block
parameters:
  blockPool: replicapool
  # The value of "clusterNamespace" MUST be the same as the one in which your rook cluster exist
  clusterNamespace: rook-ceph
  # Specify the filesystem type of the volume. If not specified, it will use `ext4`.
  fstype: xfs
# Optional, default reclaimPolicy is "Delete". Other options are: "Retain", "Recycle" as documented in https://kubernetes.io/docs/concepts/storage/storage-classes/
reclaimPolicy: Retain

```

在目录`rook/cluster/examples/kubernetes`下执行`kubectl create-f mysql.yml`验证pv是否创建成功

```shell
$ kubectl get pvc
NAME             STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
mysql-pv-claim   Bound     pvc-95402dbc-efc0-11e6-bc9a-0cc47a3459ee   20Gi       RWO           1m
wp-pv-claim      Bound     pvc-39e43169-efc1-11e6-bc9a-0cc47a3459ee   20Gi       RWO           1m
```

如果需要使用纠删码块存储，请看[文档](https://rook.github.io/docs/rook/v0.9/ceph-block.html#advanced-example-erasure-coded-block-storage)。

## 部署sharefilesystem

``````shell
$ kubectl apply -f filesystem.yml
$ kubectl -n rook-ceph get pod -l app=rook-ceph-mds
NAME                                    READY   STATUS    RESTARTS   AGE
rook-ceph-mds-myfs-a-568dbd6cd9-xtppw   1/1     Running   0          2d
rook-ceph-mds-myfs-b-766f5c9545-s4dz8   1/1     Running   0          2d

$ ceph status
  cluster:
    id:     4a82dd81-3405-4aee-976b-036e5e0cc757
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum a,b,c
    mgr: a(active)
    mds: myfs-1/1/1 up  {0=myfs-a=up:active}, 1 up:standby-replay
    osd: 3 osds: 3 up, 3 in
    rgw: 1 daemon active
``````

测试file system是否是否可用

```shell
$ kubectl apply -f kube-registry.yml
$ kubectl get po -n kube-system | grep kube-registry
kube-registry-v0-67hf4                  1/1     Running   0          3m32s
kube-registry-v0-p8kjc                  1/1     Running   0          3m32s
kube-registry-v0-v6tbk                  1/1     Running   0          3m32s
```

### ceph用户隔离

##### [参考文章](https://www.yisu.com/zixun/14781.html)

#### 1. 创建pool添加到cephfs

```shell
$ ceph osd pool create cephfs-metadata 64 64
$ ceph osd pool create cephfs-data 256 256
$ ceph fs add_data_pool myfs ceph-data

#创建一个 CephFS, 名字为 cephfs:需要指定两个创建的pool的名字
$ ceph fs new cephfs cephfs-metadata cephfs-data
```

#### 2. 创建具体目录使用权限的用户

```shell
#创建用户（此用户只能rw /mnt/zdk/zhoudekai目录下的文件）
$ ceph auth get-or-create client.zhoudekai mon 'allow r' mds 'allow r, allow rw path=/zhoudekai' osd 'allow rw pool=myfs-data0, allow rw pool=myfs-metadata'
#验证key是否生效
$ ceph auth get client.zhoudekai
#执行下面的命令，本地只能操作/mnt/zdk/zhoudekai/目录下创建文件等操作
$ sudo mount -t ceph monip:6790:/ /mnt/zdk -o name=zhoudekai,secret=AQACiPpg5+YRNxAAuDI0L32GVmH0RiJGwDAiYg==
$ ceph auth list
client.zhoudekai
	key: AQACiPpg5+YRNxAAuDI0L32GVmH0RiJGwDAiYg==
	caps: [mds] allow r, allow rw path=/zdk
	caps: [mon] allow r
	caps: [osd] allow rw pool=myfs-data0, allow rw pool=myfs-metadata
```

#### 3. 其他命令

```shell
# 从cephfs中删除pool
$ ceph fs rm_data_pool myfs sns_data
#删除 pool
$ ceph osd pool rm sns_data sns_data --yes-i-ready-ready-mean-it
#测试目录权限并创建大文件
$ sudo dd if=/dev/zero of=test1 bs=1M count=1000
#删除用户
$ ceph auth del {TYPE}.{ID}
#查看用户密钥
$ ceph auth print-key {TYPE}.{ID}
```



## 部署object storage

### 部署

```shell
# 创建 对象存储
$ kubectl create -f object.yaml

# 验证rgw pod正常运行
$ kubectl -n rook-ceph get pod -l app=rook-ceph-rgw

# 创建对象存储user
$ kubectl create -f object-user.yaml

# 获取 accesskey secretkey
$ kubectl -n rook-ceph get secret rook-ceph-object-user-my-store-my-user -o yaml | grep AccessKey | awk '{print $2}' | base64 --decode
$ kubectl -n rook-ceph get secret rook-ceph-object-user-my-store-my-user -o yaml | grep SecretKey | awk '{print $2}' | base64 --decode

#部署rgw nodeport
$ kubectl apply -f rgw-external.yaml
$ kubectl -n rook-ceph get service rook-ceph-rgw-my-store rook-ceph-rgw-my-store-external
NAME                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
rook-ceph-rgw-my-store            ClusterIP   None           <none>        80/TCP         3d1h
NAME                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
rook-ceph-rgw-my-store-external   NodePort    10.254.251.5   <none>        80:31301/TCP   3d1h
#通过nodeport 31301 可以在集群外与s3交互
```

### 验证

```python
import boto
import boto.s3.connection
access_key = '2G8OXZ5K09ENDQGSEMHV'
secret_key = 'yKCTo6aTgXnoESx7IttnfVv6wG9BOnIEZZMHGL41'
conn = boto.connect_s3(
    aws_access_key_id = access_key,
    aws_secret_access_key = secret_key,
    host = '192.168.103.9', port=31301,
    is_secure=False,
    calling_format = boto.s3.connection.OrdinaryCallingFormat(),
)
bucket = conn.create_bucket('zhoudekai-bucket-test')
for bucket in conn.get_all_buckets():
        print "{name}\t{created}".format(
                name = bucket.name,
                created = bucket.creation_date,
)
```

```shell
#执行代码
$ python createbucket.py
my-s3-bucket2	2021-07-20T10:01:06.717Z
test	2021-07-21T02:32:53.988Z
zhoudekai-bucket-test	2021-07-23T10:41:18.478Z
```

