### -1.改变目录所有者和权限

chown -R gaoxin:gaoxin /opt

chmod 777 /opt

**chgrp 用户名  文件名 -R**

**chown 用户名  文件名 -R**

### 0.配置阿里yum源

~~~shell
#配置阿里yum源命令
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

下载repo 到 /etc/yum.repos.d/
epel(RHEL7)
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

#运行以下命令生成缓存
yum clean all
yum makecache
~~~

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
cat /etc/redhat-release
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

### 4.mount命令用法和问题

#### 1.mount用法

~~~shell
挂载方法： mount Device MOUNT_POINT
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

#当加入一块新设备时 1.要对设备进行格式化（制作文件系统） 2. 将其挂载到指定目录
#格式化
mkfs -t ext4 /dev/sdb
#挂载
mount /dev/sdb /xxx/


#进阶版
$ mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
#输出信息的格式和含义
fs_spec on fs_file type fs_vfstype (fs_mntopts)
fs_spec:挂载的块设备或者远程文件系统
fs_file:文件系统的挂载点
fs_vfstype:文件系统的类型
fs_mntopts:与文件系统的相关选项
第一行的含义:挂载的设备是sysfs 挂载点是/sys 文件系统的类型是sysfs 括号中rw代表可读写的方式挂载文件系统 noexec表示不能再该文件系统上直接运行程序
~~~

- overlay

~~~shell
mkdir layer1 layer2
mkdir ./rootfs/{merged,diff,work} -p
mount -t overlay overlay -o lowerdir=./layer1:./layer2,upperdir=./rootfs/diff,workdir=./rootfs/work ./rootfs/merged

merged:挂载点
diff: upper
work: work
~~~

#### 2.遇到的bug

~~~shell
#问题描述
#当创建一个新的进程并且挂载proc时，退出后在终端执行命令 mount 会出现下面的错误
mount: failed to read mtab: No such file or directory
#解决办法 执行下边的命令 就可以在终端执行mount命令了
mount -t proc proc /proc
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
--vm-hang N 指定在free前的秒数
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

### 12.内网安装Mariadb

~~~shell
$ yum install mariadb-server mariadb
$ systemctl start mariadb
$ systemctl enable mariadb
#修改密码
$ mysql
$ use mysql;
$ update user set password=password("Gree!2018") where user ='root';
$ flush privileges;
#设置白名单
$ grant all privileges on *.* to 'username'@'host' identified by 'Gree!2018' with grant option;
$ flush privileges;
~~~

### 13.安装mysql

~~~shell
$ wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
$ yum localinstall mysql57-community-release-el7-8.noarch.rpm
$ yum repolist enabled | grep "mysql.*-community.*"
$ yum install mysql-community-server
# 找到初始密码
$ vi /var/log/mysqld.log

#查找密码位置
$ /temporary password

#当找不到密码时，编辑这个文件 
$ vi /etc/my.cnf 
#添加一行代码，然后重启mysql 然后输入mysql直接进入  
skip-grant-tables 

#设置密码
$ set password for 'root'@'localhost'=password('password!'); 
# 允许远程连接 
$ grant all on *.* to root@'%' identified by 'password' with grant option;  
# 如果需要设置简单密码 
$ set global validate_password_policy=0; 
$ set global validate_password_length=1; 
$ set global validate_password_mixed_case_count=2;  
#增加白名单
mysql> use mysql 
mysql> GRANT ALL ON *.* to root@'172.28.171.176' IDENTIFIED BY 'your-root-password';   
mysql> FLUSH PRIVILEGES;  
 
#完全卸载mysql 
$ yum remove mysql 
$ find / -name mysql 
rm -rf /usr/lib64/mysql 
rm -rf /usr/share/mysql 
rm -rf /usr/bin/mysql
rm -rf /etc/logrotate.d/mysql
rm -rf /var/lib/mysql
rm -rf /var/lib/mysql/mysql
rm –rf /usr/my.cnf
rm -rf /root/.mysql_sercret  
rm -rf /var/logs/mysql
~~~

### 14.df du free

~~~shell
du -h: 显示每个文件和目录的磁盘使用空间~~~文件的大小 
du -h --max-depth 1
df -h：显示磁盘分区上可以使用的磁盘空间 #-a    #查看全部文件系统，单位默认KB   -h  使用-h选项以KB、MB、GB的单位来显示，可读性高~~~（最常用）
df -T：查看磁盘格式
free -h：可以显示Linux系统中空闲的、已用的物理内存及swap内存,及被内核使用的buffer

