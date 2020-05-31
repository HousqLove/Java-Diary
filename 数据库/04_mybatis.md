# 概述
MyBatis是一个优秀的基于Java的持久层框架，内部封装了jdbc，使开发者只需要关注sql语句本身，而不用处理驱动，连接，Statement等繁杂过程。采用ORM的思想解决了实体类和数据库映射的问题。

OGNL：对象图导航语言（Object Graphic Navigation Language）通过对象的取值方法来获取数据，在写法上把get省略：user.username

# 用法
1. 构建SQLSessionFactory：可以通过SQLSessionFactoryBuilder构建。
2. 从SQLSessionFactory中获取SQLSession。
3. 使用SqlSession获取映射器类的动态代理对象
4. 执行动态代理对象的方法
5. 关闭资源

# 作用域和生命周期
理解不同作用域和生命周期类别是至关重要的，错误的使用会导致非常严重的并发问题。
> 依赖注入框架可以创建线程安全的、基于事务的SQLSession和映射器，并将它们注入到你的Bean中。

## SqlSessionFactoryBuilder
这个类可以被实例化、使用和丢弃，一旦创建了SQLSessionFactory就不再需要他了。因此SQLSessionFactoryBuilder实例的最佳作用域是方法作用域（也就是局部方法变量）。可以重用SQLSessionFactoryBuilder来创建多个SQLSessionFactory实例，但最好还是不要一直保留他，以保证所有的xml资源可以被释放给更重要的事。

## SQLSessionFactory
SQLSessionFactory一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃他或者重新创建另一个实例。使用SQLSessionFactory的最佳实践是在应用运行期间不要重复创建多次。因此SQLSessionFactory的最佳作用域是应用作用域。

## SQLSession
每个线程都应该有他自己的SQLSession实例。SQLSession实例不是线程安全的，因此不能被共享，因此他的最佳作用域是请求或方法作用域。绝不能将SQLSession实例引用放在一个类的静态作用域，甚至一个类的实例变量也不行。也不能将SQLSession实例的引用放在任何类型的托管作用域中，比如Servlet框架中的HTTPSession。

## 映射器实例
映射器是一些绑定映射语句的接口。映射器接口的实例是从SqlSession中获得的。虽然从技术层面上来讲，任何映射器实例的最大作用域与请求他们的SQLSession相同，但方法作用域才是映射器最合适的作用域。

# XML配置
mybatis的配置文件要求子标签有固定的顺序，配置文档的顶层结构如下：
- configuration(配置)
    - properties(属性)
    - settings(设置)
    - typeAlias(类型别名)
    - typeHandlers(类型处理器)
    - objectFactory(对象工厂)
    - plugins(插件)
    - environments(环境配置)
        - environment(环境变量)
            - transactionManager(事务管理器)
            - dataSource(数据源)
    - databaseIdProvider(数据库厂商标识)
    - mappers(映射器)

## properties
properties相当于在xml中配置变量，配置好的属性可以通过```${}```在整个配置文件中替换需要动态配置的属性值。如果一个属性在多处配置，则加载顺序如下：
1. 首先读取properties元素体内的property的值。
2. 然后根据properties元素中的resources/url的值读取类路径下属性文件, 并覆盖之前同名属性。
3. 最后读取方法参数传递的属性，并覆盖之前同名属性。

