# 概述
SQL是结构化查询语言的缩写，用来访问和操作数据库系统。SQL语句既可以查询数据库中的数据，也可以添加、更新和删除数据库中的数据，还可以对数据库中进行管理和维护操作。不同的数据库都支持SQL。

SQL语言定义了几种操作数据库的能力：
- DDL：Data Definition Language。DDL允许用户定义数据，也就是常见表，删除表，修改表结构等操作。DDL由数据库管理员执行。
- DML：Data Manipulation Language。DML为用户提供添加、删除、更新数据的能力。这些是应用程序对数据库的日常操作。
- DQL：Data Query Language。DQL允许用户查询数据，这也是通常最频繁的数据库日常操作。

SQL语言不区分大小写。

## 数据类型
对于关系表，除了定义每一列的名称外，还需要定义每一列的数据类型。关系数据库支持的标准数据类型包括数值、字符串、时间等：
| 名称 | 类型 | 说明 |
| :-- | :-- | :-- |
| INT | 整型 | 4字节整数类型 |
| BIGINT | 长整型 | 8字节整数类型 |
| REAL | 浮点型 | 4字节浮点数 |
| DOUBLE | 浮点型 | 8字节浮点数 |
| DECIMAL(M,N) | 高精度小数 | 由用户指定经度的小数，如DECIMAL(20,10)表示共20位，其中小数100位 |
| CHAR(N) | 定长字符串 | 存储指定长度的字符串，如CHAR(100)总是存储100个字符的字符串 |
| VARCHAR(N) | 变长字符串 | 存储可变长度的字符串，如VARCHAR(100)可以存储0~100个字符的字符串 |
| BOOLEAN | 布尔类型 | 存储TRUE或FALSE |
| DATE | 日期类型 | 存储日期，如：2020-05-24 |
| TIME | 时间类型 | 存储时间， 如：12:20：59 |
| DATETIME | 日期和时间类型 | 存储日期+时间 |

上表列出了最常用的数据类型。很多类型还有别名，如REAL又可以写成FLOAT(24)。还有一些不常用的数据类型，如TINYINT（0~255）。各个数据库厂商还会支持特定的数据类型，如JSON。

# 主键
能够通过某个字段唯一区分出不同的记录，这个字段称为主键。记录一旦插入到表中，主键最后不要再修改，否则会造成一系列的影响。选区主键的基本原则是不使用任何业务相关的字段作为主键。一般命名为id。常见作为id字段的类型有：
- 自增整数类型：数据库会在插入数据时自动为每一条记录分配一个自增整数。
- 全局唯一GUID类型：使用一种全局唯一的字符串作为主键。GUID算法通过网卡mac地址、时间戳和随机数保证任意生成的字符串都是不同的。

## 联合主键
关系数据库还允许通过多个字段唯一标识记录，即两个或更多字段都设置为主键，这种主键被称为联合主键。联合主键允许一列有重复，只要不是所有主键列都重复即可。

# 外键
在关系型数据表中，存在一对一，一对多，多对一和多对多的对应关系。在这些关系中，通常在一个表中存储另一个表中某一列的值，来表达对应关系。这种列称为外键。

外键是通过定义外键约束来实现的：
```sql
ALTER TABLE students ADD CONSTRAINT fk_class_id FOREIGN KEY （class_id） REFERENCES classes(id);
```
> fk_class_id表示约束的名称，可以任意指定。class_id表示外键。classes(id)表示指定这个外键关联到classes表的id列。

通过定义外键约束，关系数据库保证无法插入无效的数据。由于外键约束会降低数据库的性能，大部分程序并不设置外键约束，而是仅靠程序自身来保证逻辑的正确性。

删除外键约束：
```sql
ALTER TABLE students DROP FOREIGN KEY fk_class_id;
```
> 删除外键并没有删除这一列。

## 多对多
多对多的关系实际上是通过一个中间表，关联两个一对多关系，来形成多对多的。

## 一对一
一对一关系是指，一个表的记录对应到另一个表的唯一记录。一些应用会把一个大表拆成两个一对一的表，把经常读取和不经常读取的字段分开，以获得更高的性能。

# 索引
索引是关系数据库中对某一列或多个列的值进行预排序的数据结构。通过使用索引，可以让数据库系统不必扫描整个表，而直接定位到复合条件的记录，大大加快查询速度。