~~~

###15.ipvs iptables

1. 三种工作模式

- nat
- tun
- dr

### 16.添加环境变量

~~~shell
$ vi /etc/profile
unset i
unset -f pathmunge
#添加环境变量
PATH=$PATH:/usr/local/go/bin
export PATH
$ source /etc/profile
~~~

###17.centos7 安装 OpenSSL

~~~shell
$ wget http://www.openssl.org/source/openssl-1.0.2f.tar.gz
$ tar -zxvf tar -xzf openssl-1.0.2f.tar.gz
$ cd openssl-1.0.2f
$ mkdir /usr/local/openssl
$ ./config --prefix=/usr/local/openssl
$ make
$ make install
$ ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
$ cd /usr/local/openssl
$ ldd /usr/local/openssl/bin/openssl
$ vim /etc/ld.so.conf
#在最后追加一行
/usr/local/openssl/lib 
$ ldconfig /etc/ld.so.conf
$ openssl version
~~~

###18.删除网桥

~~~shell
$ yum install bridge-utils
#添加网桥
$ brctl addbr br0
#设置br0
$ ifconfig br0 192.168.1.100.1 netmask 255.255.255.0
#查看网桥
$ brctl show
#删除网桥
$ brctl delbr br0
# 将eth0端口加入网桥br0
$ brctl addif br0 eth0
# 从网桥br0中删除eth0端口
$ brctl delif br0 eth0
~~~

### 19.安装jdk和maven

~~~shell
$ mkdir /usr/local/java/
$ tar -zxvf jdk-8u171-linux-x64.tar.gz -C /usr/local/java/
$ mv /usr/local/java/jdk1.8.0_191 /usr/local/java/jdk
$ vi /etc/profile

export JAVA_HOME=/usr/local/java/jdk
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

$ source /etc/profile
#建立一个软连接
$ ln -s /usr/local/java/jdk/bin/java /usr/bin/java


#修改链接
$ ln -snf [新目标地址] [软连接地址]
#删除
$ rm -rf [软连接地址]
#安装maven
$ tar -zxf apache-maven-3.3.9-bin.tar.gz
$ unzip apache-maven-3.5.4-bin.zip
$ mv apache-maven-3.3.9 /usr/local/maven/
$ vi /etc/profile
M2_HOME=/usr/local/maven/apache-maven-3.3.9
export PATH=${M2_HOME}/bin:${PATH}
$ source /etc/profile
~~~

### 20.shell中正则表达

正则表达式是对文本进行过滤的工具，之所以有过滤文本的功能，因为它定义了一系列元字符，通过元字符配合其他字符来表达出一种规则。只有符合该规则上午文本才能保留下来

元字符： 描述字符的字符

元字符作用： 对表达式的内容转换以及各种操作信息进行描述

- 基础表达式

~~~shell
#基础表达式仅支持最基本的元字符集
#基础正则表达式的元字符主要有：

#1. 行首定位符 "^"
#用来匹配行首的字符。表示行首的字符是 ^ 后面的那个字符 行首定位符位于所作用的字符之前
# 例 列出 /etc 目录中以字母po开头的文件名
$ str = 'ls /etc | grep "^po"'
$ echo "$str" 

#2. 行尾定位符 "$"
#作用： 定位文本行的末尾 行尾定位符位于所作用的字符之后
#例 列出 /etc 目录中以conf结尾的文件名
$ ls /etc | grep 'conf$'
#注意： ^cat$ 完全匹配cat的文本行 单独的 ^ 和 $ 没有任何意义，因为任何一个文本行都有开头和结尾。

#3. 单个字符匹配 "."
# . 用来匹配任何单个字符。 包括空格 但不包括换行符\n。 当使用 "." 意味着该位置一定有一个字符 无论他是什么字符
# 例 列出所有的包含字符串 "samba" 的文件名， 不管samba后面有没有字符
$ ls /etc | grep 'samba'
# 列出包含字符串samba 且samba后面只是含有一个字符
$ ls /etc | grep 'samba.' 
#可以连续使用..来匹配多个字符，如l..p，匹配含义字母l,然后是两个任意字符，再接着是字母p的字符串

