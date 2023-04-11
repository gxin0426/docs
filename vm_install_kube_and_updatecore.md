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
yum remove kernel-tools-libs.x86_64 kernel-tools.x86_64
yum -y --enablerepo=elrepo-kernel install kernel-ml.x86_64 kernel-ml-tools.x86_64
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
server ntp1.aliyun.com iburst
local stratum 10
allow
EOF
systemctl restart chronyd
systemctl enable chronyd

#node
yum install -y chrony
sed -i 's/^server/#&/' /etc/chrony.conf
cat >> /etc/chrony.conf  << EOF
server 192.168.8.204 iburst
EOF
systemctl restart chronyd && systemctl enable chronyd
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
  "registry-mirrors": ["https://h3blxdss.mirror.aliyuncs.com"],
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
#yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```
```bash
 kubeadm config images list 
```
```bash
kubeadm config images list --image-repository=registry.aliyuncs.com/google_containers

```
## init master

```shell
kubeadm init --image-repository=registry.aliyuncs.com/google_containers  --kubernetes-version=v1.23.16 --service-cidr=10.1.0.0/16 --pod-network-cidr=10.244.0.0/16
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
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
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

