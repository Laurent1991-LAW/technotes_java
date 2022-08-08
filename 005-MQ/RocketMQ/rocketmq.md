# RocketMQ



## 集群配置

### 集群方案

![20200709000005103](E:\doc_repo\005-MQ\RocketMQ\images\20200709000005103.png)



在 `rocketmq/conf` 目录下提供了四种集群方案的配置样例

- 2m-2s-async：双主双从异步复制，消息发到主服务器就向生产者发回反馈信息，之后再向从服务器复制
- **2m-2s-sync：双主双从同步复制，消息复制到从服务器后才向生产者发回反馈信息**
- 2m-noslave：双主
- dledger： raft主从切换

一般选择**双主双从同步复制**方案。



### 配置要点说明 

- 四台服务器的集群名 brokerClusterName 相同。集群名称相同的服务器共同组成服务集群 。

- 从服务器通过名字与主服务器关联在一起，brokerName 与主服务器相同。

- brokerId为0是主服务器。从服务器的值是非零值，例如如果有四个从服务器，他们的 brokerId 应该是 1, 2, 3, 4。

- brokerRole的值为 SYNC_MASTER 是同步复制的主服务器。如果是 ASYNC_MASTER 则为异步复制的主服务器。

  

![Snipaste_2022-08-08_11-34-21](E:\doc_repo\005-MQ\RocketMQ\images\Snipaste_2022-08-08_11-34-21.png)



## 配置文件与启动

### 准备工作

创建 数据存储目录 与 日志存储目录

```
mkdir /usr/local/rocketmq/store/
mkdir /usr/local/rocketmq/store/broker-a
mkdir /usr/local/rocketmq/store/broker-a/commitlog
mkdir /usr/local/rocketmq/store/broker-b
mkdir /usr/local/rocketmq/store/broker-b/commitlog
mkdir /usr/local/rocketmq/store/broker-as
mkdir /usr/local/rocketmq/store/broker-as/commitlog
mkdir /usr/local/rocketmq/store/broker-bs
mkdir /usr/local/rocketmq/store/broker-bs/commitlog
```



### 主服务器配置

在**服务器1**修改 样例配置文件：`rocketmq/conf/2m-2s-sync/broker-a.properties`

在**服务器2**修改 样例配置文件：`rocketmq/conf/2m-2s-sync/broker-b.properties`

在样例配置文件中，添加最下三项配置：

```properties
brokerClusterName=DefaultCluster
# broker-b.propertiesw为broker-b
brokerName=broker-a 
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH

# ------
listenPort=10911
# broker-b.propertiesw为broker-b
storePathRootDir=/usr/local/rocketmq/store/broker-a
storePathCommitLog=/usr/local/rocketmq/store/broker-a/commitlog
```



### 从服务器配置

在**服务器1 **修改样例配置文件：`rocketmq/conf/2m-2s-sync/broker-b-s.properties` 

在**服务器2** 修改样例配置文件：`rocketmq/conf/2m-2s-sync/broker-a-s.properties` 

```properties
brokerClusterName=DefaultCluster
# broker-b-s.propertiesw为broker-b
brokerName=broker-a
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH

listenPort=11911
# broker-b-s.properties为broker-bs
storePathRootDir=/usr/local/rocketmq/store/broker-as
storePathCommitLog=/usr/local/rocketmq/store/broker-as/commitlog
```



### 启动语句

- 在两台服务器上启动两个 name server，它们不用做任何集群的配置，都是**作为独立服务运行，它们之间也不会进行数据复制**；
- 所有broker服务启动后，要同时连接这两个 name server，向两个 name server 进行注册;
- 参数说明：
  - **-n参数**：指定name server地址列表，多个地址用分号分隔
  - **-c参数**：指定配置文件，使用指定的配置文件启动 broker

```shell
# 控制台启动
nohup java -jar rocketmq-console-ng-1.0.1.jar \
--server.port=8080 \
--rocketmq.config.namesrvAddr='192.168.64.192:9876;192.168.64.141:9876' \
&

# A服务器上启动：A-master 与 B-slave（区别在于指定不同配置文件）
nohup sh mqbroker \
-n '192.168.64.141:9876;192.168.64.192:9876' \
-c $ROCKETMQ_HOME/conf/2m-2s-sync/broker-a.properties \
&

nohup sh mqbroker \
-n '192.168.64.141:9876;192.168.64.192:9876' \
-c $ROCKETMQ_HOME/conf/2m-2s-sync/broker-b-s.properties \
&


# B服务器上启动：B-master 与 A-slave（区别在于指定不同配置文件）
nohup sh mqbroker \
-n '192.168.64.141:9876;192.168.64.192:9876' \
-c $ROCKETMQ_HOME/conf/2m-2s-sync/broker-a-s.properties \
&

nohup sh mqbroker \
-n '192.168.64.141:9876;192.168.64.192:9876' \
-c $ROCKETMQ_HOME/conf/2m-2s-sync/broker-b.properties \
&
```



### 查看控制台

![20200709172705572](E:\doc_repo\005-MQ\RocketMQ\images\20200709172705572.png)