创建索引:
```sql
ALTER TABLE students ADD INDEX idx_score(score)
```
> 创建了一个名为idx_score，使用score的索引。索引名称是任意的，如果有多个列可以在括号里依次写上，用逗号分隔。

索引的效率取决于索引列的值是否散列。即该列的值越不相同索引效率越高。可以对一张表创建多个索引。索引在提高查询效率的同时，会在插入、更新和删除时因为要修改索引而降低效率。对于主键，关系数据库会自动对其创建索引。使用主键索引效率最高，因为主键保证绝对唯一。

## 唯一索引
在设计关系数据表时，看上去唯一的列，如身份证号、邮箱地址等，因为有业务含义，不宜作为主键，但根据业务要求，又具有唯一性约束。这时可以给该列添加一个唯一索引：
```sql
ALTER TABLE students ADD UNIQUE INDEX uni_stu_no(stu_no);
```

也可以对某一列添加唯一约束而不创建唯一索引：
```sql
ALTER TABLE students ADD CONSTRAINT uni_stu_no(stu_no);
```

# 查询数据
查询数据库表中的数据，需要使用SELECT语句：
```sql 
SELECT * FROM <表名>; 
```
可以通过执行SELECT 1;来测试数据库连接。

## 条件查询
SELECT语句可以通过```WHERE```来设定查询条件, ```WHERE```后跟条件表达式，可以设置多个条件表达式并用```AND|OR```连接，或```NOT```修饰，查询条件可以使用小括号，查询结果是满足查询条件的记录：
```sql
SELECT * FROM student WHERE (score < 80 OR score > 90) AND gender = 'M';
```
如果不加括号，条件运算按照NOT、AND、OR的优先级进行。

常用的条件表达式：=、>、>=、 <、 <=、 <>（不等于）、 LIKE（判断相似）。

## 投影查询
如果只想查询某些列的数据，可以使用列名代替*来指定查询的列, 指定列的时候还可以指定别名，别名只要跟在列名之后即可。
```sql
SELECT id, score points, name FROM students;
```

## 排序
大部分数据库根据主键排序，使用```ORDER BY```子句可以指定根据其他条件排序，默认ASC正序，加上DESC表示倒序。如果排序列有相同的数据，要进一步排序，可以继续添加列名。:
```sql
SELECT * FROM students ORDER BY score DESC, gender;
```

## 分页查询
查询结果集数据量很大时可以采用分页的形式。可以通过```LIMIT <M> OFFSET <N>```子句实现：
```sql
SELECT * FROM students ORDER BY score DESC LIMIT 3 OFFSET 0;
```
> 上述语句表示，对结果集从0号记录开始，最多取3条。

OFFSET如果超过查询的最大数量，会返回一个空的结果集。分页查询时，随着N越来越大，查询效率也越来越低。

## 聚合查询
SQL提供了专门的聚合函数，使用聚合函数进行查询就是聚合查询。如```COUNT(*)```，聚合的计算结果虽然是一个数字，但查询的结果仍然是一个二维表，只是只有一行一列。使用聚合查询时应该给列名设置一个别名。

SQL还提供如下聚合函数：
| 函数 | 说明 |
| :-- | :-- |
| SUM | 计算某一列的合计值，该列必须为数值类型 |
| AVG | 计算某一列的平均值，该列必须为数值类型 |
| MAX | 计算某一列的最大值 |
| MIN | 计算某一列的最小值 |

如果where没有查询到数据，COUNT会返回0，SUM、AVG、MAX、MIN会返回NULL。

### 分组
对于聚合查询，SQL提供了分组聚合的功能，采用```GROUP BY```子句指定分组依据。
```sql
SELECT class_id, COUNT(*) num FROM students GROUP BY class_id;
```
> 执行该SELECT语句时，会把class_id相同的列先分组，再分别进行计算，有多少group就有多少结果。

也可以对多个列进行分组```GROUP BY class_id, gender```。

## 多表查询
SELECT还可以从多张表同时查询数据。语法是```SELECT * FROM <表1>, <表2>```。表示从多个表的“乘积（各个表中的每一行都两两拼在一起）”中查询，查询的结果也是一个二维表。结果集的列数是各表的列数之和，行数是各表行数之积。这种多表查询又称为笛卡尔查询。

