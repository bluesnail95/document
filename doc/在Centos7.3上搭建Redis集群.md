### 1.下载Redis

下载地址：http://download.redis.io/releases/redis-5.0.0.tar.gz

### 2.手动构建Redis集群

输入的命令如下:

```
#进入/usr/local/src文件夹
$cd /usr/local/src

#开始下载
$wget http://download.redis.io/releases/redis-5.0.0.tar.gz

#解压缩
$tar zxf redis-5.0.0.tar.gz

#建立redis目录的软链接
$ln -s redis-5.0.0 redis

$make

$make install

#查询reids的版本
$redis-cli -v

#进入redis文件夹
$cd redis

#查看文件夹的内容
$ls

#建立data,logs,conf文件夹
$mkdir data
$mkdir logs
$mkdir conf

#复制一份配置文件到conf中
$cp redis.conf conf/redis-6380.conf

#编辑redis-6380.conf文件
$vi conf/redis-6380.conf

修改以下属性:
port 6380
logfile "/usr/local/src/redis/logs/redis-6380.log"
daemonize yes
cluster-enabled yes
cluster-node-timeout 15000
cluster-config-file "nodes-6380.conf"

vi编辑器按i进入编辑，按ESC键退出编辑，:q退出 :wq保存退出

$cd conf

#复制另外5份配置文件，将redis-6381.conf文件中的6380修改为6381，其他文件类似。
$cp redis-6380.conf redis-6381.conf
$cp redis-6380.conf redis-6382.conf
$cp redis-6380.conf redis-6383.conf
$cp redis-6380.conf redis-6384.conf
$cp redis-6380.conf redis-6385.conf

#退回到/usr/local/src/redis文件夹
$cd ../

#启动
$src/redis-server conf/redis-6380.conf
$src/redis-server conf/redis-6381.conf
$src/redis-server conf/redis-6382.conf
$src/redis-server conf/redis-6383.conf
$src/redis-server conf/redis-6384.conf
$src/redis-server conf/redis-6385.conf

#查看日志文件信息
$cat logs/redis-6380.log

#登录redis6380客户端
$src/client-cli -h 127.0.0.1 -p 6380

#查看集群信息，目前集群是创建失败的，因为各个节点还没有联系起来
127.0.0.1:6380>cluster info

#查看各个节点信息
127.0.0.1:6380>cluster nodes

#将各个节点联系起来
127.0.0.1:6380>cluster meet 127.0.0.1:6381
127.0.0.1:6380>cluster meet 127.0.0.1:6382
127.0.0.1:6380>cluster meet 127.0.0.1:6383
127.0.0.1:6380>cluster meet 127.0.0.1:6384
127.0.0.1:6380>cluster meet 127.0.0.1:6385

#分配槽 注意这里是大括号和两个点
$src/redis-cli -h 127.0.0.1 -p 6380 cluster addslots {0..5461}
$src/redis-cli -h 127.0.0.1 -p 6381 cluster addslots {5462..10922}
$src/redis-cli -h 127.0.0.1 -p 6382 cluster addslots {10923..16383}

#从节点复制主节点 cluster replicate {nodeId} nodeId是要复制主节点的节点ID
127.0.0.1:6383>cluster replicate {主节点的节点ID}
127.0.0.1:6384>cluster replicate {主节点的节点ID}
127.0.0.1:6385>cluster replicate {主节点的节点ID}

#再次查询集群状态，显示成功
127.0.0.1:6385> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:5
cluster_my_epoch:2
cluster_stats_messages_ping_sent:970
cluster_stats_messages_pong_sent:1020
cluster_stats_messages_meet_sent:3
cluster_stats_messages_sent:1993
cluster_stats_messages_ping_received:1018
cluster_stats_messages_pong_received:973
cluster_stats_messages_meet_received:2
cluster_stats_messages_received:1993

#查询集群的节点状态，三主三从
127.0.0.1:6385> cluster nodes
9f64837ca8869f2ca9b8977774de191b9f3dc0fe 127.0.0.1:6385@16385 myself,slave 3d0652d11190595c6d4e360ce2e64f64d864c476 0 1541238728000 5 connected      
3c41ec04ad2186058a66d857f3525b8228f3328f 127.0.0.1:6381@16381 master - 0 1541238729000 1 connected 5462-10922
3d0652d11190595c6d4e360ce2e64f64d864c476 127.0.0.1:6382@16382 master - 0 1541238727000 2 connected 10923-16383
b76258c9ab37d31d8ac087e96631d0983ab5ef3d 127.0.0.1:6383@16383 slave ef7eaafc183d473ca6bac90758a8b951b5e0158f 0 1541238729994 3 connected 
b0b991431a893d6dfd09623bd2aab2b29e045fa1 127.0.0.1:6384@16384 slave 3c41ec04ad2186058a66d857f3525b8228f3328f 0 1541238730000 4 connected 
ef7eaafc183d473ca6bac90758a8b951b5e0158f 127.0.0.1:6380@16380 master - 0 1541238730997 0 connected 0-5461

```

