# Feign



**功能：**将 远端的服务调用 化为 消费方内部的一个接口来实现

## 应用实践

**第一步**：在服务消费方，添加项目依赖 spring-cloud-starter-openfeign：



**第二步**：在启动类上添加@EnableFeignClients注解

```java
@EnableFeignClients     //告诉Spring容器对未来使用@FeignClient注解描述的接口创建其实现类/对象
@SpringBootApplication
public class ConsumerApplication {…}
```

@EnableFeignClients缺失 = > FeignClientRegister缺失 => 不会对FC注解的接口进行实例化 => No bean found



**第三步**：定义Http请求API，基于此API借助OpenFeign访问远端服务，细则如下：

1. 一般来说，一个接口对应一个远程调用的服务，用 **@FeignClient**(name = "sca-provider") 标记，name参数为服务名；
2. 内部方法根据 目标controller的请求方法添加对应注解 **@GetMapping**("/provider/echo/{msg}")，默认值为对应路径；
3. restful风格的请求，url参数必须用@PathVariable进行绑定；

```java
@FeignClient(name = "sca-provider")
public interface remoteProviderService {
    
    @GetMapping("/provider/echo/{msg}")
    String echoMsg(@PathVariable("msg") String msg);
    
// Feign只能兼容到jdk5，当时的restFul还无法直接接收PathVriable的参数，因此必须为 @PathVariable("msg")，但后方变量名随意 
}
```

- 若@PathVariable注解不带参数：

​                  ![img](https://docimg10.docs.qq.com/image/IA9p6A3AQI0rrgmP5CsiAQ.png?w=704&h=88)         

- @FeignClient定义远端调用规范，属性name为远端服务名，同时也是remoteProviderService接口实现类的Bean对象名（类似Mapper接口的bean也是Spring框架建立） ;
- Consumer端 访问远程服务的服务层 接口命名：remoteProviderService ;
- Feign接口（remoteProviderService）基于内部方法上GetMapping的value属性去远端调用对应方法 => GetMapping注解必需 ;
- 若存在多个接口@FeignClient（name ="sca-provider"）—— 毕竟一个服务里有多类服务，我们在调用时也希望对其进行分类 —— 由于它们创建代理实现类时名字都为sca-provider，会出现调用冲突，可利用contextId属性给其中一个命名，则不会出现重名情况 也可通过如下配置解决 `spring.main.allow-bean-definition-overriding=true ` ;



## FallBackFactory



**Hystrix**：Netflix开源的一款容错框架，具有自我保护能力

FallbackFactory接口是Feign提供的容错框架，它可以对业务进展中出现的问题/错误进行反馈。在远程调用的接口的 @FeignClient注解内 定义其fallbackFactory属性前，必须：

1.定义一个实现FallbackFactory\<T>接口的实现类，其中泛型类型为FeignClient所修饰的接口类型：实现类中重写create（）方法，参数为错误异常的顶级父类Throwable，方法体为错误反馈方式，比如返回提示"服务维护中，请稍后重试…" ;

2.在配置文件中开启Hystrix服务：Feign.Hystrix.enabled = true ;

3.@FeignClient添加参数fallbackFactory = 实现类名

![20220604](E:\doc_repo\005-Feign\images\20220604.png)







