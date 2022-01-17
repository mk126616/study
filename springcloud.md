# mvn子项目单独打可执行jar包

mvn自带的打包工具默认不会把香满园依赖的jar包打进去，如果要单独作为jar包运行的话需要在pom上配置**spring-boot-maven-plugin**插件，但是

父项目在单独打包时引入了**spring-boot-maven-plugin**插件则直接打成可运行jar包，微服务子项目在打包时引入**spring-boot-maven-plugin**插件后还需要配置**epackage**参数才行。配置了**spring-boot-maven-plugin**插件打包后的jar包会进行二次打包，包中文件路径发生变化，并且不能被别的项目引用，因此打包时只作为公共资源jar包而不单独运行的jar包不能够添加**spring-boot-maven-plugin**插件。    

```xml
<build>
    <!--        mvn默认只会把resources下的文件打包进去，resources底下的文件夹也不行，需要添加下打包资源-->
    <resources>
        <resource>
            <directory>${basedir}/src/main/resources</directory>
            <filtering>true</filtering>
            <includes>
                <include>**/*.xml</include>
                <include>**/*.yml</include>
            </includes>

        </resource>
        <resource>
            <directory>${basedir}/src/main/java</directory>
            <filtering>true</filtering>
            <includes>
                <include>**/*.xml</include>
            </includes>
        </resource>
    </resources>
    <plugins>
        <!--            此插件会在打包时把项目中依赖的jar包全部打进去，默认不打进去的-->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            //子项目默认不进行二次打包，需要配置此属性才可以生成单独运行的jar包
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>8</source>
                <target>8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

# mvn依赖的传递性

mvn在通过pom引入jar包时会传递的引入jar中依赖的jar包，引入的jar包pom中的依赖jar包必须指明版本号，并且不能通过父项目指定，必须直接指定。

# idea中多个微服务管理仪表盘配置

项目idea目录workspace.xml中增加如下配置即可：

```xml
<component name="RunDashboard">
    <option name="configurationTypes">
      <set>
        <option value="SpringBootApplicationConfigurationType" />
      </set>
    </option>
  </component>
```



# 微服务架构概念：

将一个大型的项目按照功能模块拆分为单体服务应用，模块之间松耦合，每个应用之间独立开发独立部署，并且所有服务之间通过轻量级的通信方式（如：restful风格的http请求进行通信）协调作用，为用户提供一个完整的应用服务。

## 微服务架构的好处：

- 各个服务之间能够相互解耦，易于开发和维护，系统的可扩展性强（开发者）
- 整个系统内的服务可以采用不同的技术去开发、技术栈改造替换更容易（技术异构）
- 各个服务应用独立部署线上环境维护升级替换更容易，不需要替换整个系统（线上环境维护）
- 线上部署可以根据需求灵活组合服务（线上部署）
- 可靠性更高（系统拆分为很多服务，部分服务故障对整个系统影响小）

## 微服务的弊端：

- 影响性能：服务之间通过网络请求进行通信
- 运维复杂：服务众多，需要对所有服务状态进行监控
- 开发时部分代码难度提升：如分布式事务、分布式锁

## 主流微服务架构系统开发技术：

springboot:  	快速构建开发微服务模块

springcloud：系统内所有服务之间相互调用、监控、维护解决方案

## CAP理论

- CAP理论：一个分布式系统不可能同时满足一致性，可用性和分区容错性这个三个基本需求，最多只能同时满足其中两项
- C（一致性）：所有的节点上的数据时刻保持同步
- A（可用性）：每个请求都能接受到一个响应，无论响应成功或失败
- P（分区容错）：系统能持续提供服务，即使系统内部有消息丢失（分区）

## CAP定理的应用

- 放弃P：如果希望能够避免系统出现分区容错性问题，一种较为简单的做法就是将所有的数据(或者是与事物先相关的数据)都放在一个分布式节点上，这样索然
- 无法保证100%系统不会出错，但至少不会碰到由于网络分区带来的负面影响
- 放弃A:其做法是一旦系统遇到网络分区或其他故障时，那马受到影响的服务需要等待一定的时间，英雌等待期间系统无法对外提供正常的服务，即不可用
- 放弃C:这里说的放弃一致性，并不是完全不需要数据一致性，是指放弃数据的强一致性，保留数据的最终一致性。
  



# Spring Cloud技术体系![image-20220107173801932](C:\Users\19746\AppData\Roaming\Typora\typora-user-images\image-20220107173801932.png)

spring cloud升级后某些组件不再维护，又出现了相应的替代品,如下所示：

![image-20220107144437979](C:\Users\19746\AppData\Roaming\Typora\typora-user-images\image-20220107144437979.png)

- 服务注册中心：注册各个微服务service

- 服务降级：在核心业务服务繁忙时，返回兜底数据保证整体服务对客户的可用、且友好

- 服务网关：包含了路由转发（接收外界请求转发到微服务上）和过滤器（横切，如完成权限校验、限流、监控）

- 服务配置：统一配置中心

- 服务总线：

  

# spring boot热部署

热部署只能在开发环境使用，线上环境需要去掉，否则修改文件后会自动重启

1.增加依赖和插件

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```