#4. 限定符 "*"
#限定符本身不代表任何字符，用来指定其前面的一个字符必须重复出现多次才能满足匹配。而星号表示匹配其前导字符的任意次数 包括0次
#例 筛选出以字符s开头 紧跟着1个字符s 再接着任意个字符s的文件名
$ ls /etc | grep '^sss*'

#5. 字符集匹配 "[]"
#只要某个字符串在方括号所在的位置上出现了方括号中的任意一个字符 就满足匹配条件。 对于连续的数字和字母 可使用 - 来表示一个范围   如： [a-z]表示匹配a到z中的任意一个字符 [0-9]匹配0-9任意一个数字

#6. [^] 字符集不匹配
~~~

- 扩展正则表达

~~~shell
#egerep命令默认使用扩展正则表达式

#1. 限定符 "+"  
#限定符+ 用于限定前面的字符至少出现一次
#例 筛选以字符串 "s" 开头， 后面至少紧跟着1个字符 "s" 的文本行
$ ls /etc | egrep '^ss+'

#2. 限定符 "?"
#限定前面的字符最多只出现一次
#例 筛选以字符串 "s"开头 后面跟着0个或者1个s的文本行
$ ls /etc | egrep '^ss?'

#3. 竖线 "|" 和圆括号 "()"
#竖线|表达多个正则表达式之间的关系 括号表示一组可选的集合 竖线和圆括号经常一起使用 表示一组可选值
#例 筛选含有字符串 "ssh" 或 "ssl" 或者 以字符串 "yum" 开头的文本行
$ ls /etc | egrep '(ssh|ssl|^yum)'
~~~

- perl正则表达式

~~~shell
1.数组匹配 \d
2.非数字匹配 \D
3.空白字符匹配 \s
4.非空白字符匹配 \S
~~~

- 应用

~~~shell
#1.单个一般字符 
#搜索文本中含有 "a" 的文本行
$ grep 'a' demo.txt

#2.转义后的元字符
#要匹配元字符本身 需要在这些字符前面加 \ 关闭这些元字符的特殊意义 保留其字面意义 反斜线也是元字符 如果匹配 要 '\\'

#3.方括号表达式
#当元字符位于方括号中时， 除了连字符 - 或者 ^ 之外 其他元字符都会失去特殊意义 [\.] 表示的是反斜线\和原点.这两个字符 如果匹配原点 [.] 就够了

#4. 方括号或星号等配合
#匹配任意多个字符 "o"
$ egrep 'lo*king' demo.txt

#优先级
1. \ 转义符
2. [] 方括号表达式
3. () 分组
4.  *.+?{m} 限定符
5. 普通字符 从左到右
6. ^ $ 定位符
7. | 或运算
（从高到低）
~~~

### 21.linux 网络管理命令

- linux中主要三类网络管理命令：1. ifconfig route netstat 属于传统功能单一 网络命令 2. ip ss 属于综合类网络命令 3. nmcli适用于RHEL7的综合网络命令

1. ifconfig 

格式： ifconfig [interface] [up | down]

常用选项： -a 显示所有网络接口信息 -s 显示网络接口统计信息 

ifconfig INT address 配置指定网络接口的IP 地址

ifconfig INT IP/MASK 修改指定设备的ip地址 

例： ifconfig eth0 192.168.1.166 255.255.255.0   

2. route 

格式:  route 查看路由条目        

-n 对域名不进行解析 以IP地址进行显示             

3. IP命令     

格式： ip [options] object { command | help }     

object 为 link 时 用于配置本机的二层链路属性配置    对应command为：  

ip link set DEVICE {up | down | arp {on | off }}    

例： ip link set eth0 down     ip link show  

object为address时 用于设置本机ip



ip addr { add | del } IFADDR dev STRING: 对指定网络接口添加或删除IP地址

ip addr { show | flush } [ dev STRING ]: 查看或清空指定设备的IP地址

add IP/MASK: 为设备添加地址

delete IP: 删除设备配置的地址

flush: 清空指定设备中的配置

show: 查看IP地址配置

通过add命令添加指定IP地址

ip addr 192.168.1.88/24 dev eth0    && ip addr show dev eth0



object 为 route 时 用于设置本机路由条目

ip route { list | flush } SELECTOR: 查看或清空路由条目

ip route { add | del | change | append | replace | monitor } ROUTE: 修改路由条目

add 添加路由条目

ip route add 0/0 via 10.10.10.252

change修改路由条目

ip route change 0/0 via 192.168.1.2 dev eth0



