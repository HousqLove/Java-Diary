# 概述
Spring是一个支持快速开发Java EE应用程序的框架。他提供了一套底层容器和基础设施，并可以和大量常用的开源框架无缝集成。随着Spring越来越受欢迎，在Spring基础上，又诞生了Spring Boot，Spring Cloud，Spring Data，Spring Security等一系列基于Spring Framework。的项目。

Spring Framework主要包括几个模块:
- 支持IoC和AOP的容器
- 支持JDBC和ORM的数据访问模块
- 支持声明式事务的模块
- 支持基于Servlet的MVC开发
- 以及集成JMS，JavaMail，JMX、缓存等其他模块

# IoC容器
容器就是一种为某种特定组件的运行提供必要支持的一个软件环境。如Tomcat就是一个Servlet容器，为Servlet的运行提供运行环境。类似Docker也是一个容器，提供了必要的Linux环境以便运行一个Linux进程。通常容器除了提供环境外还提供许多底层服务。如Servlet容器实现了TCP连接，解析Http协议等非常复杂的服务。

Spring的核心就是提供了一个Ioc容器。 他可以管理所有轻量级的JavaBean组件，提供的底层服务包括组件的生命周期管理，配置和组装服务、AOP支持，以及建立在AOP基础上的声明式事务服务等。

IoC（Inversion of Control）控制反转，又称为依赖注入。在IoC下，组件的创建不再由应用程序而是由IoC容器负责，这样应用程序只要直接使用以及创建并且配置好的组件即可，为了让组件能在IoC容器中装配出来，需要某种“注入”机制，所以又叫依赖注入（DI Dependency Injection）。不直接new一个组件而是注入一个组件带来的好处有：
- 调用者不再关心如果创建组件
- 一个组件被共享变的简单
- 测试容易，可以采用虚拟组件提供功能

IoC还负责管理组件的生命周期。


## XML声明依赖关系
一种最简单的方式告诉容器如何创建组件的方式是通过xml文件来实现，如：
```
<beans>
    <bean id="dataSource" class="HikariDataSource">
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test">
    </bean>
    <bean id="bookService" class="BookService">
        <property name="dataSource" ref="dataSource">

        </property>
    </bean>
    
</bean>
```
> 上诉配置只是IoC创建2个JavaBean组件, 为dataSource组件的jdbcUrl属性注入字符串，并把id为dataSource的组件通过属性dataSource（调用setDataSource方法）注入到bookService组件中。

依赖注入也可以向构造方法注入。基本类型和字符串通过value注入，引用类型通过ref注入，集合类型通过property的子标签<list>或<map>注入。

Spring容器中，把组件称为JavaBean，配置组件就是配置JavaBean。

最后只需要创建一个Spring的Ioc容器，加载配置文件，Spring容器即可为我们创建并装配好配置文件中指定的所有Bean：
```java
ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
```
> 就能从Spring容器中取出Bean了。BookService bookService = context.getBean(BookService.class)

## Application
Spring容器就是ApplicationContext，他是一个接口，有很多实现类，其中ClassPathXmlApplicationContext表示会从classpath中查找指定的xml配置文件。获取Bean可以根据Bean的ID获取，也可以根据Bean的类型获取。

Spring还提供另一种IoC容器叫BeanFactory，使用方式和ApplicationContext类似。区别在于BeanFactory是按需创建，ApplicationContext会一次性创建所有的Bean。

## 注解配置Bean
使用XML的优点是所有的Bean一目了然，并通过配置能直观地看到每个Bean的依赖，确定是写起来繁琐。Spring还提供了一种使用注解的方式配置JavaBean。

常用的注解有：
- @Component：构建定义JavaBean，有一个可选的名称，默认是类名首字母小写。
- @Autowired：自动按照类型注入
- @Configuration：表示一个配置类。
- @ComponentScan：表明容器需要自动搜索当前类所在的包以及子包，把所有标注为@Component的Bean自动创建出来，并根据@Autowired自动装配。
- @Scope：声明组件的声明周期
- @Order：定义List组件在装配时的顺序
- @Bean：声明工厂方法生成组件
- @PostConstruct：标记组件生成并且Autowired后执行的初始化方法
- @PreDestory：标记组角销毁之前的清理方法
- @Qualifier：指定组件别名，或注入指定组件名的组件，在需要多个Bean实例的时候使用
- @Primary：标明主要Bean，在多个Bean实例的情况下，如果没有指定注入的注解就注入Primary组件
- @Value：加载资源文件，参数一般有```classpath:/logo.txt```,```file:/path/to/logo.txt```
- @PropertySource: 读取配置文件，该注解标在class上，在字段上即可使用@Value("${app.zone:Z}")注入。指明key和默认值。还可以把@Value注解写到方法参数上。
> @Value("#{smtpConfig.host}") 表示从JavaBean获取属性

