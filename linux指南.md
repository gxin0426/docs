### 1.rpm

```shell
rpm -qa |grep [包名]
rpm -e //删除包

－ivh：安装显示安装进度--install--verbose--hash
－Uvh：升级软件包--Update；
－qpl：列出RPM软件包内的文件信息[Query Package list]；
－qpi：列出RPM软件包的描述信息[Query Package install package(s)]；
－qf：查找指定文件属于哪个RPM软件包[Query File]；
－Va：校验所有的RPM软件包，查找丢失的文件[View Lost]；
－e：删除包

#参考文章
https://blog.csdn.net/zhaoyue007101/article/details/8485186
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

### 8. snat dnat MASQUERADE

https://www.cnblogs.com/Dicky-Zhang/p/5934657.html

### 9.进程信息区统计信息区域各列含义

1. PID ---进程id
2. PPID ---父进程id
3. RUSER --- REAL USER NAME
4. UID --- 进程所有者的用户id
5. USER ---进程所有者的用户名
6. GROUP ---进程所有者的组名
7. TTY --- 启动进程的终端名。不是从终端启动的进程则显示为 ?
8. NI --- nice值。负值表示高优先级，正值表示低优先级
9. P --- 最后使用的CPU，仅在多CPU环境下有意义
10. %CPU ---  上次更新到现在的CPU时间占用百分比
11. %MEM --- 进程使用的物理内存百分比
12. TIME --- 进程使用的CPU时间总计，单位秒
13. VIRT --- 进程使用的虚拟内存总量 单位kb。VIRT=SWAP+RES

### 10.stress 工具

~~~shell
-v 显示版本号
-q 不显示运行信息
-n 显示已完成的指令情况
-t --timeout N 指定运行N秒后结束
--backup N 指定N微秒后运行
-c 产生n个进程 每个进程都反复不停的计算随机数的平方根
-i 产生n个进程 每个进程反复调用sync()，sync()用于将内存上的内容写到硬盘上
-m --vm n 产生n个进程,每个进程不断调用内存分配malloc和内存释放free函数
--vm-bytes B 指定malloc时内存的字节数 （默认256MB）
--vm-hang N 指定在free钱的秒数
-d --hadd n 产生n个执行write和unlink函数的进程
-hadd-bytes B 指定写的字节数
--hadd-noclean 不unlink
--vm-hang 表示malloc分配的内存多少时间后在free()释放掉
时间单位可以为秒s，分m，小时h，天d，年y，文件大小单位可以为K，M，G

-d forks
--hdd forks 产生多个执行write()函数的进程
--hdd-bytes bytes 指定写的Bytes数，默认是1GB
--hdd-noclean 不要将写入随机ASCII数据的文件Unlink
~~~

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
~~~

### 11.端口查询

~~~shell
#netstat
-t 指明tcp -u 指明udp -l 仅显示监听套接字 -p 显示进程标识符和程序名称 -n 不进行DNS轮询，显示ip
netstat -ntlp 查看所有tcp端口
netstat -tunlp | grep 80 #查看所有80端口使用情况
#lsof
lsof -i:80 #查看80占用
lsof ab.txt #显示开启文件ab.txt的进程
lsof -c -p 1234 #列出进程号为1234的进程所有打开的文件
lsof +d /usr/local #显示目录下被进程开启的文件
#一般使用方法
$netstat -tunlp | grep 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      27314/nginx: master 
tcp6       0      0 :::80                   :::*                    LISTEN      27314/nginx: master 
udp        0      0 0.0.0.0:68              0.0.0.0:*                           2808/dhclient       
udp6       0      0 fe80::5054:ff:fe02::123 :::*                                2501/ntpd 
$ps -aux | grep 27314
root     27314  0.0  0.1 125108  2116 ?        Ss   08:58   0:00 nginx: master process /usr/sbin/nginx
root     28492  0.0  0.0 112708   980 pts/0    R+   09:17   0:00 grep --color=auto 27314

~~~

### 12.df du free

~~~shell
du -h: 显示每个文件和目录的磁盘使用空间~~~文件的大小
df -h：显示磁盘分区上可以使用的磁盘空间 #-a    #查看全部文件系统，单位默认KB   -h  使用-h选项以KB、MB、GB的单位来显示，可读性高~~~（最常用）
free -h：可以显示Linux系统中空闲的、已用的物理内存及swap内存,及被内核使用的buffer
~~~

## 13.ipvs iptables

1. 三种工作模式

- nat
- tun
- dr