（ip route ip rule iptables 关系）https://www.cnblogs.com/EasonJim/p/8424731.html



### 22.给虚拟机添加新磁盘

~~~shell
#添加过程不做描述
#1.添加之后重启虚拟机 
$ shutdown -r now
#2.使用fdisk -l 命令查看当前系统的分区
Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00007915

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    41943039    19921920   8e  Linux LVM

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xc223a63b

#3.显示sdb未分区 对新建的磁盘进行分区
$ fdisk /dev/sdb

Command (m for help): 
Command action

#输入m 则会出现提示
#再输入n （add a new partition）添加一个新分区
#依次输入p 和 1 接着便是提示卷的起始地址和结束地址 都保持默认按回车（意思是只有一个分区）
#输入w保存退出
#再次使用fdisk -l 这个命令来查看会发现出现了 /dev/sdb1

#4.进行格式化操作 格式化成xfs文件系统
$ mkfs -t xfs /dev/sdb1
#5.对分区好的磁盘进行挂载
$ mount /dev/sdb1 /dataall

#常用小命令
$ df -lhT # 查看文件系统格式 mount 也可以 或者 file -s /dev/sda1



#使用parted分区
# 使用 lsblk,fdisk,df 等命令查看当前分区信息 
lsblk
fdisk -l
df -TH

# 使用 /dev/sdb1 为例
parted /dev/sdb1
parted (GNU parted) 3.1
Welcome to GNU Parted! Type 'help' to view a list of commands.

# 使用 help 查看帮助
(parted) help
  check NUMBER                             do a simple check on the file system
  cp [FROM-DEVICE] FROM-NUMBER TO-NUMBER   copy file system to another partition
  help [COMMAND]                           prints general help, or help on COMMAND
  mklabel,mktable LABEL-TYPE               create a new disklabel (partition table)
  mkfs NUMBER FS-TYPE                      make a FS-TYPE file system on partititon NUMBER
  mkpart PART-TYPE [FS-TYPE] START END     make a partition
  mkpartfs PART-TYPE FS-TYPE START END     make a partition with a file system
  move NUMBER START END                    move partition NUMBER
  name NUMBER NAME                         name partition NUMBER as NAME
  print [free|NUMBER|all]                  display the partition table, a partition, or all devices
  quit                                     exit program
  rescue START END                         rescue a lost partition near START and END
  resize NUMBER START END                  resize partition NUMBER and its file system
  rm NUMBER                                delete partition NUMBER
  select DEVICE                            choose the device to edit
  set NUMBER FLAG STATE                    change the FLAG on partition NUMBER
  toggle [NUMBER [FLAG]]                   toggle the state of FLAG on partition NUMBER
  unit UNIT                                set the default unit to UNIT
  version                                  displays the current version of GNU Parted and copyright information

# 建立磁盘标签
(parted) mklabel GPT
# 如果没有任何分区，它查看磁盘可用空间，当分区后，它会打印出分区情况
(parted) print
# 创建主分区，n 为要分的分区占整个磁盘的百分比
(parted) mkpart primary 0% 100%
#  分区完后，直接 quit 即可，不像 fdisk 分区的时候，还需要保存一下，这个不用
(parted) quit

# 让内核知道添加新分区
partprobe

# 格式化
mkfs.ext4 /dev/sdb1

# 挂载分区
mkdir /data
mount /dev/sdb1 /data

# 设置开机自动挂载磁盘
vim /etc/fstab
/dev/sdb1    /data    ext4    defaults    0    0

# fdisk 命令无法使用可以用 parted
fdisk -l
parted -l

# parted 有 2 种模式，使用命令行模式方便自动化
 命令行模式: parted [option] device [command]
交互模式: parted [option] device
~~~

### 23.kill命令与信号处理

#### 1.kill命令

~~~shell
# -l选项告诉kill命令启动进程的用户已注销的方式结束进程 当使用该选项时 kill命令也试图杀死所留下的子进程 但这个命令也不是总能成功 
$ kill -l pid

# 强大又危险 这个命令迫使进程在运行时突然停止 进程在结束后不能自我清理 危害是导致系统资源无法正常释放 一般不推荐使用
# 当使用该命令时，一定要通过ps -ef确认没有剩余任何僵尸进程 只能通过终止父进程来消除僵尸进程 如果僵尸进程被init收养 问题就比较严重了 杀死init进程意味着关闭系统 如果系统中有僵尸进程 并且父进程是init 僵尸进程占用了大量的系统资源 那么就需要在某个时候重启机器清除进程表了
$ kill -9 pid
~~~