2.对idea进行设置

![image-20220108151122733](C:\Users\19746\AppData\Roaming\Typora\typora-user-images\image-20220108151122733.png)





**Ctrl+shift+alt+/打开设置**，设置后**重启idea**

![image-20220108151509079](C:\Users\19746\AppData\Roaming\Typora\typora-user-images\image-20220108151509079.png)





# 注册中心

当服务数量太多时，服务管理就会混乱，因此需要使用注册中心进行服务管理，可以实现服务的注册时与发现，并且可以进行负载均衡

## eureka

- eureka设计为cs架构，服务器端提供微服务的注册与发现，客户端将自己注册时到服务器端的注册表上，服务的唯一标识为服务的别名（springboot中使用配置项**spring.application.name**指定）

- eureka集群：

  集群之间相互注册、相互守望

- eureka客户端会通过心跳机制定期给服务端发送信息告诉服务端自己还活着。

- eureka有自我保护机制：eureka在客户端心跳不正常时，默认不会立刻清理掉服务的注册信息，防止网络卡顿等情况导致心跳信息未按时收到。可以关闭自我保护模式并设置服务剔除时间，超时则剔除。

  

## ReseTemplate(服务调用工具)

spring提供的一个rest请求的客户端工具，服务调用时需要使用此工具进行调用（spring通过拦截器赋予了它负载均衡的能力，在发送请求时会被拦截，然后去注册时中心根据负载均衡策略根据服务名获取一个此服务具体的服务地址去调用）

默认情况下不具有负载均衡能力需要在配置restTemplate对象时增加**@LoadBalanced**注解。

```java
/**
 * rest请求便捷操作工具
 * @return
 */
@Bean
@LoadBalanced //赋予template负载均衡的能力（加此注解后在发送请求时会被拦截器拦截，并去注册中心获取一个服务具体的地址，然后请求这个实际的地址）
public RestTemplate restTemplate(){
    return new RestTemplate();
}
```



## zookeeper

zookeeper中节点的注册为临时的并非持久（一旦服务停止后zookeeper服务器上的服务注册信息会立刻被清除）



## consul

是一个分布式的服务注册和配置管理，使用go语言开发。可以实现服务发现、健康检查、kv存储、多数据中心、可视化web界面



## 三个注册中心对比

![image-20220109200602135](C:\Users\19746\AppData\Roaming\Typora\typora-user-images\image-20220109200602135.png)







# 服务调用

### DiscoveryClient

微服务发现客户端工具，每个注册时组件自己实现此接口，使用时需要在spring boot主启动类增加@EnableDiscoveryClient注解，spring clod帮我们注册其到ioc中，我们直接使用即可。

### ServiceRegistry

微服务注册器，每个注册时组件自己实现此接口



## ribbon

是**restTemplate**负载均衡的底层原理，微服务中进行服务负载均衡调用的工具。在进行服务调用时，先去注册中心获取到可用服务对应的地址缓存到内存中，然后在本地通过负载均衡策略选取一个具体的服务去调用服务。ribbon是一个本地负载均衡工具，集成在服务的消费方，而nginx是个服务器端的负载均衡。



### ribbon负载均衡算法：

ribbon的负载均衡算法为**IRule**接口的实现类，默认的实现有：

（1）RoundRobinRule：轮询（**默认**）；

（2）RandomRule：随机；

（3）AvailabilityFilteringRule：会先过滤掉由于多次访问故障而处于断路器状态的服务，还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问；

（4）WeightedResponseTimeRule：根据平均响应时间计算所有服务的权重，响应时间越快的服务权重越大被选中的概率越大。刚启动时如果统计信息不足，则使用RoundRobinRule（轮询）策略，等统计信息足够，会切换到WeightedResponseTimeRule；

（5）RetryRule：先按照RoundRobinRule（轮询）策略获取服务，如果获取服务失败则在指定时间内进行重试，获取可用的服务；

（6）BestAvailableRule：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务；

（7）ZoneAvoidanceRule：复合判断Server所在区域的性能和Server的可用性选择服务器；



### 自定义ribbon负载均衡算法



