



# Spring Boot作用

帮助开发者快速构建一个项目，自动管理项目jar包依赖，简化spring配置，内置tomcat。

# 项目依赖

## 父项目

spring boot业务项目需要继承spring提供的父项目**spring-boot-starter-parent**，父项目帮我们定义了所有相关组件的jar包版本。

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```



## 启动器

spring boot提供了44种启动器，根据自身业务需要引入相应的启动器即可，启动器中已经定义了该场景所需要的所有jar包(**我们也可以定义自己想要的版本进行覆盖**)

示例如下：

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
</dependencies>
```



# 使用

## 项目启动：

采用中的**main**方法启动，只需要增加***@SpringBootApplication*** 注解即可，main方法默认必须放在业务代码包的外层包中才能对业务代码进行管理，如果没有放在最外层需要是使用**@scanBasePackages **定义要扫描的包。

```java
package com.mk;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBootStart
{
    public static void main(String[] args)
    {
        SpringApplication.run(SpringBootStart.class,args);
    }
}

```

## @SpringBootApplication注解

此注解为spring的启动类的标注注解，并且是一个继承了**@SpringBootConfiguration**、**@EnableAutoConfiguration**、**@ComponentScan**的复合注解。

1. **@SpringBootConfiguration**注解继承了**@Configuration**注解标识一个类spingboot项目的配置类。**@Configuration**继承了**@Component**将配置类作为spring容器中的一个组件。
2. **@EnableAutoConfiguration**注解继承了**@AutoConfigurationPackage**注解（扫描依赖组件自动配置类）还继承了**@Import({AutoConfigurationImportSelector.class})**（自动配置类的选择器）

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```



自动配置类定义在**autoConfigure** jar(此包中包含了我们业务依赖组件配置类)包中的**spring.factories**中

![image-20211225104403105](C:\Users\19746\AppData\Roaming\Typora\typora-user-images\image-20211225104403105.png)

## 配置文件

spring boot 默认支持**application.properties**和**application.yml** 两种配置文件并且名称为固定的。

### application.properties

```properties
user.name=张三
user.age=18
user.dept.name=大数据
user.dept.code=X6457
user.list=v1,v2
user.map.k1=v1
user.map.k2=v2
```

### application.yml：

```yml
user:
  name: 张三
  age: 18
  dept:
    name: 大数据
    code: X6457
  list:
    - v1
    - v2
  map: {k1: v1,k2: v2}
```

## bean中取值的注解：

| @Value                            | @ConfigurationProperties                                     |
| --------------------------------- | :----------------------------------------------------------- |
| 对单个属性使用                    | 对注册到ioc中的对象使用（加在bean注册处  -》类或者@bean方法上） |
| 不支持松散绑定                    | 可以支持松散绑定（比如有个值的key为user-name，在代码中使用key为userName时可以取到值） |
| 支持spring的spel表达式语言        | 不支持                                                       |
| 不支持                            | 支持JSR303校验（类上增加@Validated，属性上增加属性值类型的注解，对值得格式进行校验，如@Email、@NotNull） |
| 不支持list、map这类复杂对象的注入 | 支持复杂对象注入                                             |







## 配置文件导入

| @PropertySource    | @ImportResource                                              |
| ------------------ | ------------------------------------------------------------ |
| 引入properties文件 | spring boot中没有也不能识别spring 的xml，如需要使用则用此注解引入spring xml配置文件（spring boot不推荐此方式，可以使用配置类方式替代） |





## 配置文件切换

### properties文件方式

```properties
#开发、测试、线上环境可以配置不同的配置文件如：application-dev.properties、application-prod.properties
#改变此参数后 可以使用不同的配置文件
spring.profiles.active=dev
```



### yml文件方式

```yml
server:
  port: 8085
#设置不同的环境变量可以切换配置块
spring:
  profiles:
    active: prod    
#分割不同的配置块
---
server:
  port: 8081
spring:
  profiles: dev
#分割不同的配置块
---
server:
  port: 8082
spring:
  profiles: prod
```

项目运行时可以增加命令行参数去修改配置的变量：

```txt
java -jar springboot-1.0-SNAPSHOT.jar --spring.profiles.active=devjava -jar springboot-1.0-SNAPSHOT.jar  --spring.profiles.active=dev
```

## 配置文件件加载位置

配置文件可以放在不同路径下，但是不同的路径配置文件有优先级，高优先级会覆盖低优先级中相同的配置项，也可以运行时增加参数

**--spring.config.loacation=<路径>**   修改默认的配置文件路径，也可以用此命令加载jar包外部的配置文件。

默认配置路径如下：

![image-20211225154718937](C:\Users\19746\AppData\Roaming\Typora\typora-user-images\image-20211225154718937.png)

可用配置文件的优先级：

```txt
1.命令行参数