## 条件装配
Spring为应用程序准备了Profile的概念，用来表示不同的环境。创建某个Bean时，Spring容器可以根据注解@Profile来决定是否创建。

除了Profile，还能使用Conditional来决定是否创建某个Bean。用法：
- 使用@Conditional(OnSmtpEnvCondition.class)注解修饰Bean。OnSmtpEnvCondition需实现Condition接口。
> SpringBoot还提供了@ConditionalOnProperty(name="app.smtp", havingValue="true")判断配置文件，@ConditionalOnClass(name = "javax.mail.Transport")判断classpath存在类。

# AOP
AOP是Aspect Oriented Programming的缩写，即面向切面编程。OOP作为面向对象编程的模式，把系统看做多个对象的交互，AOP把系统分解为不同的关注点，称之为切面。如一个业务组件的每个业务方法，除了业务逻辑还需要安全检查，日志记录，事务处理等。这些功能实际上横跨多个业务方法，可以使用代理模式抽取，不过比较麻烦，要先抽取接口，再实现代理类。AOP正式这样的框架，可以把不同的切面植入到业务逻辑中。实际上是对业务逻辑的调用方法进行拦截，并在拦截前后植入切面

Java平台，AOP植入有三种方式：
- 编译期：在编译时，由编译器把切面调用编译进字节码，这种方式需要定义新的关键字并扩展编译器。AspectJ就是扩展了Java编译器，使用关键字aspect来实现植入。
- 类加载期：在目标被加载到JVM时，通过一个特殊的类加载器，对目标类的字节码进行增强。
- 运行期：目标对象和切面都是普通Java类，通过JVM的动态代理功能或第三方库实现运行期动态植入。

Spring的AOP实现就是基于JVM的动态代理。是在容器启动时为我们自动创建注入了Aspect的子类，取代了原始的JavaBean。对接口类型使用JDK动态代理，对普通类使用CGLIB创建子类，如果一个Bean的class是final的，Spring将无法为其创建子类。

## 装配AOP
在AOP编程中，会遇到下面的概念：
- Aspect：切面，即一个横跨多个核心逻辑的功能。
- Joinpoint：连接点，定义在应用程序流程的何处插入切面的执行
- PointCut：切入点，一组连接点的集合
- Advice：增强，指特定连接点上执行的动作
- Introduction：引介，为一个已有的Java对象动态地增加新的接口
- Weaving：织入，将切面整合到程序的执行流程中
- Interceptor：拦截器，一种实现增强的方式
- Target Object：目标对象，真正执行业务的核心逻辑对象
- AOP Proxy：AOP代理，客户端持有的增强后的对象引用

拦截器是定义应在业务逻辑的什么位置执行拦截，织入切面，有以下类型：
- @Before：先执行拦截器代码，再执行目标代码，如果拦截器抛出异常，目标代码就不执行
- @After：先执行目标代码，再执行拦截器代码。无论目标是否异常，拦截器代码都会执行
- @AfterReturning：和@After不同的是，只有当目标代码正常返回时，才执行拦截器代码
- @AfterThrowing：和@After不同的是，只有当目标代码抛出了异常，才执行拦截器代码
- @Around：能完全控制目标代码是否执行，并可在执行前后、抛出异常后执行任意拦截代码，包含上面所有功能。

# Commons-dbutil


# JdbcTemplate
[JdbcTemplate](02_Spring中的JdbcTemplate.md)

# Spring的事务控制
JavaEE体系分层开发，事务处理位于业务层。Spring提供了分层设计业务层的事务处理解决方案。提供了一组事务控制的接口，在spring-tx包中。Spring的事务控制都是基于AOP的。

## PlatformTransactionManager
PlatformTransactionManager是Spring的事务管理器。提供了常用的操作事务的方法：
- getTransaction：获取事务状态信息
- commit：提交事务
- rollback：回滚事务
常用的实现类有:
- org.springframework.jdbc.datasource.DataSourceTransactionManager
- org.springframework.orm.hibernateTransactionManager

## TransactionDefinition
事务的定义信息对象
- getName：获取事务对象名称
- getIsolation：获取事务隔离级别，默认数据库的隔离级别
- getPropagation：获取事务传播行为
- getTimeout：获取超时时间，默认-1没有限制，如果有以秒为单位
- isReadOnly：是否只读

# 响应式编程