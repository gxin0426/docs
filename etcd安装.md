## ETCD安装

- 下载二进制包（安装cfssl方式一）

~~~shell
$ wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
$ chmod +x cfssl_linux-amd64
$ sudo mv cfssl_linux-amd64 /root/local/bin/cfssl
$ wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
$ chmod +x cfssljson_linux-amd64
$ sudo mv cfssljson_linux-amd64 /root/local/bin/cfssljson
$ wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
$ chmod +x cfssl-certinfo_linux-amd64
$ sudo mv cfssl-certinfo_linux-amd64 /root/local/bin/cfssl-certinfo
$ export PATH=/root/local/bin:$PATH
~~~

- go安装（方式二）

~~~shell
$ go get -u github.com/cloudflare/cfssl/cmd/...
$ echo $GOPATH
/usr/local
$ ls /usr/local/bin/cfssl*
cfssl cfssl-bundle cfssl-certinfo cfssljson cfssl-newkey cfssl-scan
~~~

- 创建CA证书

~~~shell
$ mkdir /root/ssl
$ cd /root/ssl
$ cfssl print-defaults config > ca-config.json（修改参数）
$ cfssl print-defaults csr > ca-csr.json
#创建CA证书签名请求配置ca-csr.json
$ cat ca-csr.json
{
    "CN": "gree",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "shenzhen",
            "ST": "shenzhen",
            "O":"gree",
            "OU":"cloudgree"
        }
    ]
}


#ca-config.json中可以定义多个profile，分别设置不同的expiry和usages等参数。如上面的ca-config.json中#定义了名称为frognew的profile，这个profile的expiry 87600h为10年，useages中：

#signing表示此CA证书可以用于签名其他证书，ca.pem中的CA=TRUE
#server auth表示TLS Server Authentication, 即client可以用该 CA 对server提供的证书进行验证
#client auth表示TLS Client Authentication，即server可以用该CA对client提供的证书进行验证
$cat ca-config.json
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "etcd": {
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ],
                "expiry": "87600h"
            }
        }
    }
}

#下面使用cfssl生成CA证书和私钥:
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
$ ls ca*
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem
~~~

- 创建etcd证书签名请求配置etcd-csr.json：

~~~shell
{
    "CN": "etcd",
    "hosts": [
      "127.0.0.1",
      "192.168.1.155",
      "192.168.1.156",
      "192.168.1.157"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "shenzhen",
            "L": "shenzhen",
            "O": "gree",
            "OU": "cloudgree"
        }
    ]
}
~~~

注意上面配置hosts字段中制定授权使用该证书的IP和域名列表，因为现在要生成的证书需要被etcd集群各个节点使用，所以这里指定了各个节点的IP和hostname。



下面生成etcd的证书和私钥：

~~~shell
$ cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=frognew etcd-csr.json | cfssljson -bare etcd

#对生成的证书可以使用cfssl或者openssl查看：
$ cfssl-certinfo -cert etcd.pem
$ openssl x509  -noout -text -in  etcd.pem

#确认 Issuer 字段的内容和 ca-csr.json 一致；
#确认 Subject 字段的内容和 etcd-csr.json 一致；
#确认 X509v3 Subject Alternative Name 字段的内容和 etcd-csr.json 一致；
#确认 X509v3 Key Usage、Extended Key Usage 字段的内容和 ca-config.json 中 profile 一致；
~~~

- 安装etcd（二进制)

~~~shell
wget https://github.com/coreos/etcd/releases/download/v3.1.6/etcd-v3.1.6-linux-amd64.tar.gz
#解压缩etcd-v3.1.6-linux-amd64.tar.gz，将其中的etcd和etcdctl两个可执行文件复制到各节点的/usr/bin目录。
mkdir -p /var/lib/etcdcd 

#使用systemctl启动和管理etcd服务，在每个节点上创建etcd的systemd unit文件/usr/lib/systemd/system/etcd.service，注意替换ETCD_NAME和INTERNAL_IP变量的值：

