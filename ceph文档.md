### 1.删除ceph osd pool

- 删除osd步骤

~~~shell
1.ceph osd down osd.num
2.ceph osd out osd.id
3.ceph osd rm osd.id
4.ceph osd crush rm osd.id
5.ceph auth del osd.id
~~~

- 删除pool步骤

~~~shell
1.ceph osd lspools
2.ceph --show-config | grep mon_allow_pool_delete
3.systemctl list-units --type=service | grep ceph
4.修改ceph配置文件
5.重启mgr服务
6.。。。
7. 。。。
8.ceph osd pool rm pool-name pool-name --yes-i-really-mean-it
~~~

### 创建pool

~~~shell
ceph osd pool create {pool-name} {pg-num} [{pgp-num}] [replicated] [crush-ruleset-name] [expected-num-objects]

#设置pool配额 支持object个数配额以及容量大小配额 设置允许最大object数量为50000
$ ceph osd pool set-quota test-pool max_objects 50000
#设置允许容量限制为200G
$ ceph osd pool set-quato test-pool max_bytes $((200 * 1024 *1024 * 1024))
#取消配额限制只需要吧对应值设置为0即可

#重命名
$ ceph osd pool rename test-pool test-pool-rename
#查看pool状态信息
$ rados df

#创建快照   ceph支持对整个pool创建快照 作用于这个pool的所有对象 但是注意ceph有两种pool模式
#1. pool snapshot 我们即将使用的模式 创建一个新的pool时 默认也是这种模式
#2. self managed snapshot 用户管理的snapshot 这个用户指的是librbd 也就是说 如果在pool创建里rbd实例就自动转化为这种模式
#这两种模式项目排斥 只能用其中一个 因此 如果pool中曾经创建了rbd对象（即使当前删除了所有的image实例） 就不能再对这个pool做快照了 反之 如果对一个pool做了快照 就不能创建rbd image了

$ ceph osd pool mksnap test-pool test-pool-snapshot
#删除快照
$ceph osd pool rmsnap test-pool test-pool-snapshot
#设置pool
#1. 设置pool的元数据
$ ceph osd pool set {pool-name} {key} {value}
#设置pool的冗余副本数量为3
$ ceph osd pool set test-pool size 3
#通过get操作获取pool的配置值 比如获取当前pg_num
$ ceph osd pool get test-pool pg_num
#获取副本数
$ ceph osd pool get test-pool size
~~~





