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

一种最简单的方式告诉容器如何创建组件的方式是通过xml文件来实现，如：
```
<beans>
    <bean id="dataSource" class="HikariDataSource"></bean>
    <bean id="bookService" class="BookService">
        <property name="dataSource" ref="dataSource">

        </property>
    </bean>
    
</bean>
```
> 上诉配置只是IoC创建2个JavaBean组件，并把id为dataSource的组件通过属性dataSource（调用setDataSource方法）注入到bookService组件中。

Spring容器中，把组件称为JavaBean，配置组件就是配置JavaBean。

