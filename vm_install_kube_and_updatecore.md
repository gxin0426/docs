## update core

```bash
#配置阿里yum源命令
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo


#运行以下命令生成缓存

yum clean all
yum makecache
yum -y update

//load pubilc key install elrepo
rpm -import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

//install el repo meta
yum list available --disablerepo='*' --enablerepo=elrepo-kernel
//view available install eprepo
yum --disablerepo=\* --enablerepo=elrepo-kernel list kernel


#install kernel
yum remove kernel-tools-libs.x86_64 kernel-tools.x86_64
#yum -y --enablerepo=elrepo-kernel install kernel-ml.x86_64 kernel-ml-tools.x86_64
sudo yum --enablerepo=elrepo-kernel install kernel-ml kernel-ml-devel
```

## startup kernel sequence
```bash
#view all available kernel on the system
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
          
#set boot new kernel
grub2-set-default 0

#运行grub2-mkconfig命令来重新创建内核配置
grub2-mkconfig -o /boot/grub2/grub.cfg

reboot
```

## basic environment
```bash

systemctl stop firewalld && systemctl disable firewalld
sed -i '/^SELINUX=/c SELINUX=disabled' /etc/selinux/config
setenforce 0

swapoff -a
sed -i 's/^.*centos-swap/#&/g' /etc/fstab

cat << EOF >> /etc/hosts
master 192.168.31.127
kube 192.168.31.128
EOF
//


# 激活 br_netfilter 模块
modprobe br_netfilter
cat << EOF > /etc/modules-load.d/k8s.conf
br_netfilter
EOF

# 内核参数设置：开启IP转发，允许iptables对bridge的数据进行处理
cat << EOF > /etc/sysctl.d/k8s.conf 
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# 立即生效
sysctl --system


#time sync
#master node
yum install -y chrony
sed -i 's/^server/#&/' /etc/chrony.conf
cat >> /etc/chrony.conf << EOF
server ntp1.aliyun.com iburst
local stratum 10
allow
EOF
systemctl restart chronyd && systemctl enable chronyd

#node
yum install -y chrony
sed -i 's/^server/#&/' /etc/chrony.conf
cat >> /etc/chrony.conf  << EOF
server 172.16.101.11 iburst
EOF
systemctl restart chronyd && systemctl enable chronyd
```

```zsh
#找到最大的磁盘  挂载到/data目录
ln -s /data/kubelet /var/lib/kubelet
ln -s /data/docker /var/lib/docker
```

