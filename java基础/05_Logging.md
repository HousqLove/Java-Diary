# 使用日志打印信息
好处：
- 设置输出样式
- 设置输出级别
- 可以重定向到文件
- 按包名控制日志级别

## Java标准库logging
java标准库内置了日志包java.util.logging。
```java
    Logger logger = Logger.getGlobal();
    logger.info("start process");
    logger.warning("memory is run out...");
    logger.fine("ignored.");
    logger.severe("process will be terminated");
    /**
     * 
        五月 20, 2020 4:11:03 下午 Test01 testLogging
        信息: start process
        五月 20, 2020 4:11:03 下午 Test01 testLogging
        警告: memory is run out...
        五月 20, 2020 4:11:03 下午 Test01 testLogging
        严重: process will be terminated
     * */
```
JDK的Logging定义了7个日志级别，从严重到普通：
- SEVERE
- WARRING
- INFO
- CONFIG
- FINE
- FINER
- FINEST

因为默认级别是INFO，因此INFO级别以下的日志不会被打印出来。调整级别就能屏蔽掉很多调试相关的输出。

使用JAVA标准库局限：
- 在JVM启动时读取配置文件并完成初始化，一旦运行main方法就无法修改配置。
- 配置不方便，需要再JVM启动时传递参数```-Djava.util.logging.config.file=<config-file-name>```

## Commons Logging
Commons Logging是由Apache创建的第三方日志库。

特色是可以通过配置文件指定挂接不同的日志系统。默认情况下，Commons Logging自动搜索并使用Log4j，如果没有找到Log4j，再使用JDK Logging。

使用Commons Logging只需要两步：
1. 通过LogFactory获取Log类的实例。
2. 使用实例方法打印日志。

Commons Logging定义6个日志级别，默认是INFO：
- FATAL
- ERROR
- WARRING
- INFO
- DEBUG
- TRACE

## Log4j
Log4j是一种流行的日志框架。

Log4j是一个组件化的日志系统，架构大致如下：
```java
log.info("user signed in")
|
|--> Appender --> Filter --> Layout --> Console
|
|--> Appender --> Filter --> Layout --> File
|
|--> Appender --> Filter --> Layout --> Socket

```
当我们使用Log4j输出一条日志时，Log4j自动使用不同的Appender把同一条日志输出到不同的目的地。

输出日志的过程中，通过Filter来过滤哪些Log需要被输出，例如仅输出ERROR级别日志。通过Layout来格式化信息，自动添加日期、时间、方法名称等信息。

## SLF4J & Logback
SLF4J类似于Commons Logging, 也是一个日志接口，而Logback类似于Log4j，是一个日志的实现。

SLF4J的日志接口传入的是一个带占位符的字符串，用后面的变量自动替换占位符。
```java
    Logger logger = LoggerFactory.getLogger(getClass());
    logger.info("Set score {} for Person {} ok.", score, p.getName());
```