所有的配置都可以在命令行上进行指定；
多个配置用空格分开； –配置项=值
　　java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087 --server.context-path=/abc

2.来自java:comp/env的JNDI属性 

3.Java系统属性（System.getProperties()） 

4.操作系统环境变量 

5.RandomValuePropertySource配置的random.*属性值

6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件

7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件

8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件

9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件

10.@Configuration注解类上的@PropertySource 

11.通过SpringApplication.setDefaultProperties指定的默认属性
```



## 自动配置原理

**@SpringBootApplication** 标识的启动类在启动时加载会加载预定义在各个启动器jar包内**META-INFO**中**factories.properties**各种自动配置类。自动配置类中会检查条件是否满足来判断此自动配置类是否生效。

以http编码处理器为例子：

```java
@Configuration(proxyBeanMethods = false) //当修改为true时当前配置类启用代理(cglb ConfigurationClassEnhancer的intercept方法中直接使用代理对象调用super方法， 此类中this指向代理对象，此类中的@Bean方法可以相互调用获取的都是ioc中同一个对象）；当为false时@Bean方法相互调用会获取到不同的bean
@EnableConfigurationProperties(HttpProperties.class) //启动指定类的@ConfigurationProperties 注解功能绑定值给指定类的bean
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)  //判断是否是web项目
@ConditionalOnClass(CharacterEncodingFilter.class) //jar包中是否有编码过滤器
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true) //配置文件中是否配置了spring.http.encoding=true ( matchIfMissing = true 标识，无论是否配置spring.http.encoding=true都为true)
public class HttpEncodingAutoConfiguration {

   private final HttpProperties.Encoding properties;
	//当bean只有一个构造参数时，会调用此构造参数实例化bean并且依赖的bean会从ioc中获取
   public HttpEncodingAutoConfiguration(HttpProperties properties) {
      this.properties = properties.getEncoding();
   }

   @Bean
   @ConditionalOnMissingBean
   public CharacterEncodingFilter characterEncodingFilter() {
      CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
      filter.setEncoding(this.properties.getCharset().name());
      filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
      filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
      return filter;
   }

   @Bean
   public LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
      return new LocaleCharsetMappingsCustomizer(this.properties);
   }

   private static class LocaleCharsetMappingsCustomizer
         implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>, Ordered {

      private final HttpProperties.Encoding properties;

      LocaleCharsetMappingsCustomizer(HttpProperties.Encoding properties) {
         this.properties = properties;
      }

      @Override
      public void customize(ConfigurableServletWebServerFactory factory) {
         if (this.properties.getMapping() != null) {
            factory.setLocaleCharsetMappings(this.properties.getMapping());
         }
      }

      @Override
      public int getOrder() {
         return 0;
      }

   }

}
```



### @conditional

使用此注解时指定一个类（如**@Conditional(OnBeanCondition.class)**），spring会调用此类中的**match（）**方法判断是否满足条件，满则则将bean加入容器。

由此注解衍生出来了更具体的判断注解，如：@ConditionalOnMissingBean（判断容器中没有这个类型的bean则加入容器）、@ConditionalOnWebApplication判断是否为web环境

### 开启自动配置启动报告

在配置文件中开启后就能在启动日志中看到已开启和未开启的自动配置类

```properties
#开启debug模式，开启后能看到自动配置类生效报告
debug=true
```

## 日志框架

为了达到项目中底层日志实现变化后，不修改项目代码的目的，日志框架都提供一个抽象层（包括接口和抽象实现），各个日志框架厂商去实现接口和抽象层与日志实现层的绑定器（slf4j的为StaticLoggerBinder）。（springboot默认使用slf4j+logback、spring默认使用common-logging）

![image-20211225184927604](C:\Users\19746\AppData\Roaming\Typora\typora-user-images\image-20211225184927604.png)

### 系统日志统一

因为不同的框架底层使用了不同的日志框架，为了让系统日志框架的统一，在导入其他框架时需要排除该框架本身使用的日志jar包，然后引入中间适配新引入框架架日志和我们指定的统一日志的jar包即可。



## web开发

spring boot对于web开发相关的自动配置类为**WebMvcAutoConfiguration.class**

### 静态资源映射

- 第三方的静态文件（如jquery）

  前端资源如js等可以以jar包形式引入（https://www.webjars.org/  可以在此网站获取对应的前段资源的webjar） 。

  ```xml
  <!--        jquery webjar-->
          <dependency>
              <groupId>org.webjars.npm</groupId>
              <artifactId>jquery</artifactId>
              <version>3.6.0</version>
          </dependency>
  ```

  下图为**WebMvcAutoConfiguration.class**中添加静态资源映射部分代码， 前段请求**/webjars/****会对应jar包中的前段资源会在**classpath:/META-INF/resources/webjars/**目录下查找

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
   if (!this.resourceProperties.isAddMappings()) {
      logger.debug("Default resource handling disabled");
      return;
   }
   Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
   CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
   if (!registry.hasMappingForPattern("/webjars/**")) {
      customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
            .addResourceLocations("classpath:/META-INF/resources/webjars/")
            .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
   }
   String staticPathPattern = this.mvcProperties.getStaticPathPattern();
   if (!registry.hasMappingForPattern(staticPathPattern)) {
      customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
            .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
            .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
   }
}
```

