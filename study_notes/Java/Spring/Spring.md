# Spring概述

> Spring概述
>

- Spring是**轻量级**、**开源**的JavaEE框架
- Spring可以解决企业应用开发的复杂性
- Spring两大核心：IOC、AOP
  - IOC：控制反转，将对象的创建过程交给Spring进行管理
  - AOP：面向切面，在不修改源代码的基础上进行功能增强



# 日志框架

主流框架：Log4j2



依赖：

```xml
	<dependency>  
        <groupId>org.apache.logging.log4j</groupId>  
        <artifactId>log4j-api</artifactId>  
        <version>2.5</version>  
    </dependency>  
    <dependency>  
        <groupId>org.apache.logging.log4j</groupId>  
        <artifactId>log4j-core</artifactId>  
        <version>2.5</version>  
    </dependency>  

```



- %d{HH:mm:ss.SSS} 表示输出到毫秒的时间
- %t 输出当前线程名称
- %-5level 输出日志级别，-5表示左对齐并且固定输出5个字符，如果不足在右边补0
- %logger 输出logger名称，因为Root Logger没有名称，所以没有输出
- %msg 日志文本
- %n 换行

-   其他常用的占位符有：

    	%F 输出所在的类文件名，如Log4j2Test.java
    	
    	%L 输出行号
    	
    	%M 输出所在方法名
    	
    	%l 输出语句所在的行数, 包括类名、方法名、文件名、行数

```xml
<?xml version="1.0" encoding="UTF-8"?>
 <!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
 <!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出-->
 <!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
 <configuration status="WARN" monitorInterval="30">
     <!--先定义所有的appender-->
     <appenders>
     <!--这个输出控制台的配置-->
         <console name="Console" target="SYSTEM_OUT">
         <!--输出日志的格式-->
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
         </console>
     <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，这个也挺有用的，适合临时测试用-->
     <File name="log" fileName="log/test.log" append="false">
        <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
     </File>
     <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
         <RollingFile name="RollingFileInfo" fileName="${sys:user.home}/logs/info.log"
                      filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log">
             <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->        
             <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
             <Policies>
                 <TimeBasedTriggeringPolicy/>
                 <SizeBasedTriggeringPolicy size="100 MB"/>
             </Policies>
         </RollingFile>
         <RollingFile name="RollingFileWarn" fileName="${sys:user.home}/logs/warn.log"
                      filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log">
             <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
             <Policies>
                 <TimeBasedTriggeringPolicy/>
                 <SizeBasedTriggeringPolicy size="100 MB"/>
             </Policies>
         <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
             <DefaultRolloverStrategy max="20"/>
         </RollingFile>
         <RollingFile name="RollingFileError" fileName="${sys:user.home}/logs/error.log"
                      filePattern="${sys:user.home}/logs/$${date:yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log">
             <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
             <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
             <Policies>
                 <TimeBasedTriggeringPolicy/>
                 <SizeBasedTriggeringPolicy size="100 MB"/>
             </Policies>
         </RollingFile>
     </appenders>
     <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
     <loggers>
         <!--过滤掉spring和mybatis的一些无用的DEBUG信息-->
         <logger name="org.springframework" level="INFO"></logger>
         <logger name="org.mybatis" level="INFO"></logger>
         <root level="all">
             <appender-ref ref="Console"/>
             <appender-ref ref="RollingFileInfo"/>
             <appender-ref ref="RollingFileWarn"/>
             <appender-ref ref="RollingFileError"/>
         </root>
     </loggers>
 </configuration>

```



# IOC

## 概念

控制反转IoC(Inversion of Control)，是一种设计思想，DI(依赖注入)是实现IoC的一种方法

IOC思想基于IOC容器实现，IOC容器底层使用了工厂模式

![未命名白板](..\..\未命名白板.jpg)



# AOP

AOP（**A**spect **O**riented **P**rogramming）：面向切面编程，利用AOP可以对业务逻辑的各个部分进行隔离，从而降低耦合度

AOP实现功能：日志、事务、权限等



## AOP术语

1. 切面（Aspect）：**把通知应用到切入点的过程**，在Aspect中包含一些Pointcut和相应Advice
2. 连接点（Joint point）：**可以被增强的方法**，表示在程序中明确定义的点，包括方法的调用、对类成员的访问以及异常处理程序块的执行
3. 切点（Pointcut）：**实际被增强的方法**，表示一组Joint point，定义了相应的Advice将要发生的地方
4. 增强/通知（Advice）：**方法被增强的逻辑部分**，定义了在Pointcut里面定义的程序点具体要执行的操作，分类：
   - 前置通知
   - 后置通知
   - 环绕通知
   - 异常通知
   - 最终通知
5. 目标对象（Target）：织入Advice的目标对象
6. 织入（Weaving）：将Aspect和其他对象连接起来，并创建Adviced 



## 动态代理

有两种情况的动态代理：

- 有接口的情况：使用JDK动态代理
- 没有接口的情况：使用CGLIB动态代理



Proxy类的newProxyInstance()方法调用：

```java
static Object newProxyInstance(ClassLoader loader, Class<?> interfaces, InvocationHandler h)
```



## 静态代理（AspectJ）

> Spring框架一般都是基于AspectJ实现AOP操作，AspectJ并不是Spring的组成部分，它是独立的AOP框架