#### 2.Linux Signal

##### 1.发送信号

- Ctrl-C发送INT signal（SIGINT),通常导致进程结束
- Ctrl-Z发送TSTP signal（SIGSTP),通常导致进程挂起（suspend）
- Ctrl-\发送QUIT signal(SIGQUIT),通常导致进程结束和dump core
- Ctrl-T(不是所有的unix都支持)发送INFO signal(SIGINFO),导致操作系统显示此运行命令的信息

`kill -9 pid`发送SIGKILL信号给进程

##### 2.处理信号

Signal handler可以通过`signal()`系统调用进行设置。如果没有设置，缺省的handler会被调用，当然进程也可以忽略此信号。

有两种信号不能被拦截和处理：`SIGKILL`和`SIGSTOP`

当接受到信号时，进程会根据信号的响应动作执行相应的操作，信号的响应动作有以下几种：

- 中止进程(Term)
- 忽略信号(lgn)
- 中止进程并保存内存信息(Core)
- 停止进程(Stop)
- 继续运行进程(Cont)

用户可以通过`signal`或`sigaction`函数修改信号的响应动作（也就是常说的“注册信号”）。另外，在多线程中，各线程的信号响应动作都是相同的，不能对某一个线程设置独立的响应动作。

##### 3.信号类型

常用的信号

|  信号   |    值    | 动作 |                   说明                   |
| :-----: | :------: | :--: | :--------------------------------------: |
| SIGHUP  |    1     | Term |     终端控制进程结束（终端连接断开）     |
| SIGINT  |    2     | Term |      用户发送INTR字符（Ctrl+C）触发      |
| SIGQUIT |    3     | Core |      用户发送QUIT字符（Ctrl+/）触发      |
| SIGILL  |    4     | Core |                 非法指令                 |
| SIGABRT |    6     | Core |            调用abort函数触发             |
| SIGKILL |    9     | Term | 无条件结束进程（不能被捕获、阻塞或忽略） |
| SIGTERM |    15    | Term |    结束进程（可以被捕获、阻塞、忽略）    |
| SIGSTOP | 17,19,23 | Stop |    停止进程（不能被捕获、阻塞、忽略）    |
| SIGSTP  | 18,20,24 | Stop |     停止进程（可以捕获、阻塞、忽略）     |

参考文章：https://colobu.com/2015/10/09/Linux-Signals/

### 24. 文件描述符

linux系统中通常会对每个进程所能打开的文件数量有一个限制 当进程中已经打开的文件描述符超过这个限制时 open（）等获取文件描述符的系统调用会返回失败

linux下最大文件描述符限制有两个方面 一个事用户级的限制 另一个是系统级限制

- 用户级限制：ulimit命令可以看到用户级最大文件描述符限制 也就说每一个用户登录后执行的程序占用文件描述符的总数不能超过这个限制
- 系统级限制：使用 sysctl命令和proc文件系统中查看到的数值是一样的 这个属于系统级限制 他是限制所有用户打开文件描述符的总和

查看用户级

~~~shell
$ ulimit -n
~~~

查看系统级

~~~shell
$ sysctl -a | grep file-max
$ cat /proc/sys/fs/file-max
~~~

修改

~~~shell
#临时修改 用户级
$ ulimit -SHn 2048

#永久修改 用户级
$ vi /etc/security/limits.conf

#修改系统级别限制
通过sysctl命令 修改/etc/sysctl.conf文件： sysctl -w fs.file-max=2048, 完成后执行 sysctl -p 即可
~~~

参考文章： https://www.jianshu.com/p/20f1e96557e3

### 25.linux的inotify机制

自内核2.6.13起，linux提供inotify机制，以允许应用程序监控文件事件

队列限制和/proc文件 

- 对inotify事件做排队处理 需要消耗内核内存 正因如此 内核会对inotify机制的操作实施各种限制 超级用户可以配置/proc/sys/fs/inotify路径三个文件调整限制

  1. max_queued_events

     调用inotify_init()时，使用该值为新inotify实例队列中的事件数量设置上限 一旦超出这个上限 系统将生成IN_Q_OVERFLOW事件 并丢弃多余的事件 溢出事件的wd字段值为-1

  2. max_user_instances

     对由每个真实用户ID创建的inotify实例数的限制值

     设置值的方法（ sysctl -w fs.inotify.max_user_watches="99999999" ）

  3. max_user_watches

     对由每个真实用户ID创建的监控项数量的限制值

