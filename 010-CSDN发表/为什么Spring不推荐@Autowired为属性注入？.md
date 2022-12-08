# 生产实例：为什么Spring不推荐@Autowired为属性注入？



>最近在生产环境曝了一个空指针异常，第一次亲身意识到为什么Spring不推荐@Autowired为属性注入。
>
>故在此做个记录。



**描述：** WebMvcConfigurer接口提供了众多代替传统的xml配置文件形式的重写方法，以对MVC框架个性化定制 ，如解决跨域问题、添加拦截器、静态资源处理 、配置视图解析器等等。在当时的生产环境中，代码里重写了addInterceptors方法注册拦截器，以对请求进行拦截并进行token校验：

```java
@Primary	// 当存在多个WebMvcConfigurer类型的Spring Bean待创建时，优先创建该类
@Configuration
@AutoConfigureAfter({RedisUtil.class})  // 在RedisUtil类Bean创建后再创建
@ConditionalOnProperty(     // 当读取到lawstar.auth.local.enabled配置为true时才创建该bean
                       prefix="lawstar.auth.local",
                       value="enabled", 
                       havingValue = "true", 
                       matchIfMissing = true)
public class MyAuthConfig implements WebMvcConfigurer 
{
    @Autowired
    private RedisUtil redisUtil;

    @Bean
    public RemoteUserInterceptor remoteUserInterceptor() {
        return new RemoteUserInterceptor(redisUtil);
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(remoteUserInterceptor()).addPathPatterns("/**");
        registry.addInterceptor(new LocalUserInterceptor(redisUtil)).addPathPatterns("/**");
    }
}
```



在重写的addInterceptors方法中，两种不同的方式注入拦截器对象，一种是直接获取配置类内部@Bean注解返回的remoteUserInterceptor对象；第二种则是采用new的方式，获取一个localUserInterceptor对象。

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(remoteUserInterceptor()).addPathPatterns("/**");
    registry.addInterceptor(new LocalUserInterceptor(redisUtil)).addPathPatterns("/**");
}
```



在生产环境中，LocalUserInterceptor类的preHandle方法运行时暴空指针异常NPE：

```java
@Component
public class LocalUserInterceptor implements HandlerInterceptor 
{
    @Autowired
    private PropertyConfig propertyConfig;

    @Autowired
    private RedisUtil redisUtil;

    @Override
    public boolean preHandle(HttpServletRequest request, 
                             HttpServletResponse response, 
                             Object handler) throws Exception 
    {
        propertyConfig.doSomething(); 	// 该行代码暴NPE
        return true;
    }

    public LocalUserInterceptor(RedisUtil redisUtil) {
        this.redisUtil = redisUtil;
    }
}
```



当时还怀疑难道是LocalUserInterceptor的Bean对象里propertyConfig属性没有注入吗？但在IOC容器里，Bean内部通过@Autowired注入的属性如果为null，则启动时就会直接报错，除非定义注解属性required为false，即@Autowired(required=false) 。所以，问题出在哪里呢？答案是，注册拦截器时传入的是一个新new的LocalUserInterceptor对象。不同于Spring为你创建的Bean对象，手动new的LocalUserInterceptor，其添加了@Autowired注解的propertyConfig属性为空。这也是为什么Spring官方推荐使用@Autowired为构造器注入的方法（如下），在这种情况下，即使是手动new的LocalUserInterceptor对象，其内部属性也已经实现了注入，不会存在调用空指针的问题。

```java
@Component
public class LocalUserInterceptor implements HandlerInterceptor 
{
   
    private PropertyConfig propertyConfig;

    private RedisUtil redisUtil;

	@Autowired
    public LocalUserInterceptor(RedisUtil redisUtil, 
    						  PropertyConfig propertyConfig) 
    {
        this.redisUtil = redisUtil;
        this.redisUtil = propertyConfig;
    }
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                             HttpServletResponse response, 
                             Object handler) throws Exception 
    {
        propertyConfig.doSomething(); 
        return true;
    }

}
```



