## 一.Redis是什么。

Redis是一个Key-Value的NoSQL数据库.

## 二.Redis的特点。

### 1.支持的数据结构：

hash,list,set,zset,string（memacached只支持string）。

### 2.单线程执行命令。

因为是单线程，所以减少了线程上下文切换的开销，同时如果一个命令执行时间过长就会引起阻塞。

### 3.数据存储

数据持久化到内存中，一定时间后会存储到磁盘中。

## 三.操作数据的命令

### 1.常用的命令

|命令|含义|
|:--------|:---|
|keys *|查看全部的键，会遍历Redis所有的键，时间复杂度是O(n)|
|scan cursor [match pattern][count number]|遍历键，cursor是游标|
|type key|查看键的类型,key是键的名称|
|dbsize |查看键的数量：dbsize 是直接获取Redis内置的键总量，时间复杂度是O(1)|
|exists key|判断某个键是否存在，存在返回1，不存在返回0.|
|del key[ key...]|返回成功删除键的个数|
|expire key time|设置键的过期时间|
|ttl key|查询某个键的剩余过期时间|
|object encoding key|查询键的内部编码|
|rename key newkey|键重命名|
|renamenx key newkey|当newkey不存在，键重命名成功|
|randomkey|随机选择一个键|
|persist key|清除键的过期时间|
|move key db|在Redis内部进行数据库迁移|
|dump + restore|在不同Redis实例间迁移数据|
|migrate|在数据库实例间迁移数据|

### 2.String数据结构

|命令|含义|
|:---|:---|
|set key value |插入键值对，key是键，value是值|
|get key |查看键的值|
|del key |删除键|
|setnx key value |当key不存在时，设置值|
|setex key seconds value |seconds是过期时间，设置键值对|
|mset key value[key value..]|批量获取值|
|mget key [key ...]|批量获取值|
|incr key|对值做自增1|
|decr key|对值做自减1|
|incrby key incrment|自增指定的数目 increment 数字|
|decrby key incrment|自减指定的数目 increment 数字|
|incrbyfloat key incrment|自增指定的浮点数 increment 数字|

内部编码有三种：int,embstr和raw

使用场景：
 - setnx和setex可用于分布式锁 
 
 - incr等可以用于计数
 
 - 统一管理用户的session
         
### 3.hash  

|命令|含义|
|:---|:---|
|hset key field value|设置hash的内容key=[{field:value}{field:value}]|
|hget key field|获取字段值|
|hdel key field|删除字段值|
|hlen key|获取key的字段数|
|hmset key field value [field value...]|批量设置key的field-value|       
|hmget key field1[field2...]|批量获得key的field的字段值|
|hexists key field|判断key的field是否存在|
|hkeys key|获取key的全部字段|
|hvals key|获取key的全部value值|
|hgetall key|获取key的全部field,value|
|hincrby key field incrment|key的字段field自增increment|
|hincrbyfloat key field increment|key的字段field自增浮点数increment|
|hstrlen key field|计算field的value的长度|

内部编码：ziplist和hashtable

### 3.list

|命令|含义|
|:---|:---|
|rpush key value[value...]|从列表右边添加元素|
|lpush key value[value...]|从列表左边添加元素|
|lrange key start end|获取指定索引范围的元素，0表示第一个，-1表示最后一个|
|linsert key before/after pivot value|在pivot元素前/后插入value元素|
|lindex key index|获取列表指定下标的元素|
|llen key|获取列表的长度|
|lpop key|从列表的左侧弹出元素|
|rpop key|从列表的右侧弹出元素|
|lrem key count value|从左到右删除count个值为value的元素|
|lset key index value|设置index位置的值|
|brpop/blpop key timeout|阻塞弹出,timeout是超时时间，0表示一直等待下去|

内部编码：ziplist(压缩列表)，linkedlist(链表)和quicklist

使用场景：

   - lpush+brpop=阻塞队列(消息队列)。
   
   - lpush+lpop=Stack(栈)
   
   - lpush+rpop=Queue(队列)
   
   - lpush+ltrim=Capped Collection(有限集合)

### 4.set

|命令|含义|
|:---|:---|
|sadd key element[element...]|添加元素|
|srem key element[element...]|删除元素|
|scard key|计算元素个数|
|sismember key element|判断element元素是否在集合中|
|srandmember key [count]|随机生成count个元素，默认是1个|
|spop key|随机弹出一个元素|
|smembers key|查询全部的元素|
|sinter key [key...]|查询多个集合的并集|
|sunion key [key...]|查询多个集合的交集|
|sdiff key [key...]|查询多个集合的差集|
|sinterstore destination key [key...]|查询多个集合的并集,存储到destination中|
|sunionstore destination key [key...]|查询多个集合的交集,存储到destination中|
|sdiffstore destination key [key...]|查询多个集合的差集,存储到destination中|

内部编码：intset,hashtable

使用场景：

  - sadd=Tagging(标签)
  
  - spop/srandmember=Random item(随机数抽奖)
  
  - sadd+sinter=Social Graph(社交需求)
         
        
### 5.zset

|命令|含义|
|:---|:---|
|zadd key score memeber[score memeber...]|添加成员|
|zcard key|计算成员个数|
|zscore key member|计算成员的分数|
|zrank/zrevrank key member|计算成员的排名|
|zrem key member[member...]|删除成员|
|zincrby key increment member|增加成员的分数|
|zrange/zrevrange key start end [withscores]|从低到高，返回指定排名范围的成员|
|zrangebyscore key min max [withscores] [limit offset count]|从低到高，返回指定分数范围的成员|
|zrevrangebyscore key max min [withscores] [limit offset count]|返回指定分数范围的成员|
|zcount key min max|返回指定范围的成员个数|
|zremrangebyrank key start end|删除指定排名内的升序元素|
|zremrangebyscore key min max|删除指定分数范围的成员|
|zinterstore destination numberkeys key [key...] [weights weight [weight...]] [aggregate sum/min/max]|两个有序集合的交集，numberkeys指有序集合进行交集的个数|
|zunionstore destination numberkeys key [key...] [weights weight [weight...]] [aggregate sum/min/max]|两个有序集合的并集，numberkeys指有序集合进行并集的个数|

内部编码：ziplist(压缩列表)和skiplist(跳跃表)

使用场景：

  - 排行榜(点赞)

参考资料：

《Redis开发与运维》