### 26.crontab

![](image\linux\crontab.png)



### 27. firewall 常用操作

~~~shell
#对外暴露端口
$ firewall-cmd --permanent --add-port=8080
#只允许某一个网段访问主机的某一个端口
$ firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="1.1.1.1/24" port protocol="tcp" port="3306" accept" 
#reload
$ firewalld-cmd --reload
#端口转发，将到本机的3306端口的访问转发到192.168.1.1服务器的3306端口
#开启伪装IP
$ firewalld-cmd --permanent --add-masquerade
#配置端口转发
$ firewalld-cmd --permanent --add-forward-port=3306:proto=tcp:toaddr=192.168.1.1.2:toport=13306


#查看状态
$ firewalld-cmd --state
#查看规则
$ firewalld-cmd --list-all
#查看所有的防火墙策略
$ firewalld-cmd --list-all-zones
#重新加载配置文件
$ firewalld-cmd --reload

#查看所有打开的端口： firewall-cmd --zone=public --list-ports
#查看区域信息:  firewall-cmd --get-active-zones
#查看指定接口所属区域： firewall-cmd --get-zone-of-interface=eth0
#拒绝所有包: firewalld-cmd --panic-on
#取消拒绝状态： firewalld-cmd --panic-off

~~~

### 28.nmap

~~~shell
nmap -sP -PR 192.168.0.0/24
~~~

### 29.linux常用命令

- **netstat**

~~~shell
-n 直接使用ip地址
-l 显示监控中服务器的socket
-p 显示正在使用socket程序识别码和进程名称
-t tcp
-u udp
~~~

- **ps**

~~~BASH
-A 显示所有进程 == -e
f 显示进程间的关系
--lines 一页显示多少行
aux 列出所有正在内存当中的程序
~~~

- **free**

~~~shell
#可以显示linux系统中空闲的和正在使用的物理内存 swap内存以及被内核使用的buffer
-m 　以MB为单位显示内存使用情况
-t 　显示内存总和列
-h   易懂的方式输出
~~~

- **du**

~~~shell
#显示每个目录或文件使用的磁盘空间
-h 人性化输出
~~~

- **top**

~~~shell
#top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器
PID   进程id
USER  进程所有者
PR    进程优先级
NI    nice值
VIRT  进程使用的虚拟内存总量
RES   进程使用的 未被换出的物理内存大小
S     进程状态 D不可中断的睡眠状态 R运行 S睡眠  T跟踪/停止 Z僵尸进程

-d   每隔几秒显示所有进程占用情况
-p   显示某一个进程的资源使用情况
~~~

-  python

~~~shell
echo '{"name":"chen","age":"11"}' |python -m json.tool
~~~



### 30.LInux状态码的意义

~~~shell
0		命令成功结束
1		通用未知错误
2		误用shell命令
126		命令不可执行
127		没有找到命令
128		无效退出命令
128+x	Linux信号x的严重错误
130		Linux信号2的严重错误 即命令通过SIGINT(Ctrl+C)终止
255		退出状态码越界
~~~

### 31.一些有用的linux命令

~~~shell
#sort 排序
ls -all | sort -nk5
#-n字符串按数值比较 -k表示第几列

#shuf 乱序输入文件的行
echo -e 'a\nb\nc' | shuf

#uniq 前提是内容已经排序 -c用来计数 -u用来只显示不重复的内容 -d用来只显示重复的内容

#join  
https://my.oschina.net/tinyhare/blog/828320
~~~

### 32.tar命令

~~~shell
#打包命令
tar -cvf test.tar a b c d
#解压命令
tar -xvf test.tar
~~~

### 33. docker 安装 yapi

```https://blog.csdn.net/Dream_xun/article/details/106986677```

### 34.软连接

```
由于目录有存储空间限制，当存储空间不足时，可以使用软链接缓解空间不足的问题：将文件内容移动到较大空间的目录内，使用ln命令建立实际存储路径与虚拟访问路径的软链接（需要root用户登录）：

移动文件夹至实际存储路径,前一个为原路径，后一个为目的路径：

 mv /usr/share/nginx/html/nextcloud/data /data4/nextcloud/

建立软链接，前一个为实际存储路径，后一个为虚拟访问路径，两者权限相同：

ln -s /data4/nextcloud/data /usr/share/nginx/html/nextcloud/data
```

