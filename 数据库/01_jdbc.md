# 概述
JDBC是Java DataBase Connectivity的缩写，是java程序访问数据库的标准接口。

java程序访问数据库时，java代码是通过jdbc接口去访问数据库，而jdbc接口则通过jdbc驱动来实现真正的对数据库的访问。JDBC接口是java标准库自带的，而具体的JDBC驱动是由数据库厂商提供的。因此访问某个具体的数据库时，只需要引入该厂商提供的JDBC驱动，就可以通过JDBC接口访问，就保证了java程序编写一套数据库访问的代码，却可以访问不同的数据库。

# JDBC链接
Connection代表一个JDBC连接，相当于java程序到数据库的连接（通常是tcp），打开一个Connection时，需要准备URL，用户名和口令，才能成功连接到数据库。

# JDBC查询
获取到JDBC连接之后，就可以查询数据库了：
1. 通过connection提供的createStatement()方法创建一个Statement对象，用于执行一个查询。
2. 执行Statement对象的executeQuery()并传入SQL语句，执行查询并获取返回的结果，使用ResultSet来引用这个结果集。
3. 反复调用ResultSet的next()方法读取每一行结果。

Statement和ResultSet都是需要关闭的资源，ResultSet的next()方法用于判断是否有下一行记录，如果有，索引就自动指向下一行（刚获得ResultSet时当前行不是第一行）。ResultSet获取列时，索引从1开始，必须根据SELECT的列调用对应的getxxx()方法，否则将报错。

# SQL注入
使用Statement拼字符串非常容易引发SQL注入问题，这是因为SQL参数通常是从方法参数传入的，如果传入的是一个精心构造的字符串，就可以拼接出意想不到的SQL，而造成意想不到的结果。要避免SQL注入，可以使用PreparedStatement。因为PrepareedStatement始终使用```?```作为占位符，并且把数据连同SQL本身传给数据库。

# 数据类型
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