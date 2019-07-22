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

### 5.几个常用的关机命令

```shell
halt: 关机但不关闭电源（加p参数关闭电源）不加参数时调用shutdown命令
halt -p 相当于poweroff
halt -f 强制关机
halt -i 关机或重启前关闭所有网络接口


shutdown实际上是调用init 0, init 0会cleanup一些工作然后调用halt或者poweroff。其实主要区别是halt和poweroff，做没有acpi的系统上，halt只是关闭了os，电源还在工作，你得手动取按一下那个按钮，而poweroff会发送一个关闭电源的信号给acpi。但在现在的系统上，他们实际上都一样了

```

### 6.lsblk命令

```shell
lsblk可以列出所有可用块设备的信息。比如逻辑磁盘，而df -h 是查看文件系统级别的信息
```

### 7.快捷键

```shell
1.ctrl+a #光标回到行首
2.ctrl+e #光标回到行尾
3.ctrl+w #移除光标前的一个单词
4.Ctrl+k #删除光标处到行尾的字符
5.Ctrl+y #粘贴Ctrl+u，Ctrl+k，Ctrl+w删除的文本
6.Esc+b #移动到当前单词的开头
7.Esc+f #移动到当前单词的结尾
8.Ctrl+d #向行尾删除一个字符
```

