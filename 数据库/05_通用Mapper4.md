# 概述
通用 Mapper4 是一个可以实现任意 MyBatis 通用方法的框架，项目提供了常规的增删改查操作以及Example 相关的单表操作。通用 Mapper 是为了解决 MyBatis 使用中 90% 的基本操作，使用它可以很方便的进行开发，可以节省开发人员大量的时间。

详细文档参见<https://github.com/abel533/Mapper/wiki>

# 集成

## Spring集成
1. 添加依赖
```xml
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper</artifactId>
    <version>最新版本</version>
</dependency>
```
2. xml配置或注解配置

 xml配置：使用通用Mapper的MapperScannerConfigurer替换mybatis的MapperScannerConfigurer
```xml
<bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="tk.mybatis.mapper.mapper"/>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    <property name="properties">
        <value>
            参数名=值
            参数名2=值2
            ...
        </value>
    </property>
</bean>
```
注解配置：使用通用Mapper的MapperScan注解，比官方注解多两个属性properties和mapperHelperRef。mapperHelperRef优先级更高，只需要选择其中一个即可。
```java
// 使用properties
@Configuration
@MapperScan(value = "tk.mybatis.mapper.annotation",
    properties = {
            "mappers=tk.mybatis.mapper.common.Mapper",
            "notEmpty=true"
    }
)
public class MyBatisConfigProperties {

}

// 使用mapperHelperRef
@Configuration
@MapperScan(value = "tk.mybatis.mapper.annotation", mapperHelperRef = "mapperHelper")
public static class MyBatisConfigRef {
    //其他
 
    @Bean
    public MapperHelper mapperHelper() {
        Config config = new Config();
        List<Class> mappers = new ArrayList<Class>();
        mappers.add(Mapper.class);
        config.setMappers(mappers);

        MapperHelper mapperHelper = new MapperHelper();
        mapperHelper.setConfig(config);
        return mapperHelper;
    }
}
```

## SpringBoot集成
1. 基于starter的自动配置
只需要添加通用Mapper的依赖就完成了基本集成，需要再所有接口上增加@Mapper注解,引入该starter时，和mybatis的starter不会冲突，但是mybatis的starter不会生效。
```xml
<dependency>
  <groupId>tk.mybatis</groupId>
  <artifactId>mapper-spring-boot-starter</artifactId>
  <version>版本号</version>
</dependency>
```
如果需要进行配置，可以在配置文件中配置mapper.前缀的配置：
```yml
mapper:
  mappers:
    - tk.mybatis.mapper.common.Mapper
    - tk.mybatis.mapper.common.Mapper2
  notEmpty: true
```
或
```properties
mapper.mappers=tk.mybatis.mapper.common.Mapper,tk.mybatis.mapper.common.Mapper2
mapper.notEmpty=true
```

SpringBoot支持Relax方式的参数，所以notEmpty可以写成not-empty。

2. 基于@MapperScan注解的配置
可以给待遇@Configuration的类配置@MapperScan注解，或直接配置到SpringBoot的启动类上。
```java
@tk.mybatis.spring.annotation.MapperScan(basePackages = "扫描包")
@SpringBootApplication
public class SampleMapperApplication implements CommandLineRunner {
```

# 对象关系映射
通用Mapper使用JPA注解和自己的注解来实现对象关系映射。

