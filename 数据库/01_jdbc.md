# 概述
JDBC是Java DataBase Connectivity的缩写，是java程序访问数据库的标准接口。

java程序访问数据库时，java代码是通过jdbc接口去访问数据库，而jdbc接口则通过jdbc驱动来实现真正的对数据库的访问。JDBC接口是java标准库自带的，而具体的JDBC驱动是由数据库厂商提供的。因此访问某个具体的数据库时，只需要引入该厂商提供的JDBC驱动，就可以通过JDBC接口访问，就保证了java程序编写一套数据库访问的代码，却可以访问不同的数据库。

## JDBC链接
Connection代表一个JDBC连接，相当于java程序到数据库的连接（通常是tcp），打开一个Connection时，需要准备URL，用户名和口令，才能成功连接到数据库。

# JDBC查询
获取到JDBC连接之后，就可以查询数据库了：
1. 通过connection提供的createStatement()方法创建一个Statement对象，用于执行一个查询。
2. 执行Statement对象的executeQuery()并传入SQL语句，执行查询并获取返回的结果，使用ResultSet来引用这个结果集。
3. 反复调用ResultSet的next()方法读取每一行结果。

Statement和ResultSet都是需要关闭的资源，ResultSet的next()方法用于判断是否有下一行记录，如果有，索引就自动指向下一行（刚获得ResultSet时当前行不是第一行）。ResultSet获取列时，索引从1开始，必须根据SELECT的列调用对应的getxxx()方法，否则将报错。

## SQL注入
使用Statement拼字符串非常容易引发SQL注入问题，这是因为SQL参数通常是从方法参数传入的，如果传入的是一个精心构造的字符串，就可以拼接出意想不到的SQL，而造成意想不到的结果。要避免SQL注入，可以使用PreparedStatement。因为PrepareedStatement始终使用```?```作为占位符，并且把数据连同SQL本身传给数据库。

## 数据类型
JDBC在java.sql.Types定义了一组常量来表示如何映射SQL数据类型：
| SQL数据类型 | java数据类型 |
| :-- | :-- |
| BIT,BOOL | boolean |
| INTEGER | int |
| BIGINT | long |
| REAL | float |
| FLOAT,DOUBLE | double |
| CHAR,VARCHAR | String |
| DECIMAL | BigDecimal |
| DATE | java.sql.Date,LocalDate |
| TIME | java.sql.Time,LocalTime |

# JDBC更新
数据库操作总结起来就是CRUD：Create、Retrieve、Update和Delete。增删改查。

## 插入
插入操作是INSERT，实际上也是使用PreparedStatement的executeUpdate()方法执行SQL语句，执行成功之后返回int类型的值，表示插入的记录数量。如果要获取自增主键，需要再创建PreparedStatement时指定一个Statement.RETURN_GENERATED_KEYS的标志位，并通过getGeneratedKeys()方法返回的ResultSet查看结果（自增值不一定有一个，也不一定是主键）。

## 更新
更新操作是```UPDATE```语句，可以一次更新多条语句，在JDBC代码层面上跟插入操作没有区别，除了SQL语句，返回实际更新的行数。

## 删除
删除是```DELETE```语句，可以一次删除若干列，和更新一样，只有SQL语句不同。

# JDBC事务
数据库事务(Transaction)由若干SQL语句构成的一个操作序列。数据库系统保证在一个事务中的SQL要么全部执行，要么全部不执行，即数据库事务具有ACID特性：
- Atomicity：原子性
- Consistency：一致性
- Isolation：隔离性
- Durability：持久性

数据库事务可以并发执行，而数据库系统从效率考虑，对事务定义了不同的隔离级别。SQL标准定义了4中隔离级别，分别对应可能出现的数据不一致的情况：
| Isolation | 脏读(Dirty Read) | 不可重复读(Non Repeatable Read) | 幻读(Phantom Read) |
| :-- | :-- | :-- | :-- |
| Read Uncommitted | YSE | YES | YES |
| Read Committed | - | YES | YES |
| Repeatable Read | - | - | YES |
| Serializable | - | - | - |

在JDBC中执行事务，就是把多条SQL包裹在一个数据库事务中执行。开启事务的关键代码是setAutoCommit(false)，表示关闭自动提交。提交事务的代码在执行完指定的若干条SQL语句后，调用commit()。如果事务提交失败，回抛出SQL异常，我们必须捕获并调用rollback()回滚事务。最后在finally中setAutoCommit(trur)恢复状态。

设定事务的隔离级别，通过```connection.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);```

# JDBC Batch
JDBC操作数据库的时候经常会执行一些批量操作。SQL数据库对SQL语句相同，但只要参数不同的若干语句可以作为batch执行，这种操作有特别优化，速度远远快于循环执行每个SQL。

通过在给Prepareds set参数之后调用PreparedStatement的addBatch()添加一组SQL的参数，然后调用executeBatch()执行batch，返回值是int数组，数组内保存每组参数执行后影响的结果数量。

# JBDC连接池
数据库的链接是一种昂贵的系统资源，为了避免频繁的创建和销毁JDBC连接，可以通过连接池(Connection Pool)复用已经创建好的连接。

JDBC连接池有一个标准的接口javax.sql.DataSource。要使用连接池，还要选择一个接口的实现，常用的JDBC连接池有：
- HikariCP
- C3P0
- BoneCP
- Druid

DataSource也是一个昂贵的操作，通常DataSource实例总是作为一个全局变量存储，并贯穿整个应用的声明周期。通过getConnection()获取一个连接，调用close()方法时，并不是关闭连接，而是释放到连接池中，以便下次获取连接时能直接返回。

连接池内维护了若干个Connection实例。