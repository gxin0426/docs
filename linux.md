### 1.rpm

```shell
rpm -qa |grep [包名]
rpm -e //删除包
```

### 2.查看系统版本

```shell
#查看系统发行版本
lsb_release -a
#查看linux内核版本
uname -a
```

### 3.centos中的tmpfs

~~~shell
/dev - 含有针对所有设备的设备文件的目录
/dev/shm - 包含共享内存分配
/run - 用于系统日志
/sys/fs/cgroup - 用于cgroup 一个针对特定进程限制 管理和审计资源利用的内核特性

#你可以使用systemctl命令在tmp目录启用tmpfs， 首先用下面的命令来检查这个特性是否可用：
systemctl is-enabled tmp.mount
#这会显示当先的状态，（如果未启用，）你可以使用下面的命令来启用它：
systemctl enable tmp.mount

#可以在/etc/fstab中添加下面这行，来手工在/tmp下挂载 tmpfs
tmpfs /tmp tmpfs size=512m 0 0





~~~

### 4.mount命令用法

~~~shell
挂载方法： mount DECE MOUNT_POINT
-t vsftype: 指定要挂载的设备上的文件系统类型
-r readonly : 只读挂载
-w read and write: 读写挂载
-L :以卷标指定挂载设备
-U : 以uuid指定要挂载的设备
-B --bind : 绑定目录到另一个目录上

#卸载命令umount
umount DEVICE
umount MOUNT_POINT
#重新挂载命令
mount -o remount /dev/shm

~~~

