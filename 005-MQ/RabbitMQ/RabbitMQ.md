# RabbitMQ



### 消息队列基本步骤是什么？

1.创建连接工厂 -> 获取新连接 -> 创建信道 ;

2.队列声明（消费者与发布者均可完成，先到先建立），queueDeclare方法参数 :

- 队列名 ;
- 是否为持久化队列（是否将队列信息存储到磁盘） ;
- 是否为独占队列 ;
- 若无消费者，是否将自动删除 ;
- 队列其他参数 ;

3.发布动作basicPublish()参数 :

- 交换机名 ;
- 队列名称 ;
- 其他参数 : 键值对形式，如` MessageProperties.PERSISTENT_BASIC ` ;
- 消息内容，若为String，应转化为bytes[] ;

4.消费动作basicConsume() :

- 队列名 ;
- **是否开启自动确认acknowlage（是否需要消费回执）** ;
- 消息处理回调对象 ;
- 取消处理回调对象 ;



**注意 :** 消息的生产者不一定有队列，但消费者必然有队列，可能是生产者直接发到队列，也有可能是 发到交换机，前提是 消费者的队列已和交换机绑定



### 交换机有哪几种？

**fanoutExchange :** 广播模式交换，接收到生产者消息后即可播报，无论是否存在消费者 ;

**directExchange :** 订阅模式，消费者根据 队列的路由键 消费对应消息，队列可与多个key绑定 ;

**TopicExchange :** 主题模式，队列路由键具有特殊格式，如\*.\*.rabbit、lazy.# 可匹配多种键 ;





### 发布-订阅模式的基本步骤？

**生产者 :** 



1.声明交换机channel.exchangeDeclare() ;

- 交换机名
- 交换机类型，如BuiltinExchangeType.FANOUT 或 "fanout" ;



2.发布动作basicPublish()参数 :

- 交换机名 ;
- 队列名称（一般设为空字符串""——发布订阅模式下生产者直接向交换机发布消息，不经过队列，况且生产者一旦声明交换机，第二个参数则为 路由键） ;
- 其他参数 : 键值对形式，如` MessageProperties.PERSISTENT_BASIC ` ;
- 消息内容，若为String，应转化为bytes[] ;



**消费者 :**

1.声明队列channel.queueDeclare();  

2.声明交换机channel.exchangeDeclare() :

- 交换机名
- 交换机类型，如BuiltinExchangeType.FANOUT 或 "fanout" ;

3.队列与交换机绑定channel.queueBind(队列名，交换机名) ;

4.消费动作basicConsume() 。



### 主题模式的基本步骤？



**生产者 :** 

1.声明交换机channel.exchangeDeclare("xxx", BuiltinExchangeType.DIRECT) ;

2.发布动作basicPublish()参数：交换机名、**路由键**、消息其他属性、消息内容 ；



**消费者 ：**

1.声明队列 + 声明交换机 ；

2.关键词绑定 ： channel.queueBind(队列名、交换机名、**路由键**) ；

3.消费动作basicConsume()。



### SpringBoot整合RabbitMQ注解

**依赖 :**

```pom
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**主要注解包括 :**

- 简单、工作模式不涉及交换机的创建，因此在配置类中里用@Bean显示的创建队列 :

```java
	@Bean
    public Queue taskQueue() {
        return new Queue("task_queue"); //此处只给队列名，其他参数是下面的默认值
        // return new Queue("task_queue",true,false,false);
    }
```

- 广播、订阅、主题模式涉及交换机的创建，在配置类中里用@Bean显示的创建交换机 :

```java
	@Bean
    public TopicExchange logs() {
        // 广播模式为 FanoutExchange
        // 订阅模式为 DirectExchange
        return new TopicExchange("topic_logs", false, false);
        //交换机名、非持久，不自动删除
    }
```

- 生产者发布消息，需要**注入IoC容器内的AmqpTemplate，其内部方法convertAndSend为发布函数** :

```java
	@Autowired
    private AmqpTemplate amqpTemplate;

	amqpTemplate.convertAndSend(String name,String key，String message);
	//参数1: 队列名 或 交换机名
	//参数2: 路由键（可无）
	//参数3: 消息内容（无需如原生API中getBytes方法转换）