## settings
定义mybatis运行时行为的设置。
```xml
<settings>
  <setting name="cacheEnabled" value="true"/><!--开关所有映射器文件中已配置的任何缓存，默认true-->
  <setting name="lazyLoadingEnabled" value="true"/><!--关联对象的延迟加载开关，默认false-->
  <setting name="multipleResultSetsEnabled" value="true"/>><!--是否允许单个语句返回多结果集，需数据库驱动支持， 默认true-->
  <setting name="useColumnLabel" value="true"/>><!--使用列标签替代列名，默认true-->
  <setting name="useGeneratedKeys" value="false"/>><!--允许jdbc支持自动生成主键，默认false-->
  <setting name="autoMappingBehavior" value="PARTIAL"/>><!--指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。 默认PARTIAL-->
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>><!--指定发现自动映射目标未知列（或未知属性类型）的行为。
NONE: 不做任何反应
WARNING: 输出警告日志（'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior' 的日志等级必须设置为 WARN）
FAILING: 映射失败 (抛出 SqlSessionException)，默认NONE-->
  <setting name="defaultExecutorType" value="SIMPLE"/>><!--配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。默认SIMPLE-->
  <setting name="defaultStatementTimeout" value="25"/>><!--设置超时时间，它决定数据库驱动等待数据库响应的秒数。-->
  <setting name="defaultFetchSize" value="100"/>><!--为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。-->
  <setting name="safeRowBoundsEnabled" value="false"/>><!--是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。默认false-->
  <setting name="mapUnderscoreToCamelCase" value="false"/>><!--是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。-->
  <setting name="localCacheScope" value="SESSION"/>><!--MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。-->
  <setting name="jdbcTypeForNull" value="OTHER"/>><!--当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。默认OTHER-->
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>><!--指定对象的哪些方法触发一次延迟加载。-->
</settings>
```

## environments
mybatis可以配置成适应多种环境，这种机制有助于将SQL映射应用于多种数据库之中。每个SQLSessionFactory实例只能选择一种环境。
```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```
- 默认环境id对应于多个environment定义的id中的其中一个
- 事务管理器配置有两种类型（如果使用spring + mybatis，则不必配置事务管理器，Spring会使用自带的管理器覆盖前面的配置）：
    - JDBC：直接使用JDBC的提交和回滚，依赖从数据源获得的连接来管理事务作用域。
    - MANAGED：从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期。
- dataSource使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。有三种内建的数据源类型：
    - UNPOOLED：每次请求时打开和关闭连接。
    - POOLED：利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。
    - JNDI：为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用

## mappers
用于定义SQL映射语句。
```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

# XML映射文件
# 动态SQL
# JavaAPI
# SQL语句构建器
# 日志
mybatis通过使用内置的日志工厂提供日志功能。内置日志工厂将会把日志委托给下面的实现之一：
- SLF4J
- Apache Commons Logging
- Log4j 2
- Log4j
- JDK logging
mybatis内置日志工厂会基于运行时检测信息选择日志委托实现。会按照上面顺序使用第一个查找到的实现。没有找到时，会禁用日志功能。也可以通过mybatis配置来选择日志实现：
```xml
<configuration>
  <settings>
    ...
    <setting name="logImpl" value="LOG4J"/>
    ...
  </settings>
</configuration>
```
> 可选的值有：SLF4J、LOG4J、LOG4J2、JDK_LOGGING、COMMONS_LOGGING、STDOUT_LOGGING、NO_LOGGING，或者是实现了 org.apache.ibatis.logging.Log 接口，且构造方法以字符串为参数的类完全限定名。

# 连接池
使用连接池可以减少获取连接所消耗的时间。连接池就是一个存储连接的容器，其实就是一个集合对象，该集合必须是线程安全的，不能两个线程拿到同一个连接，还必须实现先进先出的队列特性。

Mybatis连接池提供了三种方式的配置：
- 在主配置文件中使用DataSource标签，使用type属性表示采用何种连接方式，取值有三：
    - POOLED：采用传统的javax.sql.DataSource规范中的连接池，mybatis中有针对规范的实现。
    - UNPOOLED：采用传统连接，实现javax.sql.DataSource接口，但没有采用连接池的思想。
    - JNDI：采用服务器提供的JNDI技术实现。如果不是web或maven的war工程是不能使用的。

# 事务
mybatis自动开启事务。

# 动态sql
mybatis可以通过if、where、set、foreach，choose标签实现动态sql的生成。

详细教程可以查看网站<https://mybatis.org/mybatis-3/zh/index.html>

# 延迟加载