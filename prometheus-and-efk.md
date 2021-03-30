## 1.prometheus pvc 挂载ceph或者nfs

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

## 2. es介绍

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
https://segmentfault.com/a/1190000011661882

```

## 3.elk收集pod日志 以及docker打印日志的几种方式

- docker日志可以分成两类：1.stdout标准输出 2.是文件日志

stdout是写在标准输出里面的日志，比如你在程序里面，通过print或者echo来输出的时候，这种输出标准在linux上面其实是往一个ID为零的文件表述书里面去写；另外的就是文件日志，文件日志就是写在磁盘上的日志，一般来说我们会在传统的应用里面会用得多一些。

我们通过exec.Command启动了一个命令，带一些参数，然后就可以通过标准的pipeline拿到标准输出，后面就可以拿到程序运行过程中产生标准输出。 Docker也是用这个原理来拿的，所有的容器通过Docker Daemon启动，实际上属于Docker的一个子进程， 它可以拿到你的容器里面进程的标准输出，然后拿到标准输出之后，会通过它自身的一个叫做LogDriver的模块来处理，LogDriver就是Docker用来处理容器标准输出的一个模块。 Docker支持很多种不同的处理方式，比如你的标准输出之后，在某一种情况下会把它写到一个日志里面，Docker默认的JSON File日志，除此之外，Docker还可以把它发送到syslog里面，或者是发送到journald里面去，或者是gelf的一个系统。

![](dockerimage\docker日志打印方式.png)

![](dockerimage\docker的logdriver.png)



## 4.filebeat影响内存的两个参数

    queue.mem.events消息队列的大小，默认值是4096，这个参数在6.0以前的版本是spool-size，通过命令行，在启动时进行配置
    max_message_bytes 单条消息的大小, 默认值是10M

filebeat最大的可能占用的内存是max_message_bytes * queue.mem.events = 40G，考虑到这个queue是用于存储encode过的数据，raw数据也是要存储的，所以，在没有对内存进行限制的情况下，最大的内存占用情况是可以达到超过80G。









## 1.常用命令

1. **获取每个节点的可用磁盘空间:**   ```curl -XGET localhost:9200/_cat/allocation?v```

2. **删除某一个索引：** ```DELETE [索引名]```

3. **删除数据（保留索引）**

   ~~~shell
   POST 索引名称/文档名称/_delete_by_query   
   {
     "query":{
       "term":{
         "_id":100000100
       }
     }
   }
   ~~~

4. **删除所有数据**

   ~~~shell
   POST /testindex/testtype/_delete_by_query?pretty
   {
       "query": {
           "match_all": {
           }
       }
   }
   ~~~

   

## 2.原理组成

## 3.es-operator

### 4.filebeat配置相关

#### 1.关于filebeat配置的文章

`http://www.ttlsa.com/elk/elk-beats-common-configure-section-describe/`
`https://segmentfault.com/a/1190000020153303`

#### 2.grafana链接es使用变量

https://my.oschina.net/CainGao/blog/4425041