### 35.查看网卡是否为万兆卡

`lspci -vvv | grep Ethernet`

### 36.Thread died in Berkeley DB library

#### 问题解决

　　01、删除yum临时库文件

　　　　rm -fr /var/lib/rpm/__db.*

　　02、重建rpm数据库

　　　　rpm --rebuilddb

　　03、清理缓存及生产yumdb缓存

　　　　yum clean all

　　　　yum makecache

### 37. linux中的中断信号

信号（signal）是linux，类unix和其他posix兼容的操作系统中用来进程间通信的一种方式。一个信号就是一个异步的通知，发送给某个进程，或者同进程的某个线程，告诉它们某个事件发生了

当信号发送到某个进程中时，操作系统会中断该进程的正常流程，并进入相应的信号处理函数执行操作，完成后再刚回到中断的地方继续执行。

| 信号    | 值       | 动作 | 说明                                                         |
| :------ | :------- | :--- | :----------------------------------------------------------- |
| SIGHUP  | 1        | Term | 终端控制进程结束(终端连接断开)                               |
| SIGINT  | 2        | Term | 用户发送INTR字符(Ctrl+C)触发                                 |
| SIGQUIT | 3        | Core | 用户发送QUIT字符(Ctrl+/)触发                                 |
| SIGILL  | 4        | Core | 非法指令(程序错误、试图执行数据段、栈溢出等)                 |
| SIGABRT | 6        | Core | 调用abort函数触发                                            |
| SIGFPE  | 8        | Core | 算术运行错误(浮点运算错误、除数为零等)                       |
| SIGKILL | 9        | Term | 无条件结束程序(不能被捕获、阻塞或忽略)                       |
| SIGSEGV | 11       | Core | 无效内存引用(试图访问不属于自己的内存空间、对只读内存空间进行写操作) |
| SIGPIPE | 13       | Term | 消息管道损坏(FIFO/Socket通信时，管道未打开而进行写操作)      |
| SIGALRM | 14       | Term | 时钟定时信号                                                 |
| SIGTERM | 15       | Term | 结束程序(可以被捕获、阻塞或忽略)                             |
| SIGUSR1 | 30,10,16 | Term | 用户保留                                                     |
| SIGUSR2 | 31,12,17 | Term | 用户保留                                                     |
| SIGCHLD | 20,17,18 | Ign  | 子进程结束(由父进程接收)                                     |
| SIGCONT | 19,18,25 | Cont | 继续执行已经停止的进程(不能被阻塞)                           |
| SIGSTOP | 17,19,23 | Stop | 停止进程(不能被捕获、阻塞或忽略)                             |
| SIGTSTP | 18,20,24 | Stop | 停止进程(可以被捕获、阻塞或忽略)                             |
| SIGTTIN | 21,21,26 | Stop | 后台程序从终端中读取数据时触发                               |
| SIGTTOU | 22,22,27 | Stop | 后台程序向终端中写数据时触发                                 |

### 38.core文件

在程序不寻常退出时，内核会在当前工作目录下生成一个core文件（是一个内存映像，同时加上调试信息）。使用gdb来查看core文件，可以指示出导致程序出错的代码所在文件和行数。

通过ulimit -c可以查看core文件的大小。0位关闭  

### 39.mlocate文件过大问题

**locate**命令用来查找文件和目录。locate命令要比find -name快得多，原因在于它不搜索具体目录，而是搜索一个数据库/var/lib/mlocate/mlocate.db。这个数据库包含本地所有文件信息。系统自动创建这个数据库，并且每天更新一次。使用updatedb命令，手动更新数据库。

mlocate.db文件过大解决办法

1.修改/etc/updatedb.conf  在	PRUNEPATHS参数后面增加不需要进行locate的目录。修改后执行

`sudo pkill updatedb.mlocate`

`updatedb`

### 40.netstat与lsof

#### netstat

（https://blog.csdn.net/LiuXingSiYe/article/details/86539480）

常用参数：

-a： 显示所有链接

-n：直接使用IP地址，而不通过域名服务器

-o： 显示计时器

-p：显示正在使用socket的程序识别码和程序名称

-l：显示监控中的服务器的socket

常用技巧：

