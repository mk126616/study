# **Spring**

 

# **特点：**

\1. 代码解耦，方便开发 ：ioc控制反转，aop切面编程

\2. Aop支持

\3. 方便测试：整合了junit4测试框架

\4. 扩展性强：方便与其他框架进行整合：比如mybatis、struts2

\5. 方便事务管理

 

# **体系结构:**

 ![image-20220106115046396](C:\Users\19746\AppData\Roaming\Typora\typora-user-images\image-20220106115046396.png)

Spring体系结构大的方面分为5个：test（测试）、core（核心）、aop（切面编程）、data assess（数据访问）、web

test：集成了junit测试框架

core：Beans（bean定义的解析与注册、ioc容器BeanFactory的实现）、Core（ioc和di思想的实现）

Context（ioc容器扩展实现ApplicationContext），Expression（表达式语言，允许使用非java语法去访问和处理对象）

aop: aop(面向切面编程)、aspectj(集成切面编程框架AspectJ)、instrument（aop类级别实现工具）

data assess:jdbc（jdbc的抽象实现，封装了数据库访问中的繁琐的非业务代码），orm（对象关系映射框架集成）、oxm（对象与xml映射）、tx（数据库访问事务控制）、jms（消息服务集成）

web：web（web基础功能如多文件上传支持multipartFile）、webmvc（mvc的实现、以servlet形式启动ioc容器支持）、struts（struts框架集成）、protlet

# **重要接口：**

## **BeanFactoryPostProcessor：**

bean定义的后置处理器接口，可以对bean的定义进行修改

方法：void postProcessBeanFactory(ConfigurableListableBeanFactory var1)，可以修改bean定义

## **BeanDefinitionRegistryPostProcessor:**

继承自BeanFactoryPostProcessor，bean定义的前置和后置处理器

方法：void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry var1)，可增加、修改、判断bean定义

 

# **自定义参数解析器**

## **接口：**

HandlerMethodArgumentResolver：并且还需要注册到ioc中才可生效

## **方法：**

1.public boolean supportsParameter(MethodParameter methodParameter)：

判断此参数解析器是否能支持当前参数解析，返回true会调用解析方法，返回false则直接返回

2.Object resolveArgument(MethodParameter methodParameter, ModelAndViewContainer modelAndViewContainer, NativeWebRequest nativeWebRequest, WebDataBinderFactory webDataBinderFactory) throws Exception ：返回对象直接绑定给当前解析的参数

 

# **自定义全局异常处理器**

## **接口：**

HandlerExceptionResolver ：并且还需要注册到ioc中才可生效

## **方法：**

ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)：传入程序抛入的异常

# **代替web.xml的配置类**

## **接口：**

WebApplicationInitializer

## **方法：**

onStartup(ServletContext var1) ：servlet容器启动时调用，可以在这个类中为应用定义servlet、listener、filter

## **原理：**

tomcat容器启动时会检查所有jar包里面META-INF

文件夹内配置的实现 javax.servlet.ServletContainerInitializer接口的实现类，并调用onStartup方法。SpringServletContainerInitializer方法中调用WebApplicationInitializer接口实现类的onStartup方法

 

# **Spring核心DispatcherServlet**

## **继承结构**

![image-20220106115119919](C:\Users\19746\AppData\Roaming\Typora\typora-user-images\image-20220106115119919.png)

### **GenericServlet**

Serlvet接口的抽象实现

 

## **HttpServlet：**

Service()中根据请求类型分发到doGet、doPost、doPut等方法中

 

## **HttpServletBean:**

从web.Xml或者WebApplicationInitializer中加载配置参数

## **FrameWorkServlet:**

initWebApplicationContext（）方法，创建ApplicationContext实例，默认为XmlWebApplicationContext，AapplicationContext内的refresh（）方法创建ioc容器Beanfactory并对ioc容器要管理的bean进行扫描和生命周期的管理

## **DispatcherServlet****:**

初始化请求映射器、请求适配器、视图解析器、并调配各个组件完成web请求处理

 

 

 

 

 

 

 

Mybatis核心原理：

![img](file:///C:\Users\19746\AppData\Local\Temp\ksohtml\wps18CF.tmp.jpg) 