- 开发者自己的静态文件

  开发者自身的静态文件会在以下五个路径下寻找（在**ResourceProperties.class**文件中定义）

  ```txt
  "classpath:/META-INF/resources/"
  "classpath:/resources/" 
  "classpath:/static/", 
  "classpath:/public/"
  "/"   ：当前项目根路径
  ```

- 自定义静态资源映射

  ```properties
  #spring默认有静态资源路径，我们也可以设置静态资源映射
  spring.resources.staticLocations = classpath:/static/
  ```





## 修改springboot默认的组件配置



- 覆盖springboot自带的配置或者增加一个配置

  springboot在自动配置时会用**@conditional **以及子类型注解进行条件判断，对于容器中只能有一个的组件则会判断用户是否自定义了该组件。对于可以有多个的组件（如视图解析器viewresolver）会将用户定义的和自带的合并起来放到ioc中。

  **因此我们在使用springboot时只需要通过配置类给容器中增加自定的组件间即可。**

  ```txt
  @conditional  底层原理：
  1.conditionnal注解中指定的value属性对应的类必须实现condition接口并重写matches（）方法
  2.为在容器上下文的refresh方法中会调用ioc容器的BeanDefinitionRegistryPostProcessor接口实现类，spring利用此接口实现类进行条件判断调用condition接口并重写matches（）方法根据返回的boolean值修改BeanDefinition定义达到根据条件判断是否加载当前配置到ioc容器中的bean对象。
  ```

  

- 对springboot默认的配置进行扩展

  springboot提供了配置的适配器（比如xxxxConfigurer、xxxxCustomizer）我们可以继承此类并且重写其中的对应组件的扩展方法即可（实现类上要标注**@Configuration**）。

- 当在配置类上增加**@EnableWebMvc**时  springboot自身的webmvc自动配置将全部失效，所有配置我们都需要自己配置、



## 注册listener、filter、servlet

springboot采用jar包嵌入式web容器，在ioc中增加如下类型对象后spring自动将其注册到web容器上。

```java
@Configuration
public class MyServerConfig
{
    /**
     * 给tomcat增加servlet，spring的DispatcherServlet也是这么注册的
     */
    @Bean
    public ServletRegistrationBean servletRegistrationBean()
    {
        ServletRegistrationBean servletRegistration = new ServletRegistrationBean();
        servletRegistration.setServlet(new MyServlet());
        String[] urlMapping = {"/myServlet"};
        servletRegistration.setUrlMappings(Arrays.asList(urlMapping));
        return servletRegistration;
    }

    /**
     * 给tomcat增加filter
     */
    @Bean
    public FilterRegistrationBean filterRegistrationBean()
    {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new MyFilter());
        String[] urlMapping = {"/myFilter"};
        filterRegistrationBean.setUrlPatterns(Arrays.asList(urlMapping));
        return filterRegistrationBean;
    }

    /**
     * 给tomcat增加listener
     */
    @Bean
    public ServletListenerRegistrationBean listenerRegistrationBean()
    {
        return new ServletListenerRegistrationBean(new MyServletContextListener());
    }
}
```





## 切换web容器

springboot目前支持（tomcat、jetty、undertow），springboot默认使用tomcat，切换嵌入式web容器时只需要排除tomcat的启动器，引入jetty或者别的servlet容器的启动器即可

