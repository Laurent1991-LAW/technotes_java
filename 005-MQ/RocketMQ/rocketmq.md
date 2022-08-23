# RocketMQ



## 一、集群配置

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



## 二、配置文件与启动

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
# 进入根目录，cd $ROCKETMQ_HOME, 启动命名服务器
nohup sh bin/mqnamesrv &

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



## 三、操作步骤

### topic创建

![20200711002829639](E:\doc_repo\005-MQ\RocketMQ\images\20200711002829639.png)



`perm` 参数是设置队列的读写权限，下面表格列出了可配置的值及其含义：

| 取值 | 含义         |
| ---- | ------------ |
| 6    | 同时开启读写 |
| 4    | 禁写         |
| 2    | 禁读         |

至此，已经在两个broker中创建出6读6写，也可以修改Topic1分别配置 broker-a 和 borker-b 上的队列数量。

![20200711002512809](E:\doc_repo\005-MQ\RocketMQ\images\20200711002512809.png)



### 负载均衡

**生产者**：以**轮询**的方式向所有写队列发送消息，这些队列可能会分布在多个broker实例上

**消费者**：

>  AllocateMessageQueueAveragely 平均分配
>
> AllocateMessageQueueAveragelyByCircle 环形分配
>
> AllocateMessageQueueConsistentHash 一致性哈希

**springboot API 操作**：

`consumer.setAllocateMessageQueueStrategy(new AllocateMessageQueueConsistentHash()); `

`consumer.setAllocateMessageQueueStrategy(new AllocateMessageQueueAveragelyByCircle()); `





### 原生API实现差异



**同步消息**：producer.send() 直接传入message对象即可；



**异步消息**：producer.send() 需要传入message对象 及 new SendCallback() 回调对象 —— 重写**内部的处理成功 onSuccess() 与出错如何处理 onThrowable()** ；



**延时消息**：msg.setDelayTimeLevel(4) 消息对象定义其延时时间；



**顺序消息**：producer.send() 传入三参数

>message对象
>
>
>
>new MessageQueueSelector() {} 重写内部select方法，参数包括select (List\<MessageQueue> list, Message message, Object o) —— object为下方的参数，比如orderId
>
>
>
>Object对象 -> 队列选择依据，比如订单id，一个订单用一个队列

```java
SendResult r = p.send(msg, new MessageQueueSelector() {
                @Override
                public MessageQueue select(List<MessageQueue> list, Message message, Object o) {
                    Long orderId = (Long) o;
                    long index = orderId % list.size();
                    System.out.println("消息已发送到：" + list.get((int) index));

                    return list.get((int) index);
                }
            }, orderId);
```





**事务消息**：

- 创建TransactionMQProducer ；
- producer.setTransactionListener() 设置事务监听器，传入的TransactionListener重写两个方法：
  - 本地事务执行 executeLocalTransaction()，根据执行情况返回枚举值LocalTransactionState：
    - COMMIT_MESSAGE
    - ROLLBACK_MESSAGE
    - UNKNOW
  - 回查方式 checkLocalTransaction



### 参数与属性

> **生产方 + 消费方**

**setNamesrvAddr();**

**start();**



> **生产者**

**Message对象三参数：**topic 、 tag、 byte[]格式的消息；

**send() 参数：**

- 同步消息、延时消息：message；
- 异步消息：message + sendCallBack() 回调函数；
- 顺序消息：message + MessageQueueSelector对象，内部重写select方法 + 选择依据Object，如OrderId



> **消费者**

**subscribe() ：**

- topic + tag (可有多个)，如 `consumer.subscribe("Topic1", "TagA || TagB || TagC");`
- topic + MessageSelector；

**consumer.registerMessageListener()**：

​	MessageListener接口两种实现 :

- MessageListener**Concurrently**，需返回ConsumeConcurrentlyStatus，保证消费成功 或 consume later；

- MessageListener**Orderly**，需返回ConsumeOrderlyStatus，保证消费成功 或 suspend current queue a moment；





### 消息自定义属性

```java
// 生产方
msg.putUserProperty("prop1", "1");
msg.putUserProperty("prop2", "2");

// 消费方
consumer.subscribe("Topic7", MessageSelector.bySql("prop1=1 or prop2=2"));
```





## 四、消息类型



### 同步消息

**概念：**发送端在发送消息时，**阻塞线程进行等待，直到服务器返回发送的结果，** 适用于对消息可靠性要求比较高的项目。发送端如果需要保证消息的可靠性，防止消息发送失败，可以采用同步阻塞式的发送，然后同步检查Brocker返回的状态来判断消息是否持久化成功。如果发送超时或者失败，则会默认重试2次，RocketMQ选择至少传输成功一次的消息模型

 

### 异步消息

**概念：** 

