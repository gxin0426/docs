## redis

### 通用命令

- **keys *     查看所有key**
- **dbsize    查看key的总数**
- **exists key   查看key是否存在**
- **del key** 
- **expire key second    设置过期时间**
- **ttl key    查看过期时间**
- **type key    查看key的数据类型**

### 字符串 str

#### 常用命令

- **常规命令**
  - **GET**
  - **SET** 
    - set key value [ex seconds] [px milliseconds] [nx|xx]
      - ex 设置秒级过期时间
      - px 设置毫秒级过期时间
      - nx：键必须不存在，才能设置成功 可以用setnx代替（返回为0表示不成功）
      - xx：键必须存在才能设置成功 可以用setxx代替
  - **mset** 批量设置 MSET key001x 'test1' key002x 'bbb’
  - **mget** 批量获取
  - **DEL**

- **INCR** 
  - incr key  将键存储的值加一
- **DECR** 
  - decr key 将键存储的值减一
- **INCRBY** 
  - incrby key amount 将健存储的值加上整数amount
- **DECRBY** 
  - decrby key amount 将键存储的值减去证书amount
- **INCRBYFLOAT**
  -  incrbyfloat key amount 将键存储的值加上浮点数amount 
- **APPEND** 
  - append key value 将value追加到给定key当前存的值末尾

- **GETRANGE**
  - getrange key start end 获取一个由偏移量start至偏移量end范围内的所有字符组成的子串，包括start和end

- **SETRANAGE**
  - setrange key offset value 将从start偏移量开始的子串设定为给定值


### 列表 list

#### 常用命令

- **rpush**
  - rpush key value [value …] 从右边插入元素
- **lpush**
  - lpush key value [value …] 从左边插入元素
- **linsert**
  - linsert key before|after pivot value 向某个元素前或者后插入元素
- **lrange**
  - lrange key start end 获取指定范围的元素列表 （0 -1表示获取所有）
- **lindex**
  - lindex key index 获取列表指定索引下标的元素

- **llen**
  - llen key 获取列表长度
- **lpop**
  - lpop key 从列表左侧弹出元素
- **rpop**
  - rpop key 从列表右侧弹出
- **lrem**
  - lrem key count value 删除指定元素，lrem命令从列表中找到等于value的元素，根据count不同分三种情况
    - count>0 从左到右 删除最多count个元素
    - count<0 从右到左 删除最多count绝对值个元素
    - count=0 删除所有
- **ltrim**
  - ltrim key start end 按照索引范围修剪列表
- **lset**
  - lset key index newVAlue 修改指定索引下标的元素
- **blpop brpop**
  - blpop key [key …] timeout 阻塞式弹出

### 散列 hash

#### 常用命令

- **hset** 
  - hset key field value 设置值
- **hget** 
  - hget key field 获取值
- **hdel** 
  - hdel key field [field…] 删除field
- **hlen** 
  - hlen key 计算field个数
- **hmset** 
  - hmset key field value [field value …] 批量设置field-value
- **hmget** 
  - hmget key field [field …] 批量获取field
- **hexists** 
  - hexists key field 判断field是否存在
- **hkeys** 
  - hkeys key 获取所有field
- **hvals** 
  - hvals key 获取所有value 
- **hgetall** 
  - hgetall key 获取所有的key-value
- **hincrby hincrbyfloat** 和incrby incrbyfloat类似 作用域是field

### 集合 set

#### 常用命令

- **sadd**
  - sadd key element [element …] 添加元素，返回结果为添加成功元素的个数
- **srem**
  - srem key element [element …] 删除元素， 返回结果为成功删除元素的个数
- **smembers**
  - smembers key 获取所有元素
- **sismembers**
  - sismembers key element 判断元素是否在集合内，如果给定元素element在集合内就返回1 否则返回0
- **scard** 
  - scard key 计算元素个数
- **spop**
  - spop key count 从集合中随机弹出元素 count指定弹出个数
- **srandmember**
  - srandmember key [count] 随机从集合返回指定个数元素 
- **sinter**
  - sinter key [key …] 求多个集合交集
- **suinon**
  - suinon key [key …] 求多个集合并集
- **sdiff**
  - sdiff key [key …] 求多个集合差集

### 有序集合 zset

#### 常用命令

- **zadd**
  - zadd key score member [score member …] 添加成员， 返回结果代表成功添加成员的个数，3.2版本后添加nx xx ch incr四个选项
    - nx ： member必须不存在 才能设置成功 用于添加
    - xx ： member 必须存在，才能设置成功 用于更新
    - ch ： 返回此次操作后，有序集合元素和分数发生变化的个数
    - incr ： 对socre做增加，相当于后面介绍的zincrby
- **zcard**
  - zcard key 计算成员个数
- **zscore**
  - zscore key member 计算某个成员的分数
- **zrank** **zrevrank**
  - zrank key member 计算成员的排名
- **zrem**
  - zrem key member 删除成员
- **zincrby**
  - zincrby key increment member 增加成员的分数
- **zrange** **zrevrange**
  - zrange key start end [withscores] 返回指定排名范围的成员，zrange是从低到高返回
- **zrangebyscore**
  - zrangebyscore key min max [withscores] [limit offset count] 返回指定分数范围的成员
- **zcount**
  - zcount key min max 返回指定分数范围的成员个数
- **zremrangebyrank**
  - Zremrangebyrank key start end 删除指定排名内的升序元素
- **zremrangebyscore**
  - zremrangebyscore key min max 删除指定分数范围的成员
- **zinterstore** 和 **zunionstore**  求集合的交集和并集