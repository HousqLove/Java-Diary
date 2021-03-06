# 概览
Java EE还提供了Listener（监听器）组件。最常用的是ServletContextListener。

实现Listener需要实现ServletContextListener接口，会在整个Web应用程序初始化完成后，以及Web应用程序关闭后分别回调通知contextInitialized(ServlertContextEvent)和contextDestory(ServletContextEvent)。可以把初始化数据库连接池等工作放在初始化回调方法中，把清理资源的工作放在销毁回调方法中，Web服务器保证contextInitialized执行后才会接受用户请求。

很多第三方框架会通过ServletContextListener接口初始化自己。

除了ServletContextListener，还有几种Listener：
- HTTPSessionListener: 监听HTTPSession创建和销毁事件
- ServletRequestListener：监听ServletRequest请求的创建和销毁事件
- ServletRequestAttributeListener：监听ServletRequest请求的属性变化事件
- ServletContextAttributeListener：监听ServletContext的属性变化事件

## ServletContext
一个Web服务器可以运行一个或多个WebApp，Web服务器都会为其创建一个全局唯一的ServletContext实例。ServletContext可以通过ServletRequest、HTTPSession等对象获得。ServletContext实例最大的作用就是设置和共享全局信息。此外还提供了动态添加Servlet，Filter、Listener等功能。