tomcat处理客户端的连接时采取bio，jetty和undertow默认采取nio。tomcat更适合处理短连接，jetty适合处理io繁忙的长连接。

```xml
<!--        web启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>
<!--        使用jetty作为web容器-->
        <dependency>
            <artifactId>spring-boot-starter-jetty</artifactId>
            <groupId>org.springframework.boot</groupId>
        </dependency>
```



## 嵌入式web容器启动原理



```java
第一步：
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Configuration
@ConditionalOnWebApplication
@Import(BeanPostProcessorsRegistrar.class)
public class EmbeddedServletContainerAutoConfiguration {

	/**
	 * Nested configuration if Tomcat is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Tomcat.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedTomcat {
		//给ioc中注入web容器工厂
		@Bean
		public TomcatEmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() {
			return new TomcatEmbeddedServletContainerFactory();
		}

	}
第二步：
//spring boot入口方法执行时会创建嵌入式容器类型的ioc的context，
SpringApplication.run(SpringBootStart.class, args);
    //此容器的refresh方法中从ioc容器中获取
    protected void onRefresh() {
    super.onRefresh();

    try {
        this.createEmbeddedServletContainer();
    } catch (Throwable var2) {
        throw new ApplicationContextException("Unable to start embedded container", var2);
    }
}
 第三步：
private void createEmbeddedServletContainer() {
     EmbeddedServletContainer localContainer = this.embeddedServletContainer;
     ServletContext localServletContext = this.getServletContext();
     if (localContainer == null && localServletContext == null) 
         //从ioc中获取容器工厂
         EmbeddedServletContainerFactory containerFactory = this.getEmbeddedServletContainerFactory();
     	//调用容器工厂的方法生成容器并启动
     	this.embeddedServletContainer = containerFactory.getEmbeddedServletContainer(new ServletContextInitializer[]				{this.getSelfInitializer()});
 		} else if (localServletContext != null) {
     	try {
         this.getSelfInitializer().onStartup(localServletContext);
     	} catch (ServletException var4) {
         throw new ApplicationContextException("Cannot initialize servlet context", var4);
    	 }
 	}

    this.initPropertySources();
    } 
//第四步：
BeanPostProcessorsRegistrar.class   //此后置处理器在run方法中法中被调用
xxxxCustomizer     //后置处理器中ioc里面所有servlet容器的定制化器被调用处理web容器的配置
```







## 使用外部servlet容器

### 使用步骤

1. 打包方式指定为war包

   ```xml
   <groupId>com.example</groupId>
   <artifactId>demo</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <packaging>war</packaging>
   <name>demo</name>
   <description>Demo project for Spring Boot</description>
   ```



2.tomcat启动器增加参数指明使用使用外部tomcat

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <!--            增加参数指定使用外部容器-->
    <scope>provided</scope>
</dependency>
```



3.springboot同级增加servlet初始化器

**初始化器的作用主要是传入主配置类信息，springapplication的run方法中解析配置类的各种注解，并扫描配置类所在包以及子包下标注的组件注册到ioc中**

```java
public class ServletInitializer extends SpringBootServletInitializer
{

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application)
    {
        return application.sources(DemoApplication.class);
    }

}
```

### 原理：

servlet3.0以后增加了一个约定，web项目启动后会创建每个jar包下**META-INF/services/javax.servlet.ServletContainerInitializer**文件内指定的类的实例（文件名为全类名并且必须实现**ServletContainerInitializer**接口），并且将此类上**@HandlesTypes({xx.class})**要加载的类型传入重写的接口**onstartup()**方法中。 

```java
@HandlesTypes({WebApplicationInitializer.class})
public class SpringServletContainerInitializer implements ServletContainerInitializer {
    public SpringServletContainerInitializer() {
    }

