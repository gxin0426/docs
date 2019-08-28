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





