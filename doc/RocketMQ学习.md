消息中间件有很多，比如RabbitMQ,RocketMQ之类的，比较了RabbitMQ和RocketMQ的书籍之后，自己选择了RocketMQ。

一：RocketMQ的安装

http://rocketmq.apache.org dowloading releases 

二：RocketMQ的概念

1.RocketMQ的角色

(1)Producer：生产者

(2)Consumer: 消费者

(3)Broker: 存储，传输消息的中介

(4)NameServer: 协调Broker的管理机构

2.RocketMQ的主题和队列

(1)Topic：不同的消息类型以不同的Topic名称区分。

(2)Message Queue：一个Topic可以设置一个或多个Message Queue发送。

三：RocketMQ双主从的配置

1.修改bin/runserver.sh和bin/runbroker.sh默认配置的内存:

(1)bin/runserver.sh

修改前最大堆和最小堆配置的是4G，年轻代是2G

修改前：JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"

修改后：JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"

(2)bin/runbroker.sh

修改前：JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"

修改后：JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx256m -Xmn256m"

参考：

https://blog.csdn.net/u012246342/article/details/80903129

https://www.jianshu.com/p/ca3a87bed2c2

2.开启namesrv

启动命令：nohup sh bin/mqnamesrv & 

3.配置broker的properties文件

(1)在139.199.210.171上的配置(conf/2m-2s-sync下的properties文件)

a.broker-a.properties:作为broker-a的主节点

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

(2)在193.112.47.238上的配置(conf/2m-2s-sync下的properties文件)

a.broker-b.properties:作为broker-b的主节点

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

b.broker-b-s.properties:作为broker-a的从节点

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

4.启动broker

5.关闭broker和namesrv

6.rocketMQ console的可视化监控





参考资料：
《RocketMQ实战与原理解析》
官方文档：http://rocketmq.apache.org