```

- 消费者消费消息，可在 类上加注解@RabbitListener(queues = "xxx") + 方法上@RabbitHandler ; 或者直接在方法上 @RabbitListener(queues = "xxx")

- 对于 广播、订阅、主题模式，在配置中创建了交换机，还必须在消费端进行**消费队列与交换机的绑定**，具体实现为配置 @RabbitListener的bindings参数

  - @RabbitListener的bindings参数 = @QueueBinding注解 :
    - value = @Queue(name="xxx", durable=false, exclusif=true, autoDelete=true)
    - exchange = @Exchange(name = "logs", declare = "false")
    - key = {"\*.\*.rabbit", "lazy.#"}

  ```java
  	@RabbitListener(bindings = @QueueBinding(
              //队列,若@Queue不附带任何参数，则为随机命名队列，默认false, true, true
              value = @Queue,
              // declare = "false" 不创建交换机 -> 使用已存在的交换机
              exchange = @Exchange(name = "logs", declare = "false"), 
      		key = {"*.*.rabbit", "lazy.#"}
      										)
                     )
      public void receive1(String msg) {
          System.out.println("消费者1收到：" + msg);
      }
  ```

  

### 如何保证MQ中的消息不丢失？

**保证消息不丢失三步走** 
1 开启事务 (不推荐)
2 开启confirm (推荐)
3 开启RabbitMQ持久化 (交换机、队列、消息)
4 关闭RabbitMQ自动ack (改成手动)



**1.确保消息在生产端能发送成功 :**

方法一 : 通过事务实现，但性能消耗较大 ;

方法二 : 一般通过confirm实现 :

```java
channel.confirmSelect(); // 将信道设置成confirm模式。

channel.waitForConfirms();
等待发送消息的确认消息，如果发送成功，则返回ture，如果发送失败，则返回false，比如:

if (channel.waitForConfirms()) {
            System.out.println("send message success");
        } else {
            System.out.println("send message failed");
        }
```



**2.队列持久化到磁盘 :** 在声明队列时开启，queueDeclare()第二个参数 ;



**3.确保消息在消费端能被消费 :** 

- basicConsume()方法中的第二个参数ack设为false（取消-消息送到消费端即自动确认为收到） ;
- 在**消息处理回调对象中，业务操作完成后，发送回执**: channel.basicAck(message.getEnvelope().getDeliveryTag(), false); 
- 设置消息消费的Quantity of services（QOS），每次接收几条消息 ` channel.basicQos(n)`



### MQ工作模式和交换机有哪些？

**模式 :** 

简单模式（1对1）、工作模式（1对多）、发布订阅模式、主题模式

**交换机 :** 

direct（默认）、fanout、topic、header（罕用）

生产者只能向交换机(Exchange)发送消息 : 交换机一边接收来自生产者的消息，另一边将消息推送到队列。交换器必须确切地知道如何处理它接收到的消息。



### 如何解决消息的幂等性问题？

**概念：**计算机科学中，多次请求所产生的影响与一次请求执行的影响效果相同，在消息队列中，主要指的就是同一条消息不能被重复消费；再比如注册时的form表单，用户点击两次提交是否会导致插入两条注册数据等

**例子：**开发一个转账功能，假设我们调用下游接口**超时**了，可能有三种情况：

- 一般情况下，**超时**可能是**网络传输丢包**的问题
- 请求时没送到
- 请求到了，**返回结果却丢**了。 

**解决方案 :** 每个消息用一个唯一标识来区分，消费前先判断标识有没有被消费过，若已消费过，则直接ACK



**方案1：Token机制实现**

![640](E:\doc_repo\002-应用\images\640.png)

**具体流程步骤：**

1. 客户端会先发送一个请求去获取 token，服务端会生成一个全局唯一的 ID 作为 token 保存在 redis 中，同时把这个 ID 返回给客户端
2. 客户端第二次调用业务请求的时候必须携带这个 token
3. 服务端会校验这个 token，如果校验成功，则执行业务，并删除 redis 中的 token
4. 如果校验失败，说明 redis 中已经没有对应的 token，则表示重复操作，直接返回指定的结果给客户端



**方案2：基于 redis 实现** 

![640 (1)](E:\doc_repo\002-应用\images\640 (1).png)

**具体流程步骤：**

1. 客户端先请求服务端，会拿到一个能代表这次请求业务的唯一字段
2. 将该字段以 SETNX 的方式存入 redis 中，并根据业务设置相应的超时时间
3. 如果设置成功，证明这是第一次请求，则执行后续的业务逻辑
4. 如果设置失败，则代表已经执行过当前请求，直接返回





### RabbitMQ如何保证消息的顺序性

将消息放入同一个交换机，交给同一个队列，这个队列只有一个消费者，消费者只允许同时开启一个线程



### RabbitMQ消息重试机制

消费者在消费消息的时候，如果消费者业务逻辑出现程序异常，这时候应该如何处理？
答案：使用消息重试机制(SpringBoot默认3次消息重试机制)



### MQ应用场景

1.用户完成注册后，向消息队列追加消息——异步发送注册成功邮件 ;

2.异步生成积分、生成用户操作日志 ;