### 3.利用redis-cli构建Redis集群

```
#在创建集群前需要各个节点建立联系(cluster meet {ip} {port})
$redis-cli --cluster create 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 --cluster-replicas 1
```

搭建过程出现以下错误：

```
[ERR] Node 127.0.0.1:6380 is not empty. Either the nodealready knows other nodes (check with CLUSTER NODES) or contains some key in database 0.
```

解决方案：https://blog.csdn.net/xiaoliuliu2050/article/details/72898828  

删除rdb,aof文件命令：

```
find . -name '*.rdb' -type f -print -exec rm -rf {} \;
find . -name '*.aof' -type f -print -exec rm -rf {} \;
```

如果在删除cluster-config-file设置的文件，删除备份文件，flushdb之后都没有用，可以采取以下措施：
```
127.0.0.1:6379>cluster reset
127.0.0.1:6380>cluster reset
127.0.0.1:6381>cluster reset
127.0.0.1:6382>cluster reset
127.0.0.1:6383>cluster reset
127.0.0.1:6384>cluster reset
```

### 4.设置集群的密码

```
#对6380,6381,6382,6383,6384,6385进行如下操作
$config set masterauth {password}
$config set requirepass {password}
$config rewrite

#将设置的密码清空(也就是不设置密码)
$config set masterauth ""
$config set requirepass ""
```

### 5.Jedis连接redis集群

用的是Jedis2.9.0版本

```
@Test
public void testSetString() {
    JedisPoolConfig config = new JedisPoolConfig();
    config.setMaxTotal(10);
    config.setMaxIdle(5);
    config.setMaxWaitMillis(3000);
    config.setTestOnBorrow(true);
    config.setTestOnReturn(true);

    Set<HostAndPort> set = new HashSet<HostAndPort>();
    set.add(new HostAndPort("139.199.210.171",6380));
    set.add(new HostAndPort("139.199.210.171",6381));
    set.add(new HostAndPort("139.199.210.171",6382));
    set.add(new HostAndPort("139.199.210.171",6383));
    set.add(new HostAndPort("139.199.210.171",6384));
    set.add(new HostAndPort("139.199.210.171",6385));

    JedisCluster cluster = new JedisCluster(set,15000,10000,10,"xxxx",config);
    cluster.setnx("hello","tomorrow will be better");
    String result = cluster.get("hello");
    System.out.println(result);
    try {
        cluster.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

遇到的问题：

(1)Could not get a resource from the pool

解决方案：要设置成bind 139.199.210.171(设置成公网IP）

参考：https://www.cnblogs.com/webyyq/p/8934289.html

(2)JedisCluster进行连接的时候需要返回资源，进行关闭close()。

(3)在没有设置密码的时候，要设置protected-mode为no。

参考资料：

https://blog.csdn.net/u013063153/article/details/71191138

《Redis开发与运维》