cat  /usr/lib/systemd/system/etcd.service
[Unit]
Description=etcd server
After=network.target
After=network-online.target
Wants=network-online.target
[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
ExecStart=/usr/bin/etcd \
  --name ${ETCD_NAME} \
  --cert-file=/etc/etcd/ssl/etcd.pem \
  --key-file=/etc/etcd/ssl/etcd-key.pem \
  --peer-cert-file=/etc/etcd/ssl/etcd.pem \
  --peer-key-file=/etc/etcd/ssl/etcd-key.pem \
  --trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \
  --listen-peer-urls https://${INTERNAL_IP}:2380 \
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://${INTERNAL_IP}:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster node1=https://192.168.202.131:2380,node2=https://192.168.202.132:2380,node3=https://192.168.202.133:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target



~~~

- –data-dir指定了etcd的工作目录和数据目录是/var/lib/etcd
- –cert-file和–key-file分别指定etcd的公钥证书和私钥
- –peer-cert-file和–peer-key-file分别指定了etcd的Peers通信的公钥证书和私钥。
- –trusted-ca-file指定了客户端的CA证书
- –peer-trusted-ca-file指定了Peers的CA证书
- –initial-cluster-state new表示这是新初始化集群，–name指定的参数值必须在–initial-cluster中

~~~shell
#启动etcd
$ systemctl daemon-reload
$ systemctl enable etcd
$ systemctl start etcd
$ systemctl status etcd
#在启动etcd的时候，可以开启另一个命令窗口，查看启动日志，确保没有报错
journalctl -f

#如果出现了形如 unkown flag的字段，表示启动参数错误，不识别,说明该参数拼写错误(如–keyfile应当为–key-file),可以到官方配置文档Configuration flags查看该参数的写法，确保正确。
#如果出现Failed to find member fXXXXXX的错误，这说明之前启动的etcd时，标识号出现错误，此时删除/var/lib/etcd/member目录，让etcd重新为每个节点分配标识号, /var/lib/etcd为etcd启动配置工作目录

#配置etcdctl
$ vi /etc/etcd/etcdctl
$ cat /etc/etcd/etcdctl
export ETCDCTL_API=3
export ETCDCTL_ENDPOINTS="https://etcd-1:2379,https://etcd-2:2379,https://etcd-3:2379"
export ETCDCTL_CACERT=/etc/etcd/ssl/ca.pem
export ETCDCTL_CERT=/etc/etcd/ssl/etcd.pem
export ETCDCTL_KEY=/etc/etcd/ssl/etcd-key.pem
$ source /etc/etcd/etcdctl
~~~

### etcdctl常用命令

~~~shell
#查看成员
[root@master1 ssl]# etcdctl member list
2019-09-16 23:07:23.125547 I | warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
22da419b0cf8d1be, started, etcd2, https://192.168.1.156:2380, https://192.168.1.156:2379
323e5ea4891348e5, started, etcd3, https://192.168.1.157:2380, https://192.168.1.157:2379
810e24f2b2956f47, started, etcd1, https://192.168.1.155:2380, https://192.168.1.155:2379
#删除一个失败的成员
etcdctl member remove 22da419b0cf8d1be
#添加一个成员
etcdctl member add member4 --peer-urls=http://10.0.0.4:2380

export ETCD_NAME="member4"
export ETCD_INITIAL_CLUSTER="member2=http://10.0.0.2:2380,member3=http://10.0.0.3:2380,member4=http://10.0.0.4:2380"
export ETCD_INITIAL_CLUSTER_STATE=existing
etcd [flags]
~~~

### 数据备份

~~~shell
#v3数据备份
$ ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT snapshot save /opt/snapshotdb.db
# 验证快照
$ ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshotdb
#数据恢复
$ etcdctl snapshot restore /opt/snapshot.db --name m3 --data-dir= /var/lib/etcd

~~~

### troubleshooting

~~~shell
#1.Failed at step CHDIR spawning /usr/bin/etcd: No such file or directory
fix:创建data-dir ：/var/lib/etcd
#2.health check for peer 323e5ea4891348e5 could not connect: dial tcp 192.168.1.157:2380: getsockopt: connection refused
~~~

### 可视化界面

```
docker run --rm  -d --name etcd-browser \
-p 8000:8000 \
--env ETCD_HOST=10.211.55.25 \
--env ETCD_PORT=2379 \
buddho/etcd-browser
```

```
#输入地址即可
docker run -it -d --name etcdkeeper \
-p 8080:8080 \
deltaprojects/etcdkeeper
```

