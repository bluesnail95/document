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

### 4.list

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

### 5.set

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
         
        
### 6.zset

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
  
### 7.Jedis对五种数据类型的操作

```
Jedis jedis = null;
try {
			
	   jedis = new Jedis("127.0.0.1", 6379, 10000);
			
	   //1.string
	   String result1 = jedis.set("string1", "value1");
	   String result2 = jedis.get("string1");
	   System.out.println(result1);//OK
	   System.out.println(result2);//value1
			
	   //2.list
	   long result3 = jedis.lpush("list1", "math","math","score","score","name","xiaoming");
	   List<String> result4 = jedis.lrange("list1", 0, -1);
	   System.out.println(result1);//OK
	   System.out.println(result4);//xiaoming, name, score, score, math, math
			
	   //3.hash
	   jedis.hset("hash1", "subject","math");
	   jedis.hset("hash1", "score","99");
	   jedis.hset("hash1", "name","xiaoming");
	   List<String> result5 = jedis.hmget("hash1", "subject","score","name");
	   System.out.println(result5);//[math, 99, xiaoming]
			
	   //4.set
	   jedis.sadd("set1", "math","math","english","chinese");
	   jedis.sadd("set2", "math","chinese","art");
	   jedis.sinterstore("set3", "set1","set2");
	   System.out.println(jedis.smembers("set3"));//[math, chinese]
			
	   //5.zset
	   jedis.zadd("zset1", 100, "math");
	   jedis.zadd("zset1", 200, "chinese");
	   jedis.zadd("zset1", 300, "english");
	   Set<String> result6 = jedis.zrangeByScore("zset1", 100, 200);
	   result6.forEach(string -> {
		      System.out.print(string+" ");
	   });//math chinese 
			
}catch(Exception e) {
	   e.printStackTrace();
}finally {
	   if(jedis != null) {
		      jedis.close();
	   }
}

```

## 四.客户端操作

### 1.client list

列出与Redis服务器相连的所有客户端信息。

![client list](../img/Redis/client_list.jpg)

属性如下：

|名称|含义|
|:---|:----|
|id|客户端的唯一标识。自增，重启后重置为0。|
|addr|客户端连接的地址和端口。|
|fd|socket的文件描述符。|
|name|客户端的名称。|
|age|当前客户端的连接时间。|
|idle|当前客户端的最近一次空闲时间。当age等于idle表示连接一直处于空闲状态。|
|flags|标识当前客户端的类型。|
|db|当前客户端正在使用的数据库索引下标。|
|sub|当前客户端订阅的频道或者模式数。|
|psub|当前客户端订阅的频道或者模式数。|
|multi|当前事务中已执行命令个数。|
|qbuf|输入缓冲区总容量。|
|qbuf-free|输入缓冲区的剩余容量。|
|obl|输出缓冲区的固定缓冲区的大小。|
|oll|输出缓冲区的动态缓冲区的大小。|
|omem|输出缓冲区使用的字节数。|
|events|文件描述符事件。|
|cmd|当前客户端最后一次执行的命令。|

## 2.输入缓冲区

作用：客户端发送的命令不是直接发送给Redis服务器，而是先存放在输入缓冲区，Redis服务器从输入缓冲区中获得命令并执行。

当输入缓冲区的输入速度大于Redis服务器的处理速度且存在大量的bigkey或是Redis服务器发生阻塞，短期不能执行命令时，都会造成输入缓冲区过大，可以通过client list查看qbuf和qbuf-free的大小或是通过info clients命令找到最大的输入缓冲区。

![info clients](../img/Redis/info_clients.jpg)

### 3.输出缓冲区

作用：Redis服务器执行命令后的结果不是直接返回给客户端，而是先存放在输出缓冲区。

输出缓冲区分为固定缓冲区和动态缓冲区，固定缓冲区是字节数组，动态缓冲区是列表，固定缓冲区使用完之后才会使用动态缓冲区。

通过client list和info clients可以监控输出缓冲区的异常情况。

### 4.客户端的分类

(1)普通客户端

(2)发布订阅客户端

(3)slave客户端

### 5.客户端操作

|命令|含义|
|:----|:----|
|config set maxclients value  |设置最大连接数 |
|config get maxclients |设置最大连接数 |
|info clients |查看当前已经连接的客户端数量|
|config set timeout value |设置超时时间，空闲时间一旦大于超时时间，客户端连接就会自动断开。|
|client setName value |设置客户端的名称|
|client getName |获得客户端的名称|
|client kill ip:port |关闭指定的ip:port的客户端|
|client pause timeout |(时间单位毫秒) 阻塞客户端timeout毫秒|

## 五.持久化

### 1.RDB
(1)概念：将当前线程数据生成快照保存在磁盘中。

(2)方式

a.手动触发

bgsave命令：Redis进程执行fork操作创建子进程，RDB的序列化由子进程完成，在fork阶段会出现堵塞。

b.自动触发

在某些情况下自动触发bgsave命令或是save命令。

(3)RDB的优缺点

a.优点

紧凑压缩的二进制文件，能够代表Redis在某个时间点的数据备份，可复制到不同的机器进行灾难恢复。Redis加载RDB恢复数据的速度快于AOF。

b.缺点

无法实现实时持久化，执行fork操作创建子进程是重量级操作，频繁执行成本较高，且老版本的Redis服务无法兼容新版本的RDB格式文件。

### 2.AOF

(1)概念：记录每次的写命令，重启后执行AOF文件中的命令以达到恢复数据的目的。可以用aof_enabled开启aof功能。

(2)特点：

a.AOF命令以文本协议格式的形式写入内容到aof_buf中，再由aof_buf同步到硬盘中。文本协议格式具有很好的兼容性以及避免了二次处理的开销。而写入到aof_buf中是为了避免直接写入硬盘，以免硬盘的容量决定了追加写入的性能。

b.aof重写将无效的命令如del去掉，将多个命令合并成一个命令，以达到压缩文件体积，加快Redis加载aof文件的速度。

参考资料：

《Redis开发与运维》