多表查询时，要使用```表名.列名```这样的方式来引用列和设置别名。SQL还允许给表设置一个别名，来让引用简洁一点。
```sql
SELECT 
    s.id sid,
    s.name,
    s.gender,
    s.score,
    c.id cid,
    c.name cname
FROM students s, classes c
WHERE s.gender = 'M' AND c.id = 1;
```

## 连接查询
连接查询是另一种类型的多表查询。连接查询对多个表进行JOIN运算，简单地说，就是先确定一个主表作为结果集，然后把其他表的行有选择地“连接”在主表结果集上。常用的连接：INNER JOIN，RIGHT OUTER JOIN，LEFT OUTER JOIN，FULL OUTER JOIN。

### INNER JOIN：内连接
```sql
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s
INNER JOIN classes c
ON s.class_id = c.id;
```
> 执行结果是会把students表中所有s.class_id = c.id的数据列出来，并在每一行上增加一个名为class_name的列。
INNER JOIN查询的写法：
1. 先确定主表，仍然使用```FROM <表1>```的语法；
2. 再确定需要连接的表，使用```INNER JOIN <表2>```的语法；
3. 然后确定连接条件，使用```ON <条件...>```，这里的条件是```s.class_id = c.id```，表示students表的class_id列与classes表的id列相同的行需要连接；
4. 加上```WHERE、ORDER BY```等子句（可选）。

### OUTER JOIN：外连接
```sql
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM student s
RIGHT OUTER JOIN classes c
ON s.class_id = c.id;
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s
LEFT OUTER JOIN classes c
ON s.class_id = c.id;
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s
FULL OUTER JOIN classes c
ON s.class_id = c.id; 
```
> RIGHT INNER JOIN和INNER JOIN相比，会把classes表中从未匹配到s.class_id = c.id条件的数据添加到结果集中，并把结果集中不是自身的字段设置为NULL。

> LEFT INNER JOIN和INNER JOIN相比，会把students表中从未匹配到s.class_id = c.id条件的数据添加到结果集中，并把结果集中不是自身的字段设置为NULL。

> FULL OUTER JOIN和INNER JOIN相比，会把两个表中从未匹配到s.class_id = c.id条件的数据都添加到结果集中，并把自身不存在的字段设置为NULL。

# 修改数据
修改数据包括增加，更新，删除操作。分别对应INSERT，UPDATE，DELETE语句。

## INSERT
修改数据需要使用INSERT语句，语法是：
```sql
INSERT INTO <表名> (字段1, 字段2, ...) VALUES (值1, 值2, ...);
```
插入一条记录，需要先列举出需要插入的字段名称，然后再VALUES子句中依次写出对应的值。因为id是一个自增主键，不需要列出id字段，如果一个字段有默认值，也可以不列出。VALUES 子句可以指定多条数据，来一次性添加多条记录。

## UPDATE
更新数据需要使用UPDATE语句，语法是：
```sql
UPDATE <表名> SET 字段1 = 值1, 字段2 = 值2, ... WHERE ...;
```
更新字段可以使用表达式。WHERE子句匹配到多条就更新多条记录，没有匹配到任何记录就一条也不更新。如果不写WHERE子句，则表示更新整个表。所以最好先用SELECT语句来测试WHERE条件是否筛选出了期望的记录，然后再更新。

## DELETE
删除数据需要使用DELETE语句，语法是：
```sql
DELETE FROM <表名> WHERE ...;
```
同UPDATE语句一样，谨慎使用WHERE子句，避免匹配到期望之外的记录。

# 管理数据库
以mysql为例：

1. 连接mysql：```mysql -u root -p```，提示输入口令，输入口令正确即可连上MySQL Server, 同时提示符变为mysql:>
2. 断开连接：exit
> 在命令提示符中输入的SQL语句通过TCP连接发送到MySQL Server，默认端口是3306。连接远程需要增加```-h```参数指定地址和端口。和数据库交互本质的方式就是使用SQL。

1. 数据库
    1. 列出所有的数据库：SHOW DATABASES;
    2. 创建新数据库：CREATE DATABASE test;
    3. 删除数据库：DROP DATABASE test;
    4. 切换为当前数据库：USE test;(操作数据库的前提)