通过AspectJ支持的语言编写切面，在**编译时**生成一个被增强了的代理类

配置方式：

- 基于xml配置文件
- 基于注解方式



### 配置（xml文件）

1. 在Spring配置文件中，开启注解扫描

   ```xml
   <context:component-scan base-package="com.su.AOP.AspectJ"></context:component-scan>
   ```

2. 使用注解创建对象：

   ```
   @Component // 声明组件
   ```

3. 在增强类上添加注解@Aspect

4. 配置文件中开启生成代理对象 

   ```xml
   <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
   ```


### 切入点表达式

语法结构：

```java
execution([权限修饰符] [返回类型] [类全路径] [方法名称] ([参数列表]))
```

例：对com.su.dao.UserDao类中的add()进行增强

```java
execution(* com.su.dao.UserDao.add(..))
```



示例：

```java
@Component
@Aspect
@Order(0) // 增强类上使用Order注解，值越小，优先级越高
public class DemoAop {
    // 两种切入点设置方式：String、方法
    private static final String POINT_CUT_1 = "execution(* 路径.方法名(..))";
    private static final String POINT_CUT_2 = "execution(* 路径.方法名(..))";
	
    @Pointcut("execution(* 路径.方法名(..))")
    private void pointCutMethod1();
    
    @Pointcut("execution(* 路径.方法名(..))")
    private void pointCutMethod2();

    // 前置通知
    @Before(value = POINT_CUT_1 + "||" + POINT_CUT_2)
    public void doBefore(JoinPoint joinPoint) {}
    
    // 后置通知
    @After(value = "pointCutMethod1() || pointCutMethod2()")
    public void doAfter(JoinPoint joinPoint) {}
	
    @AfterReturning(value = POINT_CUT_1)
    public void doAfterReturning(JoinPoint joinPoint) {}
    
    @AfterThrowing(value = POINT_CUT_1)
    public void doAfterThrowing(JoinPoint joinPoint) {}
    
    // 环绕通知
    @Around(value = "pointCutMethod1()")
    public void doAround(ProceedingJoinPoint proceedingJoinPoint) throws Throwable{
        // 环绕前
        proceedingJoinPoint.proceed(); 
        // 环绕后
    }
}

```





# 源码解析



## Spring容器

> Spring容器分类

主要包含两种主要容器类型：

- 实现BeanFactory接口的简单容器
- 实现ApplicationContext的高级容器



> ApplicationContext容器介绍

ApplicationContext内部封装了一个BeanFactory对象，来实现对容器的操作，BeanFactory封装了Bean的信息



ApplicationContext在应用DefaultListableBeanFactory对象的基础上，不仅实现了BeanFactory接口提供的功能方法，还黏合了一些面向应用的功能，如资源/国际化支持/框架事件支持等



> 容器对比

- BeanFactory：IOC容器基本实现，是Spring内部使用的接口。加载文件时不会加载对象，获取对象时才会创建
- ApplicationContext：BeanFactory接口的子接口，提供更强大的功能。加载配置文件时就会创建对象

![image-20230216161127198](C:\Users\ALIENWARE\AppData\Roaming\Typora\typora-user-images\image-20230216161127198.png)

ApplicationContext中有两个子类，分别从磁盘和类路径中加载配置文件：

- FileSystemXmlApplicationContext

- ClassPathXmlApplicationContext

![image-20230216161844638](C:\Users\ALIENWARE\AppData\Roaming\Typora\typora-user-images\image-20230216161844638.png)



## 实现流程

使用ClassPathXmlApplicationContext流程：

1. ClassPathXmlApplicationContext初始化

```java
super(parent);
// 设置配置文件 
setConfigLocations(configLocations);
// 创建bean工厂并实例化对象
if (refresh) {
	refresh();
}
```



2. setConfigLocations方法

```java
public void setConfigLocations(@Nullable String... locations) {
		if (locations != null) {
			Assert.noNullElements(locations, "Config locations must not be null");
			this.configLocations = new String[locations.length];
			for (int i = 0; i < locations.length; i++) {
				this.configLocations[i] = resolvePath(locations[i]).trim();
			}
		}
		else {
			this.configLocations = null;
		}
	}
```



3. refresh方法

```java
// 容器预先准备，记录容器启动时间和标记
prepareRefresh();

// 获取bean工厂（其中实现对BeanDefinition的装载）
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

// 往beanFactory设置值，配置bean工厂的标准上下文特性，如类加载器
prepareBeanFactory(beanFactory);

// factory增强
postProcessBeanFactory(beanFactory);

StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");

// factory增强
invokeBeanFactoryPostProcessors(beanFactory);

// 注册用于拦截bean创建过程的BeanPostProcessors
registerBeanPostProcessors(beanFactory);

beanPostProcess.end();

initMessageSource();

// 初始化上下文事件广播
initApplicationEventMulticaster();

onRefresh();

// 注册监听器
registerListeners();

// 实例化非懒加载单例bean对象
finishBeanFactoryInitialization(beanFactory);

finishRefresh();
```









# 面试题

> 聊聊Spring



> bean的生命周期



> 循环依赖



> 三级缓存



> FactoryBean和BeanFactory的区别



> ApplicationContext和BeanFactory的区别



> 设计模式























