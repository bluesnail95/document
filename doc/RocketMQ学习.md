消息中间件有很多，比如RabbitMQ,RocketMQ之类的，比较了RabbitMQ和RocketMQ的书籍之后，自己选择了RocketMQ。

### 一：RocketMQ的安装

http://rocketmq.apache.org/dowloading/releases 

### 二：RocketMQ的概念

#### 1.RocketMQ的角色

|角色名称|作用|
|-------|----|
|Producer|生产者|
|Consumer|消费者|
|Broker|存储，传输消息的中介|
|NameServer|协调Broker的管理机构|

#### 2.RocketMQ的主题和队列

|名称|作用|
|-------|----|
|Topic|不同的消息类型以不同的Topic名称区分。|
|Message Queue|一个Topic可以设置一个或多个Message Queue发送。|

### 三：RocketMQ双主从的配置

#### 1.修改bin/runserver.sh和bin/runbroker.sh默认配置的内存:

- bin/runserver.sh

修改前最大堆和最小堆配置的是4G，年轻代是2G

修改前：JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"

修改后：JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"

- bin/runbroker.sh

修改前：JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"

修改后：JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx256m -Xmn256m"

参考：

https://blog.csdn.net/u012246342/article/details/80903129

https://www.jianshu.com/p/ca3a87bed2c2

#### 2.开启namesrv

启动命令：nohup sh bin/mqnamesrv & 

#### 3.配置broker的properties文件（用自己的两台机器139.199.210.171和193.112.47.238）

(1)在139.199.210.171上的配置(rocketmq是我的文件目录，配置文件名称可以自定义)

a.rocketmq/conf/2m-2s-sync/broker-a.properties:作为broker-a主节点的配置文件

```
brokerClusterName=cluster1
brokerName=broker-a
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH
listenPort=10911
storePathRootDir=/usr/download/rocketmq/store-a
namesrvAddr=139.199.210.171:9876;193.112.47.238:9876
```

b.rocketmq/conf/2m-2s-sync/broker-b-s.properties:作为broker-b从节点的配置文件

```
namesrvAddr=139.199.210.171:9876;193.112.47.238:9876
brokerClusterName=cluster1
brokerName=broker-b
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
listenPort=11011
storePathRootDir=/usr/download/rocketmq/store-b
```

(2)在193.112.47.238上的配置

a.rocketmq/conf/2m-2s-sync/broker-b.properties:作为broker-b主节点的配置文件

```
namesrvAddr=139.199.210.171:9876;193.112.47.238:9876
brokerClusterName=cluster1
brokerName=broker-b
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH
listenPort=10911
storePathRootDir=/usr/download/rocketmq/store-b
```

b.rocketmq/conf/2m-2s-sync/broker-b-s.properties:作为broker-a从节点的配置文件

```
namesrvAddr=139.199.210.171:9876;193.112.47.238:9876
brokerClusterName=cluster1
brokerName=broker-a
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
listenrPort=11011
storePathRootDir=/usr/download/rocketmq/store-a
```

#### 4.启动broker

进入到rocketmq的目录文件夹，执行：nohup sh bin/mqbroker -c conf/2m-2s-sync/xxx.properties &

根据3的配置 broker-a master --> broker-a slave --> broker-b master --> broker-b slave

#### 5.关闭broker和namesrv

sh bin/shutdown broker

sh bin/shutdown namesrv

#### 6.rocketMQ console的可视化监控

参考：http://www.cnblogs.com/buyige/p/9437054.html

#### 注意需要在配置文件application.properties文件中修改namesrvAddr的地址。

四.rockctmq常用的命令

### 进入rocketmq安装目录，运行实例：sh bin/mqadmin updateTopic -b 139.199.210.171:10911 -n 139.199.210.171:9876 -t topic1

|命令|解释|
|----|---|
|updateTopic|创建或是修改一个Topic|
|deleteTopic|删除Topic|
|updateSubGroup|创建/修改订阅组|
|deleteSubGroup|删除订阅组|
|updateBrokerConfig|更新broker配置|
|updateTopicPerm|更新Topic的读写权限|
|TopicRoute|查询Topic的路由信息|
|TopicList|查询Topic列表信息|
|TopicStats|查询Topic统计信息|
|printMsg|根据时间查询消息|
|queryMsgById|根据消息ID查询消息|
|clusterList|查看集群消息|

参考资料：

《RocketMQ实战与原理解析》

官方文档：http://rocketmq.apache.org

