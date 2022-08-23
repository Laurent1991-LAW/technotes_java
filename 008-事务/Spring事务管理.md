# Spring事务管理



## 1 声明式事务

@Transactional 注解，是使用 AOP 实现的，本质就是**在目标方法执行前后进行拦截**。在目标方法执行前加入或创建一个事务，在执行方法执行后，根据实际情况选择提交或是回滚事务；

在代码侵入性方面，声明式事务虽然优于编程式事务，但也有不足，声明式事务管理的粒度是方法级别，而编程式事务是可以精确到代码块级别的；



### 1.1 注解参数

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    
 // 传播行为   
 Propagation propagation() default Propagation.REQUIRED;
 
 // 隔离级别
 Isolation isolation() default Isolation.DEFAULT;
    
 // 超时时间 - 执行时间超过该值，则直接回滚
 int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;
    
 // 是否为只读操作
 boolean readOnly() default false;

}

```



#### 1.1.1 传播行为参数

##### `PROPAGATION_REQUIRED`

- `PROPAGATION_REQUIRED` 这也是 @Transactional 默认的事务传播行为，指的是如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务； 
- 如下：aMethod 调用了 bMethod，只要其中一个方法回滚，整个事务均回滚 

```java
Class A {
   @Transactional(propagation=Propagation.PROPAGATION_REQUIRED)
    public void aMethod {
        //do something
        B b = new B();
        b.bMethod();
    }
}

Class B {
   @Transactional(propagation=Propagation.PROPAGATION_REQUIRED)
    public void bMethod {
       //do something
    }
}
```



##### `PROPAGATION_REQUIRES_NEW`

- `PROPAGATION_REQUIRES_NEW`，创建一个新的事务，如果当前存在事务，则把当前事务挂起。也就是说不管外部方法是否开启事务，ropagation.REQUIRES_NEW 修饰的内部方法都会开启自己的事务，且开启的事务与外部的事务相互独立，互不干扰；
- 如下：如果 aMethod()发生异常回滚，bMethod()不会跟着回滚，因为 bMethod()开启了独立的事务。但是，如果 bMethod()抛出了未被捕获的异常并且这个异常满足事务回滚规则的话, aMethod()同样也会回滚 ;

```java
Class A {
    @Transactional(propagation=Propagation.PROPAGATION_REQUIRED)
    public void aMethod {
        //do something
        B b = new B();
        b.bMethod();
    }
}

// --------------------

Class B {
    @Transactional(propagation=Propagation.REQUIRES_NEW)
    public void bMethod {
       //do something
    }
}
```



#### 1.1.2 隔离级别参数

采用默认的隔离级别 ISOLATION_DEFAULT 就可以了，也就是交给数据库来决定，可以通过 `SELECT @@transaction_isolation;` 命令来查看 MySql 的默认隔离级别，结果为 REPEATABLE-READ，也就是可重复读 



### 1.2 应用@Transactional 注解

#### 1.2.1 错误案例 

在同一个类中将需要启用事务管理的代码（如入库代码）用单一方法封装，方法带事务注解，主方法调用该方法

```java
@Service
public class OrderService{

    public void createOrder(OrderCreateDTO createDTO){
        query();
        validate();
        saveData(createDTO); //事务管理的入库方法
    }
  
    //事务操作
    @Transactional(rollbackFor = Throwable.class)
    public void saveData(OrderCreateDTO createDTO){
        orderDao.insert(createDTO);
    }

}
```

这种拆分会命中使用@Transactional注解时事务不生效的经典场景：@Transactional注解的声明式事务是通过spring aop起作用的，而spring aop需要生成代理对象，直接在同一个类中方法调用使用的还是原始对象，事务不生效。

##### 为什么同一类中调用事务方法会失效？

- TestController直接调用TestService 的test2()方法，test2()上的@Transactional生效，如果是调用test1()方法，test2()上的@Transactional不生效
- 原因：在test1方法中调用test2方法相当于使用this.test2()，**this代表的是Service类本身，并不是为事务专门生成的代理Service对象** （注意：代理一大功能就是进行功能拓展，比如前后添加事务操作），因此不能实现代理功能
- Spring 在扫描bean的时候会扫描方法上是否包含@Transactional注解，如果包含，Spring 会为这个bean动态地生成一个子类（即代理类proxy），proxy是继承原来那个bean的。此时，当这个有注解的方法被调用的时候，实际上是由proxy来实现调用并添加事务操作的；
- 如果这个有注解的方法是被同一个类中的其他方法调用的，那么该方法的调用并没有通过proxy，而是直接通过原来的那个bean，所以就不会启动transaction，即@Transactional注解无效。 

```java
@RestController
public class TestController {
    @Autowired
    private TestService testService;
     