实体类如下：
```java
public class Country {
    @Id
    private Integer id;
    private String  countryname;
    private String  countrycode;

    //省略 getter 和 setter
}
默认情况下将实体类字段安装驼峰转下划线形式的表名列名进行转换。设计到的注解和全局配置：
- @NameStyle（Mapper）：在类上进行配置，配置后对该类和其中的字段进行转换，优先级高于style全局配置，支持以下几个选项：
    - normal：原值
    - camelhump：驼峰转下划线
    - uppercase：转为大写
    - lowercase：转为小写
    - camelHumpAndUppercase：驼峰转下划线并大写
    - camelHumpAndLowercase：驼峰转下划线并小写
- @Table（JPA）：在类上配置映射的表名，可以配置name，catalog，schema，catalog优先级高于schema。
- @Column（JPA）：支持name：配置映射的列名，insertable：对提供的inser方法有效，updateable：对提供的update方法有效。如果使用了关键字，需要用单引号包裹。
- @ColumnType（Mapper）：和@Column作用相同，@Column优先级更高。还提供了jdbcType：用于指定特殊数据库类型，typeHandler：设置特殊类型处理器。
- @Transient（JPA）：用于指定不是表中的字段。默认情况，只有简单类型被自动认为是表中字段。对于复杂对象以及Map、List等不需要这个配置。
- @Id（JPA）：标注字段为主键，联合主键时需每个都标注，如果没有标注@Id字段，当使用ByPrimaryKey方法时，所有的字段会作为联合主键来使用。
- @KeySql（Mapper）：主键策略注解，配置如何生成主键，为了替换@GeneratedValue注解，下节详细介绍。
- @GeneratedValue（JPA）：主键策略注解，配置如何生成主键。
- @Version（Mapper）：乐观锁的注解
- @RegisterMapper（Mapper）：

```
dao类：
```java
import tk.mybatis.mapper.common.Mapper;

public interface CountryMapper extends Mapper<Country> {
    @Select("select * from country where countryname = #{countryname}")
    Country selectByCountryName(String countryname);
}
```

通用Mapper提供了大量的通用接口, 从Mybatis中获取Mapper后就可直接使用，如果要增加自己的方法，可以直接写在Mapper接口中：
- selectOne
- select
- selectAll
- selectCount
- selectByPrimaryKey
- ···

## 主键策略
### JDBC支持通过getGeneratedKeys方法取回主键的情况
需要数据库支持自增，并且提供的JDBC支持getGeneratedKeys方法。
常见的mysql、SqlServer支持这种模式。
```java
@Id
@KeySql(useGeneratedKeys = true)
private Long id;

// 或
@Id
@GeneratedValue(generator = "JDBC")
private Long id;
```
对应生成的xml代码：
```xml
<insert id="insert" useGeneratedKeys="true" keyProperty="id">
    insert into country (id, countryname, countrycode)
    values (#{id},#{countryname},#{countrycode})
</insert>
```
### 支持自增的数据库：
- DB2: VALUES IDENTITY_VAL_LOCAL()
- MYSQL: SELECT LAST_INSERT_ID()
- SQLSERVER: SELECT SCOPE_IDENTITY()
- CLOUDSCAPE: VALUES IDENTITY_VAL_LOCAL()
- DERBY: VALUES IDENTITY_VAL_LOCAL()
- HSQLDB: CALL IDENTITY()
- SYBASE: SELECT @@IDENTITY
- DB2_MF: SELECT IDENTITY_VAL_LOCAL() FROM SYSIBM.SYSDUMMY1
- INFORMIX: select dbinfo('sqlca.sqlerrd1') from systables where tabid=1
这类数据库主键策略配置如下：
```java
@Id
//DEFAULT 需要配合 IDENTITY 参数（ORDER默认AFTER）
@KeySql(dialect = IdentityDialect.DEFAULT)
private Integer id;
//建议直接指定数据库
@Id
@KeySql(dialect = IdentityDialect.MYSQL)
private Integer id;

// 或
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Integer id;
```
对应的xml的形式为：
```xml
<insert id="insertAuthor">
    <selectKey keyProperty="id" resultType="int" order="AFTER">
      SELECT LAST_INSERT_ID()
    </selectKey>
    insert into country (id, countryname, countrycode)
    values (#{id},#{countryname},#{countrycode})
</insert>
```