## install docker

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce-20.10
systemctl enable docker && systemctl start docker 
```
```bash
cat << EOF > /etc/docker/daemon.json
{
  "registry-mirrors": ["https://h3blxdss.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "insecure-registries": ["registry-tgq.harbor.com"]
}
EOF

 systemctl daemon-reload && systemctl restart docker
```

## install kubeadm 
```bash
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y --nogpgcheck kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6
yum install --nogpgcheck  --downloadonly --downloaddir=/opt/kubeadm kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6
```
```bash
#yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```
```bash
 kubeadm config images list 
 kubeadm config images pull  --image-repository=registry.aliyuncs.com/google_containers  --kubernetes-version=v1.23.6
```
```bash
kubeadm config images list --image-repository=registry.aliyuncs.com/google_containers

```
## init master

```shell
kubeadm init --image-repository=registry.aliyuncs.com/google_containers  --kubernetes-version=v1.23.6 --service-cidr=10.1.0.0/16 --pod-network-cidr=10.244.0.0/16
#--apiserver-cert-extra-sans 公网ip或者域名
```
```bash
# 要开始使用集群，您需要以常规用户身份运行以下命令
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 或者，如果您是root用户，则可以运行允许命令
export KUBECONFIG=/etc/kubernetes/admin.conf
```

```bash
#部署flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 注意修改ip pool 与 --pod-network-cidr 一致


# 部署calico
把calico.yaml里pod所在网段改成kubeadm init时选项--pod-network-cidr所指定的网段 
- name: CALICO_IPV4POOL_CIDR
value: "10.244.0.0/16"
# Disable file logging so `kubectl logs` works.
- name: CALICO_DISABLE_FILE_LOGGING
value: "true"

# deploy controller oeprator
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/tigera-operator.yaml

#deploy resource
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/custom-resources.yaml
```

#### flannel yaml

```yaml
---
kind: Namespace
apiVersion: v1
metadata:
  name: kube-flannel
  labels:
    k8s-app: flannel
    pod-security.kubernetes.io/enforce: privileged
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: flannel
  name: flannel
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
- apiGroups:
  - networking.k8s.io
  resources:
  - clustercidrs
  verbs:
  - list
  - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: flannel
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-flannel
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: flannel
  name: flannel
  namespace: kube-flannel
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-flannel
  labels:
    tier: node
    k8s-app: flannel
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-flannel
  labels:
    tier: node
    app: flannel
    k8s-app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      hostNetwork: true
      priorityClassName: system-node-critical
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni-plugin
        image: docker.io/flannel/flannel-cni-plugin:v1.1.2
       #image: docker.io/rancher/mirrored-flannelcni-flannel-cni-plugin:v1.1.2
        command:
        - cp
        args:
        - -f
        - /flannel
        - /opt/cni/bin/flannel
        volumeMounts:
        - name: cni-plugin
          mountPath: /opt/cni/bin
      - name: install-cni
        image: docker.io/flannel/flannel:v0.22.0
       #image: docker.io/rancher/mirrored-flannelcni-flannel:v0.22.0
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: docker.io/flannel/flannel:v0.22.0
       #image: docker.io/rancher/mirrored-flannelcni-flannel:v0.22.0
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
            add: ["NET_ADMIN", "NET_RAW"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: EVENT_QUEUE_DEPTH
          value: "5000"
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
        - name: xtables-lock
          mountPath: /run/xtables.lock
      volumes:
      - name: run
        hostPath:
          path: /run/flannel
      - name: cni-plugin
        hostPath:
          path: /opt/cni/bin
      - name: cni
        hostPath:
          path: /etc/cni/net.d
      - name: flannel-cfg
        configMap:
          name: kube-flannel-cfg
      - name: xtables-lock
        hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
```

#### calico yaml

```yaml
# This section includes base Calico installation configuration.
# For more information, see: https://projectcalico.docs.tigera.io/master/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    # Note: The ipPools section cannot be modified post-install.
    ipPools:
    - blockSize: 26
      cidr: 192.168.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()

---

# This section configures the Calico API server.
# For more information, see: https://projectcalico.docs.tigera.io/master/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}

```



## add node

```bash
#get token and ca
kubeadm token list | awk -F" " '{print $1}' |tail -n 1
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^ .* //'


kubeadm join 192.168.31.127:6443 --token <toekn> --discovery-token-ca-cert-hash sha256:<ca>
```

## GPU

### install nvidia-docker2

```shell
#Setup the repository and the GPG key:
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.repo | sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo
```

```shell
sudo yum clean expire-cache
sudo yum install -y nvidia-docker2
sudo systemctl restart docker
sudo docker run --rm --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi
```

### install nvidia-driver for centos7

```shell
#update version
sudo yum clean all
sudo yum update

#验证系统内核版本和安装开发包
sudo yum install -y gcc gcc-c++ kernel-devel-$(uname -r) kernel-headers-$(uname -r)

#验证gcc的版本 
gcc --version

#由于CUDA 11.3要求GCC的版本是6以上，下面是安装GCC7的脚本
sudo yum install centos-release-scl
sudo yum install devtoolset-7
# launch a new shell instance using the Software Collection scl tool:
scl enable devtoolset-7 bash
gcc --version

#检查当前驱动情况
sudo yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
sudo yum install nvidia-detect      # 安装nvida-detect
nvidia-detect -v                    # 检测能够升级到的驱动器版本
cat /proc/driver/nvidia/version     # 查看当前驱动版本

#卸载之前驱动。如果第一次安装，忽略
sudo /usr/bin/nvidia-uninstall

#屏蔽nouveau显卡驱动，把nvidiafb从屏蔽列表中移除
sudo rm -rf  disable-nouveau.conf
cat << EOF > disable-nouveau.conf
blacklist nouveau
options nouveau modeset=0

EOF
   
sudo chown root:root disable-nouveau.conf
sudo chmod 644 disable-nouveau.conf
sudo mv disable-nouveau.conf /etc/modprobe.d/
   
cat /etc/modprobe.d/disable-nouveau.conf
ll /etc/modprobe.d/disable-nouveau.conf


#重建 initramfs 镜像
sudo systemctl set-default multi-user.target    #设置运行级别为文本模式
sudo shutdown -r now
             

#下载驱动包 NVIDIA-Linux-x86_64-525.60.13.run (https://www.nvidia.com/Download/index.aspx?lang=en-us)
NVIDIA-Linux-x86_64-525.60.13.run
sudo rpm -ivh http://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-ml-devel-6.3.2-1.el7.elrepo.x86_64.rpm

lsmod | grep nouveau                            #查看nouveau是否已经禁用, 应该没有返回内容
nvidia_run=NVIDIA-Linux-x86_64-460.84.run
chmod 755 $nvidia_run
sudo  ./$nvidia_run
sudo systemctl set-default graphical.target     #设置运行级别回图形模式
sudo systemctl get-default
sudo shutdown -r now

#查看是否安装成功
cat /proc/driver/nvidia/version    
nvidia-smi
```

## 磁盘分区挂载

### parted

```
$ parted /dev/sdb mklabel gpt

$ parted /dev/sdb mkpart primary xfs 0% 100%

$ mkfs.xfs /dev/sdb1

$ mount /dev/sdb1 /data

$ df -hT /data

文件系统       类型  容量  已用  可用 已用% 挂载点

/dev/sdb1      xfs   100G   33M  100G    1% /data

$ vim /etc/fstab

/dev/sdb1   /data   xfs   defaults   0 0

# 实现开机自动挂载
```

### fdisk

```
fdisk /dev/sdb
n 进入分区状态
p 主分区
分区大小 默认即可
w 保存
mke2fs -t ext4 /dev/sdb1
mount /dev/sdb1 ~/newpath
修改  /etc/fstab 
UUID=c61117ca-9176-4d0b-be4d-1b0f434359a7  /newpath  ext4  defaults  0  0 
UUID 的获取可以通过这个命令 blkid /dev/sdb1
最后执行 mount -a 
```

### 节点集群间迁移

```shell
$ kubeadm reset
$ systemctl stop kubelet
$ systemctl stop docker
$ rm -rf /var/lib/cni/
$ rm -rf /var/lib/kubelet/*
$ rm -rf /etc/cni/
$ rm -rf /var/lib/etcd/*
$ ifconfig cni0 downdocker
$ ifconfig flannel.1 down
$ ifconfig docker0 down
ip link set cni0 down && ip link set flannel.1 down 
ip link delete cni0 && ip link delete flannel.1
systemctl restart docker && systemctl restart kubelet


rm -rf /root/.kube/config
然后执行命令加入到其他集群
```

### master重新加入集群

```shell
#我们有时候会有删除master节点，再重新加入master节点的需求，比如master机器改名。这里注意重新加入时，经常会出现etcd报错
[check-etcd] Checking that the etcd cluster is healthy error executionini phase check-etcd: etcd cluster is not healthy: failed to dial endpoint https://ip:2379 with maintenance client: context deadline exceeded

#这个时候，就需要去还没有停止的master节点里的etcd的pod里去，删除该老master节点对应的etcd信息
kubectl drain master01
kubectl delete node master01

#master01 执行
kubeadm reset
rm -rf /etc/kubernetes/manifests/

kubectl exec -it etcd-master02 sh
etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key member list

#找到对应的hash
etcdctl --endpoints 127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key member remove 12637f5ec2bd02b8



kubeadm init phase upload-certs --upload-certs  # 返回certificates-key


#执行加入命令
You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 172.16.101.211:9443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:ae579faf241a307a860d0a9e9ba1e308fe0e7a6006b90ece97eb42dfe9fc59b8 \
        --control-plane --certificate-key 79d731d185a93121e73899c10445f5fcaeac8d33155f5402c48bed5543f59e3b

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.101.211:9443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:ae579faf241a307a860d0a9e9ba1e308fe0e7a6006b90ece97eb42dfe9fc59b8


  kubeadm join 172.16.101.211:9443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:ae579faf241a307a860d0a9e9ba1e308fe0e7a6006b90ece97eb42dfe9fc59b8 \
        --control-plane --certificate-key 28d8cd85b5b90fe7603c599915521c20dc5ab5e6be28b52d904b44efb91eed19
```

### 离线安装教程

```
https://cloud.tencent.com/developer/article/2165251
```

离线安装NFS

```
https://blog.csdn.net/u013014761/article/details/100054241
https://qizhanming.com/blog/2018/08/08/how-to-install-nfs-on-centos-7
https://blog.csdn.net/u013014761/article/details/100054241
https://cloud.tencent.com/developer/article/2254970
```

### keepalived

```
https://www.cnblogs.com/rexcheny/p/10778567.html
```

### HA

```
https://www.cnblogs.com/wubolive/p/17140058.html#_label0_0
https://ost.51cto.com/posts/13131
https://www.linuxtechi.com/setup-highly-available-kubernetes-cluster-kubeadm/
https://developer.aliyun.com/article/1136864
https://hevodata.com/learn/kubernetes-high-availability/
```

### tecent article

```
https://tencentcloudcontainerteam.github.io/2019/08/12/troubleshooting-with-kubernetes-network/
https://tencentcloudcontainerteam.github.io/2019/12/15/no-route-to-host/
```

### kubeadm 离线安装包下载

```
yum install --downloadonly --downloaddir=/home/centos/k8s kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6
```

```
external IP 不兼容 ipvs 问题
https://blog.csdn.net/qq_41586875/article/details/124330823
```



### regenerate admin.conf

```bash
kubeadm init phase kubeconfig admin 
```