    public void test(){
         
    }
}
// ------------------------
@Service
public class TestService {
 
    public void test1(){
        test2();
    }
     
    @Transactional
    public void test2(){
    }
}
```





#### 1.2.2 正确案例1

可以将方法放入另一个类，如新增 manager层，通过spring注入，这样符合了在对象之间调用的条件。

```java
// OrderService.java类
@Service
public class OrderService{
  
    @Autowired
   private OrderManager orderManager;

    public void createOrder(OrderCreateDTO createDTO){
        query();
        validate();
        orderManager.saveData(createDTO);
    }
}

// OrderManager.java类
@Service
public class OrderManager{
  
    @Autowired
   private OrderDao orderDao;
  
  @Transactional(rollbackFor = Throwable.class)
    public void saveData(OrderCreateDTO createDTO){
        orderDao.saveData(createDTO);
    }
}
```



1.2.3 正确案例2

**前提：**在springboot启动类上加上注解@EnableAspectJAutoProxy(exposeProxy = true) 

- 利用AopContext.currentProxy() 获取代理对象；
- 通过代理对象调用方法；

```java
@Service
public class OrderServiceImpl {

    public void createOrder(OrderCreateDTO createDTO){
        
        OrderServiceImpl currentProxy = (OrderServiceImpl) AopContext.currentProxy() ;
        
        query();
        validate();
        currentProxy.saveData(createDTO); //事务管理的入库方法
    }

    //事务操作
    @Transactional(rollbackFor = Throwable.class)
    public void saveData(OrderCreateDTO createDTO) {
        orderDao.insert(createDTO);
    }   
    
}
```





### 1.3 事务失效场景

- 在A方法中调用添加了@Transactional的B方法，导致事务失效；
- 异常被catch捕获导致@Transactional失效, 解决办法：在catch语句中抛出异常；
- @Transactional 应用在**非 public 修饰的方法上** --> spring事务本身是通过cglib代理实现的，代理类是委托类的子类, 非公开的方法将无法在代理类中应用；
- @Transactional 注解属性 propagation 设置错误；
- @Transactional 注解属性 rollbackFor 设置错误；



### 1.4 阿里巴巴规范

需要在@Transactional指定rollbackfor，或者在方法中显式的rollback()



如果不对 运行时异常 进行处理，那么出现 运行时异常 之后，要么是线程中止，要么是主程序终止 ;

如果不想终止处理线程，则必须捕获所有的运行时异常，

队列里面出现异常数据了，正常的处理应该是把异常数据舍弃，然后记录日志。不应该由于异常数据而影响下面对正常数据的处理。

非运行时异常 是 RuntimeException以外的异常，类型上都属于Exception类及其子类。如 IOException、SQLException等以及用户自定义的Exception异常。对于这种异常，JAVA编译器强制要求我们必需对出现的这些异常进行catch并处理，否则程序就不能编译通过。

 1 希望checked异常也回滚：在整个方法前加上 @Transactional (rollbackFor = Exception.class)

2  希望unchecked异常不回滚： @Transactional (notRollbackFor = RunTimeException.class)

 

 

 

 

 

 

 

 

 



## 2 编程式事务

与声明式事务对应的就是编程式事务，基于底层的API，开发者在代码中手动的管理事务的开启、提交、回滚等操作。在spring项目中可以使用TransactionTemplate类的对象，手动控制事务。

```
@Autowired 
private TransactionTemplate transactionTemplate; 
 
public void save(RequestBill requestBill) { 
    transactionTemplate.execute(transactionStatus -> {
        requestBillDao.save(requestBill);
        //保存明细表
        requestDetailDao.save(requestBill.getDetail());
        return Boolean.TRUE; 
    });
} 
```