### 通过序列和任意SQL获取主键值
像Oracle中通过序列获取主键就属于这种情况，除了类似序列获取，还可以是获取UUID的SQL语句，例如：select uuid()。
```java
@Id
@KeySql(sql = "select SEQ_ID.nextval from dual", order = ORDER.BEFORE)
private Integer id;

// 或
@Id
@GeneratedValue(
  strategy = GenerationType.IDENTITY,
  generator = "select SEQ_ID.nextval from dual")
private Integer id;
```
对应的xml代码如下：
```xml
<insert id="insertAuthor">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select SEQ_ID.nextval from dual
  </selectKey>
  insert into country (id, countryname, countrycode)
  values (#{id},#{countryname},#{countrycode})
</insert>
```

# 配置介绍
由于数据库存在各种各样的差异，因此有必要做一些配置。

1. mappers

4.0之前通过mappers配置过的接口才能真正调用，4.0之后增加了@RegisterMapper注解，有了该注解通用Mapper会自动解析所有接口，如果父接口存在该注解的接口，也会自动加上，所以4.0之后不再需要这个参数。如果自己扩展通过接口，建议加上该注解，否则得配置mappers参数。

2. IDENTITY

取回主键的方式。配置时，写为：IDENTITY=MYSQL

3. ORDER（别名order，before）

<selectKey>中的order属性，可选值为BEFORE和AFTER。

4. catalog

数据库的catalog，如果设置该值，查询的时候表名会带catalog设置的前缀。

5. schema

同catalog，catalog优先级高于schema。

6. notEmpty

insertSelective 和 updateByPrimaryKeySelective 中，是否判断字符串类型 !=''。

7. style

实体和表转换时的默认规则

8. enableMethodAnnotation

可以控制是否支持（getter 和 setter）在方法上使用注解，默认false。

9. useSimpleType

默认 true，启用后判断实体类属性是否为表字段时校验字段是否为简单类型，如果不是就忽略该属性，这个配置优先级高于所有注解。

10. usePrimitiveType

为了方便部分还在使用基本类型的实体，增加了该属性，只有配置该属性，并且设置为 true 才会生效，启用后，会扫描 8 种基本类型。

11. simpleTypes

默认的简单类型在 SimpleTypeUtil 中，使用该参数可以增加额外的简单类型，通过逗号隔开的全限定类名添加。

12. enumAsSimpleType

用于配置是否将枚举类型当成基本类型对待。

默认 simpleType 会忽略枚举类型，使用 enumAsSimpleType 配置后会把枚举按简单类型处理，需要自己配置好 typeHandler。

13. wrapKeyword

配置后会自动处理关键字，可以配的值和数据库有关。

例如 sqlserver 可以配置为 [{0}]，使用 {0} 替代原来的列名。

MySql 对应的配置如下：

wrapKeyword=`{0}`
使用该配置后，类似 private String order 就不需要通过 @Column 来指定别名。

14. checkExampleEntityClass

默认 false 用于校验通用 Example 构造参数 entityClass 是否和当前调用的 Mapper<EntityClass> 类型一致。

假设存在下面代码：

Example example = new Example(City.class);
example.xxx...;//设置条件的方法
countryMapper.selectByExample(example);
注意，这里使用 City 创建的 Example，本该使用 cityMapper 来调用，但是这里使用了 countryMapper，默认情况下会出现字段不匹配的错误，更特殊的情况下会正好匹配字段，但是却操作错了表！

配置该字段为 true 后就会对不匹配的情况进行校验！

配置如下：

checkExampleEntityClass=true

15. safeDelete

配置为 true 后，delete 和 deleteByExample 都必须设置查询条件才能删除，否则会抛出异常。

配置如下：

safeDelete=true

16. safeUpdate

配置为 true 后，updateByExample 和 updateByExampleSelective 都必须设置查询条件才能删除，否则会抛出异常（org.apache.ibatis.exceptions.PersistenceException）。

17. useJavaType

设置 true 时，参数中会增加 javaType 设置，如 {id, javaType=java.lang.Long}。在 <resultMap> 中也会设置 javaType 属性。

配置如下：

useJavaType=true

# 代码生成器
## 专用代码生成器
## 通用代码生成器

# 扩展通用接口

# Example用法

# 其他配置
## 二级缓存配置
## TypeHandler