2. 表
    1. 列出当前库的所有表：SHOW TABLES;
    2. 查看一个表的结构：DESC students;
    3. 查看创建表的SQL语句：SHOW CREATE TABLE students;
    4. 删除表：DROP TABLE students;
    5. 给表增加列：ALTER TABLE students ADD COLUMN birth VARCHAR(10) NOT NULL;
    6. 修改列：ALTER TABLE students CHANGEC COLUMN birth birthday VARCHAR(20) NOT NULL;
    7. 删除列：ALTER TABLE students DROP COLUMN birth;

## 实用SQL语句

1. 插入或替换：REPLACE INTO students (id, class_id, name, gender, score) VALUES (1, 1, '小明', 'F', 99);//若id = 1记录不存在就插入，存在就先删除再插入。
2. 插入或更新：INSERT INTO students (id, class_id, name, gender, score) VALUES (1, 1, '小明', 'F', 99) ON DUPLICATE KEY UPDATE name='小明', gender='F'; //插入一条记录，如果记录存在就更新改记录。
3. 插入或忽略：INSERT IGNORE INTO students (id, class_id, name, gender, score) VALUES (1, 1, '小明', 'F', 99); //插入一条记录，如果存在就啥也不干
4. 快照：CREATE TABLE students_of_class1 SELECT * FROM students WHERE class_id = 1;//复制一份当前表的数据到一个新表，新创建的表结构和SELECT使用的表结构完全一致。
5. 写入查询结果集：INSERT INTO statistics (class_id, average) SELECT class_id, AVG(score) FROM students GROUP BY class_id;//将查询结果集写入到表中，需确保INSERT的列和SELECT语句的列能一一对应。
6. 强制使用指定索引：SELECT * FROM students FORCE INDEX (idx_class_id) WHERE class_id = 1 ORDER BY id DESC;//查询的时候，数据库系统会自动分析查询语句，并选择一个最合适的索引，但并不一定总能使用最优索引。如果我们知道如何选择，可以指定索引。

# 事务
某些业务要求，一系列操作必须全部执行，不能只执行一部分。这种把多条语句作为一个整体进行操作的功能叫做数据库事务。数据库事务可以确保该事务范围内，所有操作，全部成功或全部失败，如果事务失败，效果就和没有执行这些SQL一样，不会对数据库做任何改动。

数据库具有ACID这4个特性：
- Atomic：原子性，将所有SQL作为原子工作单元，要么全部执行，要么全部不执行。
- Consistent：一致性，事务完成后所有的数据状态都是一致的。
- Isolation：隔离性，如果多个事务并发执行，每个事务作出的修改必须与其他事务隔离。
- Duration：持久性，事务完成后，对数据库的修改被持久化存储。

对于单条SQL语句，数据库系统自动将其作为一个事务执行，这种叫隐式事务。

要手动把多条SQL语句作为一个事务执行，使用BEGIN开启一个事务，使用COMMIT提交一个事务，这种叫显示事务。

COMMIT是指提交事务，即试图把事务内所以SQL所做的修改永久保存。如果COMMIT语句执行失败了，整个事务也会失败。

有时，希望主动让事务失败，可以用ROLLBACK回滚事务，整个事务会失败。

## 隔离级别
对于并发的事务，如果涉及到同一条记录，可能会带来数据的不一致问题，包括脏读、不可重复读，幻读等。数据库系统提供了隔离级别来让我们有针对性的选择事务的隔离级别，避免数据不一致问题。

并发事务可能导致的问题：
- 脏读：一个事务会读到另一个事务更新后未提交的数据，如果另一个事务回滚，那么当前事务读到的数据就是脏数据，就是脏读（Dirty Read）。
- 不可重复读：一个事务内，多次读同一数据，在两次读取之间，有另外的事务恰好修改了这个数据，那么这个事务中，两次读取的数据就可能不一致，就是不可重复读（Non Repeatable Read）。
- 幻读：一个事务中，第一次查询不到的数据，在有一次查询时能够读到，叫做幻读（Phantom Read）。

不同的隔离级别能避免出现不同的问题：
- Read Uncommitted：没有解决任何问题，脏读、不可重复读、幻读都会出现。
- Read Committed： 解决了脏读问题，不可重复读、幻读会出现。
- Repeatable Read：解决了脏读、不可重复读，幻读会出现。
- Serializable：解决了脏读、不可重复读、幻读。
> Serializable是最严格的隔离级别。在Serializable隔离级别下，所有事务按照次序依次执行，因此，脏读、不可重复读、幻读都不会出现。但由于事务是串行执行，效率大大降低。