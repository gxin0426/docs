### 0.配置阿里yum源

~~~shell
#配置阿里yum源命令
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
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
df -h：显示磁盘分区上可以使用的磁盘空间 #-a    #查看全部文件系统，单位默认KB   -h  使用-h选项以KB、MB、GB的单位来显示，可读性高~~~（最常用）
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
$ vi /etc/profile

export JAVA_HOME=/usr/local/java/jdk1.8.0_171
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

$ source /etc/profile
#建立一个软连接
$ ln -s /usr/local/java/jdk1.8.0_171/bin/java /usr/bin/java
#修改链接
$ ln -snf [新目标地址] [软连接地址]
#删除
$ rm -rf [软连接地址]
#安装maven
$ tar -zxf apache-maven-3.3.9-bin.tar.gz
$ unzip apache-maven-3.5.4-bin.zip
$ mv apache-maven-3.3.9 /usr/local/maven/
$ vi /etc/profile
M2_HOME=/usr/local/maven/apache-maven-3.5.4
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