框架层面已经提取了抽象类直接集成此类即可

```java
/**
 * 自定义ribbon的负载均衡策略
 */
public class MyRibbonRule extends AbstractLoadBalancerRule
{

    @Override
    public void initWithNiwsConfig(IClientConfig iClientConfig){
    }

    @Override
    public Server choose(Object key)
    {
        return choose(getLoadBalancer(),key);
    }
    public Server choose(ILoadBalancer loadBalancer,Object key){
        return loadBalancer.getReachableServers().get(0);
    }
}
```



### @LoadBalancer注解赋予RestTemplate负载均衡能力底层原理

**restTemoplate**声明到ioc中时增加了此注解则会启动**restTemplate的拦截器**，**restTemplate**在调用服务时会被拦截器拦截，拦截器底层使用**DiscoveryClient**获取服务具体的实例信息，然后调用ioc中**Irule**对象的**choose(）**方法.。选取一个服务实例返回，**restTemplate**使用此实例对象的**uri**信息调用具体服务

代码实现：





```java

/**
 * 所有订单
 *
 * @return
 */
@GetMapping("/consumer/payments")
Result getAllPayment()
{
    ServiceInstance serviceInstance = null;
    try
    {
        //此处为@LoadBalanced注解赋予restTemplate负载均衡功能的原理
        List<ServiceInstance> instances = discoveryClient.getInstances("PAYMENT-SERVICE");
        serviceInstance = loadBalancer.choose(instances);
    }
    catch (Exception e)
    {
        e.printStackTrace();
    }
    return restTemplate.getForObject(serviceInstance.getUri()+"/payments",Result.class);
}
```



```java
@Component
public class MyLoadBalancer implements LoadBalancer
{
    /**
     * 记录当前请求次数
     * AtomicInteger只能保证高并发下修改数据时的原子性，业务代码一旦有问题也会导致线程安全问题。
     * 比如以下业务AtomicInteger值需要依次递增，但是业务代码中两个线程如果同时拿到了0，都增加1，增加后正确的值应该为2，
     * 调用修改方法时两个线程都给的值为1，则最终的值为1，数据出现问题。
     * 因此在此业务场景下可以对整个方法加锁，或者使用轻量级别的自旋锁
     */
    private AtomicInteger requestCount = new AtomicInteger(0);

    @Override
    public ServiceInstance choose(List<ServiceInstance> serviceInstances) throws Exception
    {
        int current;int next;int serverCount = serviceInstances.size();
        if(serviceInstances == null || serviceInstances.size() == 0){
            throw new Exception("服务不存在");
        }
        //每次请求完成需要维护请求次数值，但是高并发情况下值可能会被别的线程修改因此使用自旋锁（比synchronized轻量级，
        // 如果使用synchronized实现的话需要锁整个方法效率低）
        do {
            current = requestCount.get();
            next = current == Integer.MAX_VALUE ? 0 : current+1;
            //compareAndSet 和乐观锁比较像，如果当前值为期望值current 则更新值为next，否则返回false
        }while (!requestCount.compareAndSet(current,next));
        //此处的策略为轮询访问服务
        return serviceInstances.get(next % serverCount);
    }
}
```



## openFeign

是一个服务接口绑定器，使用声明式方式的注解进行接口方法绑定。并且整合了ribbon支持客户端负载均衡。底层为动态代理封装http请求，简化了java层面进行服务调用。

代码实现：

```java
@EnableFeignClients
@SpringBootApplication
public class OrderStarter_Eureka_openfeign_8091
{
    public static void main(String[] args)
    {
        SpringApplication.run(OrderStarter_Eureka_openfeign_8091.class, args);
    }
}
```





```java
//消费者直接调用本地feign包装的服务方接口即可
@Component
@FeignClient(value = "PAYMENT-SERVICE")
public interface PaymentService
{
    @GetMapping("payments")
    Result getAllPayment();

    @PostMapping("/payment")
    Result createPayment(PaymentEntity entity);
}
```



### 超时控制

openFeign整合了ribbon进行超时控制，使用时直接配置ribbon调用超时控制即可(默认为1000ms)

```yaml
ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000
```

### 日志打印

1. 配置日志级别bean：

   ```java
   /**
    * feign的日志配置对象创建
    */
   @Configuration
   public class FeignLog
   {
       @Bean
       public Logger.Level logLevel(){
           return Logger.Level.FULL;
       }
   }
   ```

2. feign接口配置:

```yml
logging:
  level:
    #feign以什么级别监控某个接口
    com.mk.service.PaymentService: debug
```



# 服务降级、服务熔断、服务限流



## 微服务架构问题：

服务雪崩：微服务系统中业务复杂情况下一次请求会依赖比较多的服务，难免会因为单个服务故障、网络延迟会导致服务调用方tomcat线程长期阻塞，请求长时间得不到应，阻塞的问题沿着服务调用链路向上传递，导致大规模服务异常。



- 服务降级：服务异常、网络故障时返回一个兜底、友好的信息（程序异常、超时、熔断触发降级、线程池打满）
- 服务熔断：达到服务最大连接数时直接断开服务并且进行服务降级返回友好的信息
- 服务限流：秒杀高并发等情景，请求限制并发数量，所有请求排队处理



## hystrix



微服务架构中延迟和容错的开源库，防止微服务架构系统中个别服务异常导致整个服务失败，提高分布式系统弹性。在单个服务调用异常时返回一个兜底的fallback响应防止调用方tomcat线程长时间被占用，调用方服务异常，最终导致服务异常的蔓延最终服务雪崩。

具体问题和处理方式：

1. 等待超时 ：  不再等待
2. 服务宕机或者服务出错： 返回兜底数据

### 服务降级：

服务提供方：

需要先开启服务熔断功能

```
@EnableHystrix   //开启使用hystrix降级、熔断功能
@EnableEurekaClient //标识为eureka客户端
@MapperScan(basePackages = {"com.mk.mapper"})
@SpringBootApplication
public class PaymentStarter_eureka_feign_hystrix_8092
{
    public static void main(String[] args)
    {
        SpringApplication.run(PaymentStarter_eureka_feign_hystrix_8092.class, args);
    }
}
```

- 给单个方法指定具体的出错处理方法：

```java 
//给方法设置具体的兜底处理方法
@HystrixCommand(fallbackMethod = "getAllPaymentErrorHandler", commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "5000")
})
@Override
public Result<List<PaymentEntity>> getAllPaymentError()
{
    try
    {
        int a = 10/0;
        Thread.sleep(8000);
        return Result.success(mapper.getAllPayment());
    }
    catch (Exception e)
    {
        return Result.error();
    }
}

//出错兜底方法
public Result<List<PaymentEntity>> getAllPaymentErrorHandler()
{
    return new Result<>("500", "服务提供方出错", null);
}

```

- 给整个类设置统一的出错处理方法：

  ```java
  @DefaultProperties(defaultFallback="getAllPaymentErrorHandler") //指定此类中需要进行降级处理方法统一的兜底方法
  @Service
  public class PaymentServiceImpl implements PaymentService
  {
      @Autowired
      private PaymentMapper mapper;
  
      @HystrixCommand //标识此方法需要在出现问题时进行降级处理
      @Override
      public Result<List<PaymentEntity>> getAllPaymentError()
      {
              int a = 10/0;
              return Result.success(mapper.getAllPayment());
      }
  
      public Result<List<PaymentEntity>> getAllPaymentErrorHandler()
      {
          return new Result<>("500", "服务提供方出错", null);
      }
  ```

  

服务消费者：

服务方除了对类设置统一的默认出错处理方法和对方法设置精准的出错处理方法外，还可以通过feign对一个服务接口的所有方法设置一对一出错处理方法。

1. 开启服务熔断降级处理

```java
@EnableHystrix   //开启熔断、降级处理
@EnableFeignClients //开启feign服务发现调用功能
@EnableEurekaClient //会帮我们注册服务的发现到ioc中：DiscoveryClient（微服务的发现客户端）
@SpringBootApplication
public class Order_eureka_feign_hystrix_8093
{
    public static void main(String[] args)
    {
        SpringApplication.run(Order_eureka_feign_hystrix_8093.class, args);
    }
}
```

2. 开启feign对hystrix的支持

```
##开启feign对hystrix的支持
feign:
  hystrix:
    enabled: true
```

3. 实现feign接口包装的第三方服务接口,并注入到ioc中

   ```java
   @Component
   public class PaymentServiceErrorHandler implements PaymentService
   {
       @Override
       public Result createPayment(PaymentEntity entity)
       {
           return new Result<>("500", "服务提供方出错:createPayment", null);
       }
   
       @Override
       public Result getAllPaymentOk()
       {
           return new Result<>("500", "服务提供方出错:getAllPaymentOk", null);
       }
   
       @Override
       public Result getAllPaymentError()
       {
           return new Result<>("500", "服务提供方出错:getAllPaymentError", null);
       }
   }
   ```

4. 指定服务调用出错后处理类

```java
@FeignClient(value = "payment-service",fallback=PaymentServiceErrorHandler.class)
public interface PaymentService
{
    /**
     * 创建订单
     *
     * @param entity
     * @return
     */
    @PostMapping("/payment")
    Result createPayment(PaymentEntity entity);

    /**
     * 所有订单
     *
     * @return
     */
    @GetMapping("paymentsOk")
    Result getAllPaymentOk();

    /**
     * 所有订单
     *
     * @return
     */
    @GetMapping("paymentsError")
    Result getAllPaymentError();
}
```



### 服务熔断

一段时间内服务被调用异常的频率达到设定的比例后进行服务的降级、断开服务链路，服务正常时恢复服务链路

#### 断路器状态：

- open（断路）

- half open（半开只允许一个请求通过）

- close（关闭）

  

#### 断路器断开条件：

- 一定时间

- 请求达到一定次数

- 请求失败达到一定比例

  

#### 工作过程：

断路器会在达到断开条件时进入**ope**n状态服务进行服务降级直接访问服务降级方法，在一段时间后（默认5 s）进入half open状态只允许一个请求通过，请求正常则断路器进入close状态服务正常使用。

#### 代码实现：

```java
@EnableCircuitBreaker  //启动断路器功能
@EnableHystrixDashboard 
@EnableHystrix   //开启降级、熔断功能
@EnableEurekaClient //标识为eureka客户端
@MapperScan(basePackages = {"com.mk.mapper"})
@SpringBootApplication
public class PaymentStarter_eureka_feign_hystrix_8092
{
    public static void main(String[] args)
    {
        SpringApplication.run(PaymentStarter_eureka_feign_hystrix_8092.class, args);
    }
}
```

```java
@HystrixCommand(fallbackMethod = "getAllPaymentErrorHandler",
        commandProperties = {
        @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
        //时间窗口内总请求数量
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
        //时间窗口时间
        @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),
        //时间窗口内出错比例
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")
})
@Override
public Result<List<PaymentEntity>> getAllPaymentError(Integer id)
{
    if(id < 0){
       throw new RuntimeException("参数不能小于0");
    }
    List<PaymentEntity> allPayment = mapper.getAllPayment();
    return Result.success(allPayment);
}

public Result<List<PaymentEntity>> getAllPaymentErrorHandler(Integer id)
{
    return new Result<>("500", "服务提供方出错", null);
}
```



### 熔断、降级、限流图形化监控展示仪表盘 dashboard

监控方：

```xml
<!--       hystrix图形化监控依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
<!--        web监控-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



```java
@EnableHystrixDashboard  //开启hystrix的图形化监控功能
@SpringBootApplication
public class Hystrix_dashboard_8094
{
    public static void main(String[] args)
    {
        SpringApplication.run(Hystrix_dashboard_8094.class,args);
    }
}
```





被监控方：

增加获取监控数据的servlet

```java
@Bean
public ServletRegistrationBean getServlet() {
    HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
    ServletRegistrationBean<HystrixMetricsStreamServlet> registrationBean = new ServletRegistrationBean<>(streamServlet);
    registrationBean.setLoadOnStartup(1);
    registrationBean.addUrlMappings("/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    return registrationBean;
}
```



# 网关

微服务中用户的所有请求在发起后都要通过网关进行转发，我们可以使用网关进行统一的鉴权、日志、限流的工作。

## Gateway

gateway网关非常规的servlet规范，是基于高性能的异步、事件驱动的通讯框架netty框架开发，并且使用了spring5中新特性ServerWebExchange。

### 组成：

#### 1）Route

Route 是网关的基础元素，由 ID、目标 URI、断言、过滤器组成。当请求到达网关时，由 Gateway Handler Mapping 通过断言进行路由匹配（Mapping），当断言为真时，匹配到路由。

#### 2）Predicate

Predicate 是 [Java](http://c.biancheng.net/java/) 8 中提供的一个函数。输入类型是 Spring Framework ServerWebExchange。它允许开发人员匹配来自 HTTP 的请求，例如请求头或者请求参数。简单来说它就是匹配条件。

#### 3）Filter

Filter 是 Gateway 中的过滤器，可以在请求发出前后进行一些业务上的处理。



核心为路由转发加过滤链，以下为官网给出的getway工作流程图：

![Spring Cloud Gateway工作原理](http://c.biancheng.net/uploads/allimg/190826/5-1ZR6093952150.png)



### 网关路由代码实现：

#### 依赖：

```xml
<!--       gateway网关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```



#### 配置式配置：

```yml
#mysql数据库配置
spring:
  application:
    #    应用的别名;向eureka注册时时不能带下划线，会导致服务无法找到
    name: gateway-service
   #gateway网关配置
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名称进行路由
      routes:
        - id: payment_route # 路由的id,没有规定规则但要求唯一,建议配合服务名
          #匹配后提供服务的路由地址
          #          uri: http://localhost:8001   #固定地址则直接写地址
          uri: lb://PAYMENT-SERVICE   #通过服务注册中心获取服务地址则为 lb://服务名
          predicates:
            - Path=/payments # 断言，路径相匹配的进行路由
```

#### 编码时配置：



```java
/**
 * 创建网关的路由
 * @param builder
 * @return
 */
@Bean
public RouteLocator routeLocator(RouteLocatorBuilder builder)
{
    RouteLocatorBuilder.Builder routes = builder.routes();
    RouteLocator routeLocator = routes.route("baidu", r -> r.path("/guonei").uri("https://news.baidu.com")).build();
    return routeLocator;
}
```



### 断言：

上方演示的为路径匹配断言的配置，除此以外gateway支持以下的断言：

![1450135-20191022114243645-371057197](E:\下载\图片\1450135-20191022114243645-371057197.png)









### 过滤器：

gateway网关的过滤器可以分为单个请求路径的过滤器和全局的过滤器两种，单个请求路径的过滤器（**gatewayfilter**）是需要在配置文件的gateway网关配置中进行配置的，只对配置的路由项起作用，全局的过滤器（**global filter**）需要自定义，实现**GlobalFilter**, **Ordered**接口并且注入的ioc中

gateway过滤器配置：

```yml
cloud:
  gateway:
    discovery:
      locator:
        enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名称进行路由
    routes:
      - id: payment_route # 路由的id,没有规定规则但要求唯一,建议配合服务名
        #匹配后提供服务的路由地址
        #          uri: http://localhost:8001   #固定地址则直接写地址
        uri: lb://PAYMENT-SERVICE   #通过服务注册中心获取服务地址则为 lb://服务名
        filters:  #内置过滤器配置
          - AddRequestHeader=X-Request-Foo, Bar  #给请求中增加请求头key为X-Request-Foo value为Bar
        predicates:
          - Path=/payments # 断言，路径相匹配的进行路由
```



global过滤器：

```java
/**
 *自定义网关的过滤器
 */
@Component
public class MyFilter implements GlobalFilter, Ordered
{
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain)
    {
        MultiValueMap<String, String> queryParams = exchange.getRequest().getQueryParams();
        if(!queryParams.containsKey("username")){
            System.out.println("用户非法");
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    /**
     * 定义网关的执行顺序
     * @return
     */
    @Override
    public int getOrder()
    {
        return 0;
    }
}
```

# 服务配置

所有微服务应用配置中的公共配置项进行统一的管理，配置修改后服务不需要重启即可生效。springconfig为所有为服务提供一个中心化的外部配置。配置中心分为客户端和服务端，服务端绑定git仓库地址，获取仓库上配置的配置文件。

## 服务端代码实现：



```yml
spring:
  application:
    name: spring_cloud_config_8096
  cloud:
    config:
      server:
        git:
          uri: https://github.com/mk126616/spring_cloud_config_file.git #GitHub上面的git仓库名字
            ####搜索目录
          search-paths:
            - spring_cloud_config_file   
          skip-ssl-validation: true
          username: mk126616
          password: mk190705
        ####读取分支
      label: master
#config_server作为一个单独的服务也要注册到配置中心，客户端可以通过配置中心获取服务端的地址
eureka:
  client:
    #    将自己注册到eureka上
    register-with-eureka: true
    #    是否从服务端抓取注册信息
    fetch-registry: true
    service-url:
      # 入驻地址
      defaultZone: http://eureka8083.com:8083/eureka/
  instance:
    instance-id: spring_cloud_config_8096  #微服务id
    prefer-ip-address: true #配置中新显示ip
```



```xml
<!--        配置中心-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
```

```java
@EnableConfigServer
@EnableEurekaClient
@SpringBootApplication
public class SpringCloudConfig8096
{
    public static void main(String[] args)
    {
        SpringApplication.run(SpringCloudConfig8096.class, args);
    }
}
```



配置中心服务端从git上把配置文件down下来后进行解析，具体down那个文件是通过访问的url确定，spring规定了配置文件名称，需要遵守规范才能正确的拿到对应的配置文件中的配置项。

### 文件配置标准如下：

​	label代表git上分支名称，application为应用名，profile为所处的环境（dev、tets、prod等，目的是为了不同的环境加载不同的配置文件），通过http请求时直接按照此路径请求即可。

- /{application}/{profile}[/{label}]
- /{application}-{profile}.yml
- /{label}/{application}-{profile}.yml
- /{application}-{profile}.properties
- /{label}/{application}-{profile}.properties

### 自定义解析器：

新建自定义解析器MyPropertiesHandler，继承PropertiesPropertySourceLoader，重写方法

resources文件夹下面新建META-INF文件夹，在里面创建spring.factories文件，指定使用我们自定义的解析器

```
org.springframework.boot.env.PropertySourceLoader=cn.huanzi.qch.config.configserver.MyPropertiesHandler
```





## 客户端代码实现：

**客户端的application.yml中的配置都会被git上同名配置覆盖，因此客户端自己的配置需要配置在bootstrap.yml中，防止被覆盖。**

```xml
<!--        配置客户端依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<!--        web监控-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```java
@RefreshScope //刷新配置中心的配置
@RestController
public class ConfigController
{
    @Value("${custom.key}")
    private String cloudConfigValue;
    @Value("${custom.version}")
    private String version;

    @RequestMapping("/config")
    public  String getConfig(){
        return cloudConfigValue+"_"+version;
    }
}
```



```yml
#客户端的application.yml中的配置都会被git上同名配置覆盖，此处配置必须配置bootstrap.yml中，且只有在spring cloud中才会被识别，优先级比较高，先于application.yml文件加载并且配置项不能被覆盖
spring:
  application:
    name: spring_cloud_config_client_8097
  cloud:
    config:
      label: master #文件所处git分支
      name: application #文件名
      profile: dev  #文件名后缀
#      uri: http://localhost:8096  #配置中心服务config服务地址(不使用注册中时使用，直接写死的地址)
#开启eureka服务发现功能
      discovery:
        enabled: true
        service-id: SPRING_CLOUD_CONFIG_8096
eureka:
  client:
    service-url:
      defaultZone: http://eureka8083.com:8083/eureka/
    fetch-registry: true
    register-with-eureka: true

# 暴露监控端点，目的是git配置修改后刷新本地内存中配置项使用，pom必须引入actuator包
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

**配置中心的客户端上的配置读取的是本地内存中的缓存，在git中配置修改时并不会实时刷新。需要在配置修改后使用post方式去请求客户端的http接口强制客户端刷新缓存（ip:port/actuator/refresh）**。



## 消息总线（spring cloud bus）：

为微服务的所有服务创建一个共同的消息主题，该主题中所有服务都能消费该主题上产生的消息。spring cloud消息总线支持RabbitMQ和kafka.

### 实现spring cloud config 配置中心所有客户端配置的实时同步：

代码实现：

在服务配置中心客户端配置手动刷新版的基础上，需要引入mq配置，暴露通信端点，每次git上的配置修改后只需要手动请求进行配置刷新的http接口即可达到所有配置客户端配置刷新的目的（原理是消息队列中产生一个springCloudBus的交换机，所有spring cloud的配置客户端在rabbitmq上都生成一个消息队列，都订阅了一个主题）。



此处配置随便放在配置中心的客户端和服务端都可以，为了尽量保持单一职责原则一般放在服务端。

```xml
<!--        消息总线-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

```yml
server:
  port: 8096
spring:
  application:
    name: spring_cloud_config_8096
  rabbitmq:
    host: 192.168.62.130
    port: 5672
    username: root
    password: 123456
  cloud:
    config:
      server:
        git:
          uri: https://github.com/mk126616/spring_cloud_config_file.git #GitHub上面的git仓库名字
            ####搜索目录
          search-paths:
            - spring_cloud_config_file
          skip-ssl-validation: true
          username: mk126616
          password: mk190705
        ####读取分支
      label: master

eureka:
  client:
    #    将自己注册到eureka上
    register-with-eureka: true
    #    是否从服务端抓取注册信息
    fetch-registry: true
    service-url:
      # 入驻地址
      defaultZone: http://eureka8083.com:8083/eureka/
  instance:
    instance-id: spring_cloud_config_8096  #微服务id
    prefer-ip-address: true #配置中新显示ip

#rabbitmq暴露bus刷新配置端点
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```

### 

- 配置修改后执行下列命令：

curl -X POST "http://localhost:8096/actuator/bus-refresh"

- 如果需要定点通知则需要修改下访问路径

  curl -X POST "http://localhost:8096/actuator/bus-refresh/微服务名称:端口"



# spring cloud stream

## 作用：

相当于不同消息中间件的适配器，工作在业务和消息中间件之间，屏蔽底层消息中间件的差异，降低切换成本，统一编程模型（目前只支持rabbitmq和kafka）

![WM-Screenshots-20220114200455](E:\图片\WM-Screenshots-20220114200455.png)

source：

channel：stream中发送数据的操作对象，从source/sink中获取。

binder：绑定stream和mq原生接口，进行两表消息模型对象的转换。



## 开发常用api和注解：

![WM-Screenshots-20220114204238](E:\图片\WM-Screenshots-20220114204238.png)



### 生产者代码实现：

```xml
<!--cloud_stream(中间件适配)-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

```yml
server:
  port: 8099

spring:
  application:
    name: cloud_stream_rabbitmq_provider
  cloud:
    stream:
      binders:      #此处开始配置要绑定消息中间件的信息
        defaultRabbit:
          type: rabbit   #组件类型
          environment:
            spring:
              rabbitmq:
                host: 192.168.62.130
                port: 5672
                username: root
                password: 123456
      bindings: #服务的整合处理
        output:
          destination: studyExchange  #交换机名称
          contetType: application/json   #数据类型 文本为：text/plain
          binder: defaultRabbit   #绑定消息服务的名称

eureka:
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka8083.com:8083/eureka/
  instance:
    instance-id: cloud_stream_rabbitmq_provider_8099
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 2 #心跳时间默认30s
    lease-expiration-duration-in-seconds: 5 #心跳超时时间，超时未收到心跳节点会被剔除
```

```java
@EnableBinding(Source.class)
public class MessageServiceImpl implements MessageService
{
    @Resource  //默认按照名称注入，没有找到则按照类型注入(此名称为默认的，可以再bingds中用name项进行配置)
    private MessageChannel output; //output为从Source.class动态代理实现类中获取到的一个MessageChannel


    @Override
    public Result sendMessage(String msg)
    {
        return Result.success(output.send(MessageBuilder.withPayload(msg).build()));
    }
}
```



### 消费者代码实现：

```yml
server:
  port: 8100

spring:
  application:
    name: cloud_stream_rabbitmq_consumer
  cloud:
    stream:
      binders:      #此处开始配置要绑定消息中间件的信息
        defaultRabbit:
          type: rabbit   #组件类型
          environment:
            spring:
              rabbitmq:
                host: 192.168.62.130
                port: 5672
                username: root
                password: 123456
      bindings: #服务的整合处理
        input:
          destination: studyExchange  #交换机名称
          contetType: application/json   #数据类型 文本为：text/plain
          binder: defaultRabbit   #绑定消息服务的名称
          #队列名称,不配置则会生成一个随机的队列名，并且队列不会持久化，消费者宕机时消息会有丢失，并且消息会出现同种服务不同节点都消费的问题
          #配置了队列名则会创建一个持久化的队列，并且为了消息不被同中服务持久化，一定要配置
          group: stream_consumer_queue  #队列名称     
```

```java
@Component
@EnableBinding({Sink.class})
public class MessageController
{
    @StreamListener(Sink.INPUT) //Sink.INPUT是从sink接口动态代理实现类中获取到的一个subscribeChannel信道
    public void receiveMessage(Message msg){
        System.out.println("接收到消息："+msg.getPayload());
    }
}
```

## 消息消费存在的问题：

- 同一种服务的不同节点会存在消息重复消费

- 消息消费这宕机时消息会丢失

  原因：当同一种服务的不同节点消费的group不一样时会生成不同的队列来订阅了同一个topic主题的交换机。因此此问题在同种服务的不同节点配置相同的group即可。此时因为mq中的消息只能被消费一次，因此多个节点是竞争关系。并且配置后消息队列是持久化在mq服务器上的，消费者宕机重启后消息也能被消费。

```yml
bindings: #服务的整合处理
    input:
    destination: studyExchange  #交换机名称
    contetType: application/json   #数据类型 文本为：text/plain
    binder: defaultRabbit   #绑定消息服务的名称
    #队列名称,不配置则会生成一个随机的队列名，并且队列不会持久化，消费者宕机时消息会有丢失，并且消息会出现同种服务不同节点都消费的问题
    #配置了队列名则会创建一个持久化的队列，并且为了消息不被同中服务持久化，一定要配置
    group: stream_consumer_queue  #队列名称     
```





# 服务调用链路追踪

## spring cloud sleuth和zipkin

能够记录一个客户请求在微服务中的调用路径，并展示每一个单个服务的耗时。

![WM-Screenshots-20220115202344](E:\图片\WM-Screenshots-20220115202344.png)



代码实现：

- 安装zipkin服务：docker run -d -p 9411:9411 openzipkin/zipkin
- 需要监控的服务引入jar包

```
<!--sleuth 和zipkin-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

- 配置监控服务地址 

```yml
spring:
#    应用的别名（非项目访问路径），可以不加
  application:
    name: order-service
  zipkin:
    base-url: http://192.168.62.130:9411
    sleuth:
      sampler:
        #采样率0到1之间，1为全部采集
        probability: 1
```





















































































