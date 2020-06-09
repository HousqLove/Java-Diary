# 概述
PageHelper是一款支持数据库分页查询的工具。官方文档见<https://pagehelper.github.io/>, 配合Mybatis使用效果更好。支持任何复杂的单表，多表分页。目前支持Mybatis 3.1.0+。

支持以下数据库的物理分页：
- Oracle
- Mysql
- MariaDB
- SQLite
- Hsqldb
- PostgreSQL
- DB2
- SqlServer(2005/2008)
- Infomix
- H2
- SqlServer 2012
- Derby
- Phoenix
- 达梦数据库(dm)
- 阿里云PPAS数据库
- 神通数据库
- HerdDB
最新信息查看[这里](https://github.com/pagehelper/Mybatis-PageHelper/blob/master/src/main/java/com/github/pagehelper/page/PageAutoDialect.java#L58)

# 使用方法
## 1、 jar包或Maven引入分页插件。
## 2、配置拦截器
新版拦截器是```com.github.pagehelper.PageInterceptor```。PageHelper现在是一个特殊的dialect实现类，是分页插件的默认实现类，提供了和以前一样的用法。
1. 在mybatis配置xml中配置拦截器插件
```xml
<plugins>
    <!-- com.github.pagehelper为PageHelper类所在包名 -->
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <!-- 使用下面的方式配置参数，后面会有所有的参数介绍 -->
        <property name="param1" value="value1"/>
	</plugin>
</plugins>
```
2. 在Spring配置文件中配置拦截器插件
```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <!-- 注意其他配置 -->
  <property name="plugins">
    <array>
      <bean class="com.github.pagehelper.PageInterceptor">
        <property name="properties">
          <!--使用下面的方式配置参数，一行配置一个 -->
          <value>
            params=value1
          </value>
        </property>
      </bean>
    </array>
  </property>
</bean>
```
3. 参数介绍
分页插件提供多个可选参数：
- dialect：默认情况下使用PageHelper方式进行分页，如果想自定义，可以实现```com.github.pagehelper.Dialect```接口，然后配置该属性为实现类的全限定名。

以下参数只针对默认Dialect情况：
- helperDialect：分页插件会自动检测当前的数据量连接，自动选择合适的分页方式。也可以通过该属性指定分页插件使用哪种Dialect。可以使用以下缩写值：oracle、mysql、mariadb、sqlite、hsqldb、postgresql、db2、sqlserver、infomix、h2、sqlserver2012、derby。使用sqlServer2012时必须配置，否则默认使用SqlServer2005。
- offsetAsPageNum：默认false，对使用RowBounds作为分页参数时有效。当设为true时，会将RowBounds中的offset参数当成pageNum使用，可以用页码和页面大小两个参数进行分页。
- rowBoundWithCount：默认false，对使用RowBounds作为分页参数有效。为true时，使用RowBounds分页会进行count查询。
- pageSizeZero：默认false。为true时，如果pageSize=0或RowBounds.limit=0会查询全部，相当于没有分页，结果仍是Page类型。
- reasonable：合理化分页参数，默认false。为true时，pageNum<0会查第一页，pageNum>pages会查最后一页。false时，直接根据参数查询。
- params：为了支持startPage(Object)方法，增加该参数来配置参数映射，用于从对象中根据属性取值，可以配置pageNum, pageSize, count, pageSizeZero, reasonable,不配置映射的用默认值。默认值为：pageNum=pageNum;pageSize=pageSize;count=countSql;reasonable=reasonable;pageSizeZero=pageSizeZero。
- supportMethodArguments：默认false。为true时，支持通过Mapper接口参数来传递分页参数，会从查询方法的参数值中根据params配置的字段中取值，查找到合适的值就会自动分页。
- autoRuntimeDialect：默认false。为true时，允许在运行时根据多数据源自动识别对应Dialect的分页，不支持SqlServer2012.
- closeConn：默认true。当使用运行时动态数据源或没有设置helperDialect自动获取数据库类型时，会自动获取一个数据库连接，通过该属性来设置是否关闭获取的这个连接。
- aggregateFunctions(5.1.5+)：默认为所有常见数据库的聚合函数，允许手动添加聚合函数（影响行数），所有以聚合函数开头的函数，在进行 count 转换时，会套一层。其他函数和列会被替换为 count(0)，其中count列可以自己配置。

当offsetAsPageNum=false时，由于PageNum问题，RowBounds查询的时候reasonable会强制false。使用PageHelper.startPage方法不受影响。

## 3、代码中使用
支持以下几种调用方式：
- RowBounds方式调用：
```java
List<User> list = sqlSession.selectList("x.y.selectIf", null, new RowBounds(0, 10));
```
- Mapper接口方式调用（推荐）：
```java
PageHelper.startPage(1, 10);
List<User> list = userMapper.selectIf(1);
```
- Mapper接口方式调用（推荐）：
```java
PageHelper.offsetPage(1, 10);
List<User> list = userMapper.selectIf(1);
```
- 参数方法调用：
```java
public interface CountryMapper {
    List<User> selectByPageNumSize(
            @Param("user") User user,
            @Param("pageNum") int pageNum, 
            @Param("pageSize") int pageSize);
}
//配置supportMethodsArguments=true
//在代码中直接调用：
List<User> list = userMapper.selectByPageNumSize(user, 1, 10);
```
- 参数对象：
```java
//如果 pageNum 和 pageSize 存在于 User 对象中，只要参数有值，也会被分页
//有如下 User 对象
public class User {
    //其他fields
    //下面两个参数名和 params 配置的名字一致
    private Integer pageNum;
    private Integer pageSize;
}
//存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
public interface CountryMapper {
    List<User> selectByPageNumSize(User user);
}
//当 user 中的 pageNum!= null && pageSize!= null 时，会自动分页
List<User> list = userMapper.selectByPageNumSize(user);
```
- ISelect接口方式：
```java
Page<User> page = PageHelper.startPage(1, 10).doSelectPage(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectGroupBy();
    }
});
//也可以直接返回PageInfo，注意doSelectPageInfo方法和doSelectPage
pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectGroupBy();
    }
});
//count查询，返回一个查询语句的count数
long total = PageHelper.count(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectLike(user);
    }
});
```

PageHelper方法使用了静态的ThreadLocal参数，将分页参数和线程进行绑定的。因此使用时要保证PageHelper方法调用后紧跟Mybatis查询方法。PageHelper在finally代码中自动清除了ThreadLocal存储的对象。可以通过PageHelper.clearPage方法手动清理ThreadLocal存储的分页参数。

# 重要提示
1. 只有紧跟在PageHelper.startPage方法后的第一个Mybatis查询会被分页
2. 不用配置多个分页插件。
3. 分页插件不支持带有for update语句的分页，会跑异常。建议手动。
4. 分页插件不支持嵌套结果映射。嵌套结果方式会导致结果集被折叠，分页查询的结果在折叠后会减少，无法保证分页结果数量正确。

# 继承SpringBoot
集成MyBatis, 分页插件 PageHelper, 通用 Mapper：
## 依赖
```xml
<!--mybatis-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>
<!--mapper-->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>1.2.4</version>
</dependency>
<!--pagehelper-->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.3</version>
</dependency>
```

## Spring DevTools配置
在使用 DevTools 时，通用Mapper经常会出现 class x.x.A cannot be cast to x.x.A。

同一个类如果使用了不同的类加载器，就会产生这样的错误，所以解决方案就是让通用Mapper和实体类使用相同的类加载器即可。

DevTools 默认会对 IDE 中引入的所有项目使用 restart 类加载器，对于引入的 jar 包使用 base 类加载器，因此只要保证通用Mapper的jar包使用 restart 类加载器即可。

在 src/main/resources 中创建 META-INF 目录，在此目录下添加 spring-devtools.properties 配置，内容如下：

restart.include.mapper=/mapper-[\\w-\\.]+jar
restart.include.pagehelper=/pagehelper-[\\w-\\.]+jar
使用这个配置后，就会使用 restart 类加载加载 include 进去的 jar 包。

## 集成Mybatis Generator
通过 Maven 插件集成的，所以运行插件使用下面的命令：
> mvn mybatis-generator:generate

Mybatis Geneator 详解:
<http://blog.csdn.net/isea533/article/details/42102297>

## application.properties配置
```properties
#mybatis
mybatis.type-aliases-package=tk.mybatis.springboot.model
mybatis.mapper-locations=classpath:mapper/*.xml

#mapper
#mappers 多个接口时逗号隔开
mapper.mappers=tk.mybatis.springboot.util.MyMapper
mapper.not-empty=false
mapper.identity=MYSQL

#pagehelper
pagehelper.helperDialect=mysql
pagehelper.reasonable=true
pagehelper.supportMethodsArguments=true
pagehelper.params=count=countSql
```

## application.yml配置
```yml
mybatis:
    type-aliases-package: tk.mybatis.springboot.model
    mapper-locations: classpath:mapper/*.xml

mapper:
    mappers:
        - tk.mybatis.springboot.util.MyMapper
    not-empty: false
    identity: MYSQL

pagehelper:
    helperDialect: mysql
    reasonable: true
    supportMethodsArguments: true
    params: count=countSql
```

# Executor拦截器 - QueryInterceptor规范

## Executor query方法介绍
Executor中的query方法有两个：
```java
<E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey cacheKey, BoundSql boundSql) throw SQLException;

<E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throw SQLException;
```
第一个方法多两个参数CacheKey和BoundSql，如果能拦截第一个方法，可以直接得到BoundSql对象，对SQL做处理。但是参数多的这个方法都是被少的query方法在内部调用的。而我们所有的拦截器都是层层代理的CachingExecutor或基于BaseExecutor的实现类，所以能拦截的就是参数少的方法。

分页插件开始从Executor拦截开始就一直是拦截参数少的这个方法。但是从5.0开始，query的这两个方法都可以被拦截了。

## 拦截器配置和调用顺序
拦截器的调用顺序分两大种。

一种是拦截的不同对象，比如拦截Executor和拦截StatementHandler。在Executor的query方法的执行过程中会调用StatementHandler的query方法，StatementHandler属于Executor执行过程中的一个子过程。所以这两种不同的插件在配置时，一定是先执行Executor的拦截器，然后才会轮到StatementHandler。这种情况，配置拦截器的顺序不重要，Mybatis逻辑上就已经控制了先后顺序。

第二种是拦截同一对象的同一个方法，例如都拦截Executor的query方法，这时候配置拦截器的顺序就会有影响了。首先Configuration.AddInterceptor()方法会按照拦截器配置的顺序依次添加到interceptorChain中，其内部就是List<interceptor> inceptors。然后在Configuration.newExecutor()方法创建Executor，其中会调用interceptorChain.pluginAll()方法。方法如下：
```java
public Object pluginAll(Object target) {
    for (Interceptor interceptor : interceptors) {
        target = interceptor.plugin(target);
    }
    return target;
}
```
所以配置拦截器的顺序是1、2、3。这里也会按照123的顺序被层层代理。所以执行的顺序是3>2>1>executor.query(四个参数)>1>2>3的顺序去执行。

## 拦截query方法的技巧
上一节中对拦截器的用法是最常见的一种用法。但是分页插件5.0不是这样。六个参数的方法不能被代理是因为四个参数的方法会在方法体内部生成另外的两个参数然后调用六个参数的方法。既然CachingExecutor或基于BaseExecutor的实现类能够得到另外的两个BoundSql和CacheKey参数，自己的QueryInterceptor也能够直接替代他拿到参数之后自己调用六个参数方法。这样就会导致四个参数的后续方法被跳过。由于这里的executor是代理对象，六个参数的query方法可以被代理了，同时也扰乱了上一节的执行顺序。

如果将Interceptor2换成QueryInterceptor，则整体的执行逻辑就变成了3>2>executor.query(六个参数)>2>3。如果Interceptor1拦截的是六个参数的方法，则是没有问题的。如果拦截的是四个参数的方法就会被跳过。

分页插件执行的就是类似QueryInterceptor的执行逻辑，所以当使用5.0之后的版本时，需要配置其他Executor的query插件就会遇到一些问题，解决方法看下一节。

## 拦截query方法的规范
QueryInterceptor的逻辑是进去的是四个参数的方法，出去的是六个参数的方法。这种处理方法不方便和一般的Executor拦截器搭配使用，当出现两个以上类似QueryInterceptor的插件时，也无法连贯执行下去。解决方法就是使用统一的规范。
```java
@Intercepts(
    {
        @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
        @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class}),
    }
)
public class QueryInterceptor implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        Object[] args = invocation.getArgs();
        MappedStatement ms = (MappedStatement) args[0];
        Object parameter = args[1];
        RowBounds rowBounds = (RowBounds) args[2];
        ResultHandler resultHandler = (ResultHandler) args[3];
        Executor executor = (Executor) invocation.getTarget();
        CacheKey cacheKey;
        BoundSql boundSql;
        //由于逻辑关系，只会进入一次
        if(args.length == 4){
            //4 个参数时
            boundSql = ms.getBoundSql(parameter);
            cacheKey = executor.createCacheKey(ms, parameter, rowBounds, boundSql);
        } else {
            //6 个参数时
            cacheKey = (CacheKey) args[4];
            boundSql = (BoundSql) args[5];
        }
        //TODO 自己要进行的各种处理
        //注：下面的方法可以根据自己的逻辑调用多次，在分页插件中，count 和 page 各调用了一次
        return executor.query(ms, parameter, rowBounds, resultHandler, cacheKey, boundSql);
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
    }

}
```

首先是拦截器签名同时拦截了四个和六个参数的方法，这样不管哪个插件在前在后都会被执行。第二个是判断参数的长度，如果是四个参数方法进来就获取那两个对象，如果已经被别的插件处理成六个参数的方法了，就直接从参数中取值，这样就保证当其他拦截器对这两个参数做过处理时，这两个参数会继续生效。

## 如何配置不同的Executor插件
当引入类似QueryInterceptor插件时，由于扰乱了原有的插件执行方式，当配置顺序时会导致插件无法生效。

由于QueryInterceptor是四个或六个参数进来，六个参数出去。所以在QueryInterceptor前面执行的拦截器必须是四个参数的（遵循上一节规范的都能执行），在QueryInterceptor后面执行的拦截器必须是六个参数的。这个顺序对应到配置顺序时，也就是四个参数的拦截器在下面，六个参数的拦截器在上面。
```xml
<plugins>
    <!-- 六个参数 -->
    <plugin interceptor="com.github.pagehelper.ExecutorQueryInterceptor1"/>
    <!-- QueryInterceptor本身 -->
    <plugin interceptor="com.github.pagehelper.QueryInterceptor"/>
    <!-- 四个参数 -->
    <plugin interceptor="com.github.pagehelper.ExecutorQueryInterceptor3"/>
</plugins>
```