- 查找请求最多的前20个ip（常用于查找攻击开源）

`netstat -anlp|grep 80|grep tcp|awk '{print $5}'|awk -F: '{print $1}'|sort|uniq -c|sort -nr|head -n20`

`netstat -ant |awk '/:80/{split($5,ip,":");++A[ip[1]]}END{for(i in A) print A[i],i}' |sort -rn|head -n20`



#### lsof

lsof -i:[端口号]



lsof -i:8080：查看8080端口占用
lsof abc.txt：显示开启文件abc.txt的进程
lsof -c abc：显示abc进程现在打开的文件
lsof -c -p 1234：列出进程号为1234的进程所打开的文件
lsof -g gid：显示归属gid的进程情况
lsof +d /usr/local/：显示目录下被进程开启的文件
lsof +D /usr/local/：同上，但是会搜索目录下的目录，时间较长
lsof -d 4：显示使用fd为4的进程
lsof -i -U：显示所有打开的端口和UNIX domain文件

### 41.iptables

#### 增加白名单

配置22端口的白名单 -s 指定源

`iptables -A INPUT -s 192.168.1.0/24 -p tcp --dport 22 -j ACCEPT `

这个放到最后提条（-A参数表示放到队首，-I表示放到最后。iptables由上往下匹配，一旦符合，后面都不匹配）

`iptables -A INPUT -p tcp --dport 22 -j DROP`

```
iptables -L INPUT -n --line-number #列出所有INPUT规则
iptables -D INPUT 4 # 删除第四条规则
```

- 查询：iptables [-t table] [-L] [-nv]

-t ：nat filter mangle

-L：PROROUTING INPUT FORWARD OUTPUT POSTROUTING

-n：直接显示ip，速度回快些

-v：列出更多信息，包括通过该规则的数据包总位数、相关的网络接口

- 清除：iptables [-t tables] [-FXZ]

-F: 清除所有已经制定的规则

-X：删除所有使用者自定义的chain

-Z：将所有的chain的计数与流量统计都清零



### 42.sed命令

``` 
sed -e '/abc/d' a.log #删除a.log中包含‘abc’的行，但不改变a.log本身，操作之后的结果显示在终端
sed -e ‘/abc/d’ a.log > a.log.bak #删除a.logzhong包含 ‘abc’的行，将结果保存到a.log.bak
sed '/abc/d;/efg/d' a.log > a.log.bak #删除含字符串 ‘abc’或‘efg’的行
sed -i 's/test/mytest/g' example #把test替换成mytest -i 是直接修改并保存
```

### 43.查看磁盘io

#### top命令

```
Tasks: 103 total,   2 running, 101 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 99.0 id,  0.0 wa,  0.3 hi,  0.0 si,  0.0 st
```

Tasks: 

103 total 进程总数

2 running 正在运行的进程数

101 sleeping 睡眠的进程数

0 stopped 停止的进程数

0 zombie 僵尸进程数

%Cpu(s):

0.3 us 用户空间占用cpu百分比

0.3 sy 内核空间占用cpu百分比

0.0 ni 用户进程空间内改变过优先级的进程占用cpu百分比

99.0 id 空闲cpu百分比

0.0 wa 等待输入输出的cpu时间百分比

0.3 hi 

#### iostat

iostat -dx

#### 清理 /var/log/journal

`journalctl --vacuum-size=10M`



### 44. centos查看用户和用户组

- 查看用户列表：`cat /etc/passwd`
- 查看用户组列表 `cat /etc/group`
- 查看系统中有哪些用户 `cut -d : -f 1 /etc/passwd`

- 查看可以登陆系统的用户 `cat /etc/passwd | grep -v /sbin/nologin |cut -d : -f 1`
- 查看用户操作权限： `w`
- 查看某一个用户 `w <user>`

### 45. 创建用户指定家目录

linux创建新用户，当前用户必须为root用户

```python
useradd -d /export/gaoxin -m gaoxin
```

### 46. iperf 用法

#### 1. 查看当前节点的网速

`iptraf -g`

#### 2. iperf3

常用参数：

-s –server 服务器模式

-c –client 客户端模式

-i  指定每次报告之间的时间间隔

-p 端口

-u –udp 表示采用UDP协议发送报文

-l  设置读写缓冲区的长度

-b –bandwidth 指定UDP模式使用的带宽

-t 	指定数据传输的总时间

-F   指定文件作为数据流进行带宽测试