    public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext) throws ServletException {
        List<WebApplicationInitializer> initializers = new LinkedList();
        Iterator var4;
        if (webAppInitializerClasses != null) {
            var4 = webAppInitializerClasses.iterator();

            while(var4.hasNext()) {
                Class<?> waiClass = (Class)var4.next();
                if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) && WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
                    try {
                        initializers.add((WebApplicationInitializer)waiClass.newInstance());
                    } catch (Throwable var7) {
                        throw new ServletException("Failed to instantiate WebApplicationInitializer class", var7);
                    }
                }
            }
        }

        if (initializers.isEmpty()) {
            servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
        } else {
            servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
            AnnotationAwareOrderComparator.sort(initializers);
            var4 = initializers.iterator();

            while(var4.hasNext()) {
                WebApplicationInitializer initializer = (WebApplicationInitializer)var4.next();
                initializer.onStartup(servletContext);
            }

        }
    }
}
```



springboot 的**SpringBootServletInitializer**实现了**WebApplicationInitialize接口**并且在**onstartup()**方法中调用之前我们在main方法中执行的run方法。我们只需要继承方法并且重写configure()方法将springboot启动入口类传入即可。

```java
public class ServletInitializer extends SpringBootServletInitializer
{

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application)
    {
        return application.sources(DemoApplication.class);
    }

}
```

## springboot数据访问

### 默认数据源

springboot数据源自动可以自动配置，我们只需要在配置文件中配置相应的数据库参数即可。

默认数据源：BasicDatasource、HikarDatasource、org.apache.tomcat.hdbc.pool.Datasource

```yml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.62.130:3306/mk_test_database
    driver-class-name: com.mysql.jdbc.Driver
```

### 集成其他类型的数据库连接池

以druid为例

```yml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.62.130:3306/mk_test_database
    driver-class-name: com.mysql.jdbc.Driver
    #druid配置
    type: com.alibaba.druid.pool.DruidDataSource
    maxCreateTaskCount: 10
    initialize: true
