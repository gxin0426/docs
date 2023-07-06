| dmesg | dd     | tcpdump | ss          | top               | du    |
| ----- | ------ | ------- | ----------- | ----------------- | ----- |
| iperf | find   | awk     | sed         | grep              | route |
| ip    | lsof   | netstat | rpm         | dpkg              | diff  |
| ps    | kill   | unset   | EOF >>      | 特殊符号: % $ # & | until |
| cut   | vmstat | free    | curl & wget | iptables          | nmap  |
| jq    |        |         |             |                   |       |
|       |        |         |             |                   |       |
|       |        |         |             |                   |       |
|       |        |         |             |                   |       |
|       |        |         |             |                   |       |

Shell 常用脚本30例:  https://zhuanlan.zhihu.com/p/526263882

#### dmesg

命令用于显示开机信息

#### dd

#### ss

#### top

#### tcpdump

```bash
#经过eth0  目的IP 192.168.0.22
tcpdump -i eth0 (src / dst) host 192.168.0.22

#经过eth0 目的端口 1234
tcpdump -i eth0 dst port 1234

#经过eth0 所有udp协议的包
tcpdump -i eth0 udp

#本地环路数据包
tcpdump -i lo udp

#抓取所有经过1234端口的UDP网络数据
tcpdump udp port 1234

#抓取所有经过eth0 SYN类型数据包
tcpdump -i eth0 'tcp[tcpflags]=tcp-syn' 

#所有经过eth0 的dns数据包
tcpdump -i eth0 udp dst port 53

tcpdump tcp -i eth0 -t -s 0 -c 100 and dst port !22 and src net 192.168.1.0/24 -w /opt/tmp.cap
#tcp: ip icmp arp rarp tcp udp icmp 这些参数都要放到第一个参数位置，用来过滤数据报的类型
#-i: 指定网卡
#-s: -s 0 可以抓到完整的数据包 默认抓取68字节
#-c: -c 100  只抓取100个数据包
#dst port !22: 不抓目标端口22
#src net 192.168.1.0/24: 数据包源ip范围 
#-w: 保存成文件 方便wireshark分析


```



