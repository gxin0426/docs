## centos7 NFS离线部署

### 准备工作

```shell
#升级内核（选做）
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


#view all available kernel on the system
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
          
#set boot new kernel
grub2-set-default 0

#运行grub2-mkconfig命令来重新创建内核配置
grub2-mkconfig -o /boot/grub2/grub.cfg

reboot

#关闭防火墙和selinux
systemctl stop firewalld && systemctl disable firewalld
sed -i '/^SELINUX=/c SELINUX=disabled' /etc/selinux/config
setenforce 0
```

### 安装rpm包(所有节点)

```shell
tar -zxvf nfsrpm.tar.gz
cd nfs
rpm -ivh *.rpm --force --nodeps
```

### nfs server配置

```shell
# 编辑配置文件
vi /etc/exports

# 修改配置文件，增加下面这一行数据，指定的ip地址为客户端的地址
# /data2/k8s 代表nfs server本地存储目录
/data2/k8s *(rw,no_root_squash,async)

# 加载配置文件
exportfs -arv
systemctl enable rpcbind.service 
systemctl enable nfs-server.service
systemctl start rpcbind.service
systemctl start nfs-server.service

#验证
[root@nfs ~]# showmount -e localhost
Export list for localhost:
/data2/k8s                     *
```

### nfs client配置

```shell
#查看nfs server 信息

[root@node02 ~]# showmount -e 172.16.101.13
Export list for 172.16.101.13:
/data2/k8s      *

#添加一行
vim /etc/fstab 
172.16.101.13:/data2/k8s   /data/k8s   nfs   defaults  0  0
#保存退出

mount -a 
#验证
df -h /data/k8s

#输出 表示挂载成功
[root@node02 ~]# df -h /data/k8s
Filesystem                Size  Used Avail Use% Mounted on
172.16.101.13:/data2/k8s  3.5T  626M  3.5T   1% /data/k8s

```



### 企业级挂载

```shell
#rc.local 
/etc/init.d/rpcbind start 
/bin/mount -t nfs 10.0.0.0:/data /data
```

```shell
#client 挂载时
 mount -o 选项 
 noatime （提升性能，不更新文件系统上的inode访问时间）
 nodiratime  不更新文件系统的directory inode访问时间， 高并发环境
 
 当文件系统变为只读时 可以使用下面的命令
 mount -o remount,rw 
 #强制卸载
 umount -lf /mnt
 
```

### linux 性能优化

```shell
cat /proc/sys/net/core/rmem_default 
cat /proc/sys/net/core/rmem_max
cat >>/etc/sysctl.conf<<EOF
net.core.wmem_default
net.core.rmem_default
net.core.wmem_max
net.core.rmem_max
EOF
systcl -p
```

### 生产场景NFS共享存储优化

```bash
# 硬件： sas/ssd磁盘， 买多块， raid/raid10。 网卡好
# all_squash , async
# rsize wsize noatime nodirtime nosuid noexec soft (hard, intr) 
```

- **showmount -e -a -d**

- **exportfs == reload   -rv**  
- **rpcinfo -p localhost**

```
program not register 
restart nfs即可
```

常用目录

```
/etc/fstb
/etc/exports
/var/lib/nfs/etab
/proc/mounts
```

### 如果不关闭防火墙，使用下面命令添加规则

```shell
firewall-cmd --permanent --add-service mounted
firewall-cmd --permanent --add-service rpc-bind
firewall-cmd --permanent --add-service nfs
firewall-cmd --reload
```

### 性能优化

```
https://www.cnblogs.com/yanling-coder/p/13028552.html
```

离线安装

```
yum -y install nfs-utils --downloadonly --downloaddir /home/nfs
cd /home/nfs

# 安装nfs
rpm -ivh *.rpm --force --nodeps
```







