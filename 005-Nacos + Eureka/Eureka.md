# Eureka



## 概述

- 以实现中间层服务器的负载均衡和故障转移，遵循着CAP理论中的 A (可用性)、P (分区容错性) ;

- 一个Eureka中分为eureka server和eureka client,其中eureka server是作为服务的注册与发现中心, eureka client既可以作为服务的生产者 (producer), 又可以作为服务的消费者 (provider)

  

![39e48091d676491db8ec759c42f9b4c6](E:\doc_repo\005-Nacos + Eureka\images\39e48091d676491db8ec759c42f9b4c6.png)



## 常见配置参数与注解

服务端：@EnableEurekaServer 

注册端：@EnableDiscoveryClient（适用于各种注册中心） 或 @EnableEurekaClient



- eureka.server.**enable-self-preservation**    自我保护机制

  > 若 心跳失败的比例 在15分钟内 是否超过85%，如果出现了超过的情况，Eureka Server会将当前的实例注册信息保护起来，同时提示一个警告，一旦进入保护模式，Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据。也就是不会注销任何微服务

- eureka.client.**register-with-eureka** 	表示是否将自己注册到 Eureka Server；

- eureka.client.**fetch-registry**    表示是否从Eureka Server拉取注册的服务信息, 默认true, 集群必须设置为true；

- eureka.client.**registry-fetch-interval-seconds** = 30   拉取时间间隔

- eureka.instance.**lease-renewal-interval-in-seconds** = 30       服务每30秒向注册中心发送心跳

- eureka.instance.**lease-expiration-duration-in-seconds** = 90      超过90分钟仍未收到心跳，标记下线



## 集群搭建

### Server



一、引入依赖：

```
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

二、启动类上添加注解@EnableEurekaServer ；

三、配置文件：

```yaml
server: 
  port: 8001
  
eureka:
  instance:
    instance-id: service8001	 # 服务实例Id
    prefer-ip-address: true  #访问路径可以显示IP地址
  client:
    register-with-eureka: true  # 是否向注册中心注册自己
    fetch-registry: true   # 是否从注册中心抓取已有的注册信息，默认true，集群必须设置为true
    service-url:
      # 集群中各个服务注册中心的地址
      defaultZone: http://eureka8761:8761/eureka, http://eureka8762:8762/eureka, http://eureka8763:8763/eureka
  

```



```yaml
server: 
  port: 8761
  
eureka: 
  instance:
    hostname: eureka8761 	# eureka 服务端的实例名称
  client: 
    register-with-eureka: false    # false 表示不向本端注册中心注册自己
    fetch-registry: false    # false 表示自己端就是注册中心->我的职责就是维护服务实例，无需拉取注册表
    service-url:      # Eureka 实例之间互相注册，即把自己注册到另外两个服务注册中心实例中
      defaultZone: http://eureka8761:8761/eureka/,http://eureka8762:8762/eureka/,http://eureka8763:8763/eureka/

```





### client



一、引入依赖：

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

二、启动类上添加注解 @EnableDiscoveryClient  和  @EnableEurekaClient

三、配置文件

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://eureka8762:8762/eureka/,http://eureka8763:8763/eureka/,
```





## 补充

### 选择正确网卡

eureka客户端向eureka注册时, 会自动选择网卡, 并可能注册主机名而不是ip地址
服务器有多块网卡, 要选择正确网卡的ip地址向eureka进行注册
下面配置可以选择正确网卡的ip向eureka进行注册
选择正确网卡，修改 bootstrap.yml

```yaml
spring:
  cloud:
    inetutils:
      ignored-interfaces: # 忽略的网卡
        - VM.*
      preferred-networks: # 要是用的网卡的网段
        - 192\.168\.0\..+
```



### 修改注册名

```
eureka:
  instance:
    prefer-ip-address: true # 使用ip进行注册
    instance-id: ${spring.cloud.client.ip-address}:${spring.application.name}:${server.port} # 界面列表中显示的格式也显示ip

```