一条消息送出后, 不必暂停等待服务器针对这条消息的反馈, 而是可以立即发送后续消息，使用监听器, 以异步的方式接收服务器的反馈 ；

发送端在发送消息时，传入 **回调接口实现类**，调用该发送接口后不会阻塞，发送方法会立即返回，**回调任务会在另一个线程中执行，消息发送结果会回传给相应的回调函数**。具体的业务实现 可根据发送的结果信息 来判断 是否需要重试 来保证消息的可靠性。 



### 延时消息

**概念：**消息发送到 Rocketmq 服务器后， 延迟一定时间再向消费者进行投递。

**使用场景：**比如电商里，提交了一个订单就可以发送一个延时消息，1h后去检查这个订单的状态，如果还是未付款就取消订单释放库存。

> msg.setDelayTimeLevel(3);  —— 其中 `3` 代表级别而不是一个具体的时间值，级别和延时时长对应关系是在 `MessageStoreConfig` 类种进行定义的： 

**对应关系表：**

| 级别 | 延时时长 |
| :--: | :------: |
|  1   |    1s    |
|  2   |    5s    |
|  3   |   10s    |
|  4   |   30s    |
|  5   |    1m    |
|  6   |    2m    |
|  7   |    3m    |
|  8   |    4m    |
|  9   |    5m    |
|  10  |    6m    |
|  11  |    7m    |
|  12  |    8m    |
|  13  |    9m    |
|  14  |   10m    |
|  15  |   20m    |
|  16  |   30m    |
|  17  |    1h    |
|  18  |    2h    |



### 顺序消息

以订单为例，一个订单的顺序流程是：创建、付款、推送、完成。带有一个订单号的消息 会被 先后发送 到 同一个队列中。消费时，从同一个队列接收同一个订单的消息。

![20200715180447730](E:\doc_repo\005-MQ\RocketMQ\images\20200715180447730.png)





### 事务消息



**流程：**

1. 发送半消息（半消息不会发送给消费者）；
2. 执行本地事务；
3. 事务成功提交消息 或 事务失败回滚消息；



> **Producer ----1----> MQ ----2----> Consumer **

1. 若P没向MQ发送回滚或提交操作，MQ会向P**回查**——每隔1分钟询问事务执行情况；
2. MQ向C有消息自动重发机制，在绝大多数情况下，都可以保证消息被正确消费，即使最终失败也可**由人工处理进行托底**。

 ![无标题](E:\doc_repo\005-MQ\RocketMQ\images\无标题.png)





## 五、常见问题

### RocketMQ服务器分类与功能是什么？

> nameserver 

- 轻量级的注册中心，最初为zk，但没必要维护如此大体量的注册中心——杀鸡焉用牛刀 ；
- 存储topic中的路由信息、broker关系信息，功能简单，稳定性高 ；
- 各个NameServer节点之间不相关，无需通信，单台宕机不影响其它节点。

> broker

- 消息队列，信道的载体 ;
- 每个Borker和所有NameServer保持长连接，心跳间隔为30秒。每次心跳时还会携带当前的Topic信息。当某个Broker两分钟之内没有心跳，则认为该Broker下线，并调整内存中与该Broker相关的Topic信息；
- Consumer 从 NameServer 获得 Topic 的路由信息，与对应的 Broker 建立长连接。间隔30秒发送心跳至Broker。Broker检查若发现某 Consumer 两分钟内无心跳则认为该Consumer下线，并通知该Consumer所有的消费者集群中的其他实例，触发该消费者集群重新负载均衡。
  

![v2-c8f373eb6b6c53e9e7e75c5b262163db_720w](E:\doc_repo\005-MQ\RocketMQ\images\v2-c8f373eb6b6c53e9e7e75c5b262163db_720w.jpg)



### 消息重试与死信队列是什么？

消费者从RocketMQ拉取到消息之后，需要返回消费成功来表示业务方正常消费完成。因此只有返回 CONSUME_SUCCESS 才算消费完成，如果返回 CONSUME_LATER 则会按照不同的 messageDelayLevel 时间进行再次消费，时间分级从秒到小时，最长时间为2个小时后再次进行消费重试，如果**消费满16次之后**还是未能消费成功，则不再重试，会将消息发送到死信队列，从而保证消息存储的可靠性。

未能成功消费的消息，消息队列并不会立刻将消息丢弃，而是将消息发送到死信队列，其名称是在原队列名称前加%DLQ%，如果消息最终进入了死信队列，则可以通过RocketMQ提供的相关接口从死信队列获取到相应的消息，保证了消息消费的可靠性。 

 



### 使用过RocketMQ的事务消息吗？





###  延时队列如何实现 ？





### 顺序队列使用在什么场景中 ？

 

### 消息积压如何解决？





### RocketMQ如何保证消息可靠性？

