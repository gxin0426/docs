## update core

```bash
//load pubilc key install elrepo
rpm -import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

//install el repo meta
yum --disablerepo=\* --enablerepo=elrepo-kernel repolist

//view available install eprepo
yum --disablerepo=\* --enablerepo=elrepo-kernel list kernel


#install kernel
yum -y --enablerepo=elrepo-kernel install kernel-ml.x86_64 kernel-ml-tools.x86_64
yum remove kernel-tools-libs.x86_64 kernel-tools.x86_64
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
sed -i 's/^ .*centos-swap/#&/g' /etc/fstab

cat << EOF >> /etc/hosts
master 192.168.31.127
kube 192.168.31.128
EOF


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
#server ntp1.aliyun.com iburst
local stratum 10
allow 192.168.31.0/24
EOF
systemctl restart chronyd
systemctl enable chronyd

#node
yum install -y chrony
sed -i 's/^server/#&/' /etc/chrony.conf
cat >> /etc/chrony.conf  << EOF
server 192.168.31.127 iburst
EOF
systemctl restart chronyd
systemctl enable chronyd
```

## install docker
```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce
systemctl enable docker && systemctl start docker 
```
```bash
cat << EOF > /etc/docker/daemon.json
{
  "registry-mirrors": ["https://xxxxx.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

systemctl daemon-reload
systemctl restart docker
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
```
```bash
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```
```bash
 kubeadm config images list 
```
```bash
kubeadm config images list --image-repository=registry.aliyuncs.com/google_containers

```
## init master

```
kubeadm init --image-repository=registry.aliyuncs.com/google_containers  --kubernetes-version=v1.23.6 --service-cidr=10.1.0.0/16 --pod-network-cidr=10.244.0.0/16

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
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

## add node

```bash
#get token and ca
kubeadm token list | awk -F" " '{print $1}' |tail -n 1
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^ .* //'


kubeadm join 192.168.31.127:6443 --token <toekn> --discovery-token-ca-cert-hash sha256:<ca>
```