```

```java
@Configuration
public class DruidConfig
{
    //创建数据库连接池
    @ConfigurationProperties(prefix = "spring.datasource")   //绑定配置文件中的参数
    @Bean
    public DataSource dataSource(){
        return new DruidDataSource();
    }
    //druid监控
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(new StatViewServlet(),"/druid/*");
        Map<String,String> initParam = new HashMap<String, String>();
        initParam.put("loginUsername","root");
        initParam.put("loginPassword","123456");
        initParam.put("allow","");
        initParam.put("deny","");
        registrationBean.setInitParameters(initParam);
        return registrationBean;
    }
    //druid过滤器
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new WebStatFilter());
        Map<String,String> initParam = new HashMap<String, String>();
        initParam.put("exclusions","*.js,*.css,*.html,/druid/*");
        filterRegistrationBean.setInitParameters(initParam);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/*"));
        return filterRegistrationBean;
    }
}
```



### spring-data-jpa

- jpa概念：java持久层接口，实现对象和数据库表映射（映射关系可以通过注解或者xml进行配置），并提供了统一的接口去操作数据库，我们操作时只需要创建交界口类实现jpa框架架提供的接口后，调用预定义的接口即可（底层为动态代理）。

- 实体关系映射

  ```java
  /**
   * spring-data-jpa测试
   */
  @Entity  //标识当前类为一个jpa数据库字段与实体映射类
  @Table(name = "jpa_test_person")
  public class JpaTestPerson
  {
      @Id //标识主键
      private Integer id;
  
      @Column(name="name")
      private String name;
  
      public Integer getId()
      {
          return id;
      }
  
      public void setId(Integer id)
      {
          this.id = id;
      }
  
      public String getName()
      {
          return name;
      }
  
      public void setName(String name)
      {
          this.name = name;
      }
  }
  ```



- 数据操作接口

  ```java
  /**
   * jpa访问
   * 底层为动态代理
   */
  public interface JpaPersonCrud extends JpaRepository<JpaTestPerson,Integer>
  {
  }
  ```





## springboot原理

- SpringApplication类

  **SpringApplication**为核心类，其run方法中进行ioc容器的创建与refresh（）、初始化器（**ApplicationContextInitializer**）的调用，自动配置类加载生效并将组件注册到ioc容器。

  

- SpringApplication类：initialize方法

  ```
  //此方法在SpringApplication的构造方法中被调用目的是加载类路径下META-INFO文件夹下的spring.factory中定义的组件
  private void initialize(Object[] sources) {
     if (sources != null && sources.length > 0) {
        this.sources.addAll(Arrays.asList(sources));
     }
     this.webEnvironment = deduceWebEnvironment();
     //加载ApplicationContextInitializer（spring容器的初始化器）
     setInitializers((Collection) getSpringFactoriesInstances(
           ApplicationContextInitializer.class));
     //加载ApplicationListener（springboot应用的监听器）
     setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
     this.mainApplicationClass = deduceMainApplicationClass();
  }
  ```

  

- SpringApplication类：run方法

  springboot应用启动核心方法

  ```
  public ConfigurableApplicationContext run(String... args) {
     //项目启动耗时记录器
     StopWatch stopWatch = new StopWatch();
     stopWatch.start();
     ConfigurableApplicationContext context = null;
     FailureAnalyzers analyzers = null;
     //设置java系统无显示器时也可以启动
     configureHeadlessProperty();
     //类路径下加载应用启动过程的监听器SpringApplicationRunListener（监听器中的各个方法都会在ioc容器创建并初始化的各个阶段完成后调用）
     SpringApplicationRunListeners listeners = getRunListeners(args);
     //开始启动时调用监听器的listener，监听器不只是一个工具，更是一种代码的设计模式，可以使得代码具有高可扩展性
     listeners.starting();
     try {
     //将命令行参数封装为一个bean
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
              args);
        //将命令行参数封装到配置环境bean中
        ConfigurableEnvironment environment = prepareEnvironment(listeners,
              applicationArguments);
        //打印springboot启动logo
        Banner printedBanner = printBanner(environment);
        //根据项目环境是否够为web环境创建对应的上下文对象
        context = createApplicationContext();
        //失败分析器
        analyzers = new FailureAnalyzers(context);
        //调用ApplicationContextInitializer上下文初始化器、注册命令行参包装类注册到ioc中、我们创建的springboot配置类在此处注册到BeanDefinitioRegister中
        prepareContext(context, environment, listeners, applicationArguments,
              printedBanner);
       //扫描bean定义扫描并出创建、注册ApplicationContext事监听器
        refreshContext(context);
        //回调ioc中的CommandLineRunner和ApplicationRunner
        afterRefresh(context, applicationArguments);
        listeners.finished(context, null);
        stopWatch.stop();
        if (this.logStartupInfo) {
           new StartupInfoLogger(this.mainApplicationClass)
                 .logStarted(getApplicationLog(), stopWatch);
        }
        return context;
     }
     catch (Throwable ex) {
        handleRunFailure(context, listeners, analyzers, ex);
        throw new IllegalStateException(ex);
     }
  }
  ```

- 事件监听机制

  **SpringApplicationRunListener**：sprin项目启动到初始化完毕的监听器，贯穿整个项目启动过程，自定义此监听器需要在类路径的META-INF目录下的**spring.factories**文件中配置，如下所示：

  ```txt
  # Run Listeners
  org.springframework.boot.SpringApplicationRunListener=\
  org.springframework.boot.context.event.EventPublishingRunListener
  ```

  **ApplicationContextInitializer**：spring项目上下文的初始化，生效于**prepareContext()**方法中，可以对容进行初步的初始化操作。自定义此初始化器需要在类路径的META-INFO目录下的**spring.factories**文件中配置，如下所示：

  ```txt
  # Application Context Initializers
  org.springframework.context.ApplicationContextInitializer=\
  org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
  org.springframework.boot.context.ContextIdApplicationContextInitializer,\
  org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
  org.springframework.boot.context.embedded.ServerPortInfoApplicationContextInitializer
  ```

  **ApplicationRunner**：生效于afterRefresh()方法中，调用时从io容器中获，因此我们直接实现此接口并注册到ioc中即可，可以拿到封装的容器参数
  
  **CommandLineRunner**：生效于afterRefresh()方法中，调用时从io容器中获，因此我们直接实现此接口并注册到ioc中即可，可以拿到命令行参数
  
  

## 自定义启动器

springboot推荐的启动器模式为：启动器和自动配置分为两个jar包，启动器只负责引入组件资源依赖的jar包和自动配置类所在jar包，自动配置类负责创建资源对象并注册到ioc中。

自动配置类所在包具体实现：



自动配置所在jar包META-INF文件夹下创建spring.factories文件，并如下所示进行配置，这样ioc进行refresh()就会能获取到类路径下配置并创建bean并且注册到ioc中。

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mk.spring.boot.autoconfigure.HelloServiceAutoConfiguration
```



自动配置类

```java
@Configuration
@ConditionalOnWebApplication
@EnableConfigurationProperties({HelloProperties.class})
public class HelloServiceAutoConfiguration {
    @Autowired
    private HelloProperties helloProperties;

    public HelloServiceAutoConfiguration() {
    }

    @Bean
    public HelloService helloService() {
        return new HelloService(this.helloProperties.getPrefix(), this.helloProperties.getSuffix());
    }
}
```

properties文件（和springboot的配置文件绑定）

```
@ConfigurationProperties(
    prefix = "com.mk"
)
public class HelloProperties {
    private String prefix;
    private String suffix;

    public HelloProperties() {
    }

    public String getPrefix() {
        return this.prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return this.suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```































