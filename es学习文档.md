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

4.  **删除所有数据**

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



