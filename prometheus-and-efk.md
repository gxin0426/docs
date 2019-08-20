### 1.prometheus pvc 挂载ceph或者nfs

```yaml
volumeClaimTemplates:
  - metadata:
      name: elasticsearch-logging
    spec:
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 1Gi
      storageClassName: nfs-elk
```

### 2. es介绍

- Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个 Elastic 实例。

  单个 Elastic 实例称为一个节点（node）。一组节点构成一个集群（cluster）

- Elastic 会索引所有字段，经过处理后写入一个反向索引（Inverted Index）。查找数据的时候，直接查找该索引。所以，Elastic 数据管理的顶层单位就叫做 Index（索引）。它是单个数据库的同义词。每个 Index （即数据库）的名字必须是小写。

- 倒排索引源于实际应用中需要根据属性的值来查找记录。这种索引表中的每一项都包括一个属性值和具有该属性值的各记录的地址。由于不是由记录来确定属性值，而是由属性值来确定记录的位置，因而称为倒排索引(inverted index)。带有倒排索引的文件我们称为倒排索引文件，简称倒排文件(inverted file)。

- 映射 存储有关字段的信息，每一个文档类型都有自己的映射

- 基本结构

```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

VERB： 适当的 HTTP 方法 或 谓词 : GET、 POST、 PUT、 HEAD 或者 DELETE

PROTOCOL：http 或者 https

HOST：Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点

PORT： 运行 Elasticsearch HTTP 服务的端口号，默认是 9200

PATH： API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：_cluster/stats 和 _nodes/stats/jvm

QUERY_STRING：任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读)

BODY：一个 JSON 格式的请求体 (如果请求需要的话)

```shell
#新建index
curl -X PUT 'localhost:9200/weather'

#删除index
curl -X DELETE 'localhost:9200/weather'

#检测集群是否健康
curl 'localhost:9200/_cat/health?v'

#查看分片状态
curl -X GET 'http://localhost:9200/_cluster/health?pretty'

#获取集群节点
curl 'localhost:9200/_cat/nodes?v'

#列出所有索引
curl 'localhost:9200/_cat/indices?v'

#查询索引数据
curl 'localhost:9200/brand/_search?pretty=true'

#参考链接
https://www.jianshu.com/p/7fe83806b909


```

