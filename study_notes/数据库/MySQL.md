# 初始数据库

## **SQL概述**

SQL：结构化查询语言（Structured Query Language）是关系型数据库的标准语言

## **数据库分类**

关系型数据库：使用关系模型（二维表格模型）来组织数据的数据库

- MySQL
- Oracle
- SQL Server
- DB2
- SQLite

非关系型数据库（NOSQL）：数据以对象的形式存储在数据库中，对象之间的关系通过每个对象自身的属性来决定

- Redis
- MongDB

**DBMS（数据库管理系统）**

数据库管理系统(Database Management System)是一种操纵和管理数据库的大型软件，用于建立、使用和维护数据库。

## **MySQL**

**基本概念**

MySQL软件提供了一个快速、多线程、多用户的SQL数据库服务器



# MySQL数据库

## 系统库

- information_schema：信息数据库，保存MySQL服务器所维护的所有其他数据库信息，如：数据库名、数据库表、表字段的数据类型和访问权限等

- mysql：MySQL核心数据库，主要负责存储数据库的用户、权限设置、关键字等MySQL需要的控制和管理信息

- performance_schema：内存数据库

- sys库：系统配置

  

## **MySQL安装**

1. 下载mysql community 8版本，下载后添加文件夹下的bin目录到环境变量
2. 管理员模式下进入CMD
3. 安装mysql服务：mysqld -install
4. 初始化数据库文件：mysql --initialize-insecure --user=mysql
5. 启动mysql：net start mysql

## **基本操作**

### **登录退出**

```mysql
-- 连接数据库     
# 用户名：root     
# 密码：123456 
mysql -u root -p123456; 
-- 连接服务器上的数据库 
mysql -h <host/localhost> -u <user> -p; 
Enter password:******; 
-- 退出数据库 
exit;
```



### **更改密码**

```mysql
-- 修改用户密码 
update mysql.user  set authentication_string=password('此处填写修改的密码')  where user='username' and Host='localhost'; 
-- 使用mysql8 
alter user root@localhost identified by '123456';
```



### **数据库操作**

```mysql
-- 创建 
create database [if not exits] `name`;  
-- 删除 
drop database [if exits] `name`; 
-- 使用 
use `name`;  
-- 查看 
show databases; -- 查看所有数据库
```



# 数据库表

## **属性**

**数值**

- tinyint		1个字节
- smallint   		2个字节
- mediumint 	3个字节
- **int			4个字节**
- bigint		8个字节
- float			4个字节
- double		8个字节
- decimal		字符串形式浮点数

**字符串**

- char			0~255
- **varchar 		0~65535**
- tinytext		2^8-1
- **text			2^16-1**

**时间日期**

- date			year-month-day
- time			hour:minute:second
- datetime		year-month-day hour:minute:second
  - 支持的日期范围：1000-01-01 00:00:00 ~ 9999-12-31 23:59:59

- timestamp	year-month-day hour:minute:second
  - timestamp是UTC时间（协调世界时）
  - 支持的日期范围：1970-01-01 00:00:01 ~ 2038-01-19 03:14:07

- year			年份表示

**枚举类型**

- enum		enumeration表示枚举

```mysql
-- enum 为枚举类型，数据类型后跟可选选项 
CREATE TABLE tickets (    
    title varchar(50),
    priority ENUM('Low', 'Medium', 'High') NOT NULL ); 
-- 插入数据 
INSERT INTO tickets(title, priority) VALUES('Upgrade Windows OS for all computers', 1);
```

**二进制文件**

- blob			

- - binary large object
  - 常用来存储图片或声音文件

**时间戳**

- timestamp（N）
- N的取值为0-6，默认为0
- - 时间戳，记录精确时间
  - 最高可达微秒级别（百万分之一秒）

------



## 存储引擎

### **InnoDB（默认）**

优势：

DML操作遵循ACID原则，事务具有提交、回滚和崩溃恢复功能，以保护用户数据

**区别**

|            | MYISAM | INNODB |
| ---------- | ------ | ------ |
| 事务支持   | 不支持 | 支持   |
| 数据行锁定 | 不支持 | 支持   |
| 外键       | 不支持 | 支持   |
| 哈希索引   | 支持   | 不支持 |
| 表空间     | 小     | 大     |



# 数据定义语言（DDL）

> DDL：Data Definition Language，用于对数据库内部对象的创建、修改和删除

## **表的操作**

```mysql
-- 创建    #表的名字、类型、数据、编码方式 
CREATE TABLE [IF NOT EXISTS] `student1` ( 
    `id` INT(4) NOT NULL AUTO_INCREMENT COMMENT '学号', 
    `age` INT(3) NOT NULL COMMENT '年龄', 
    PRIMARY KEY(`id`) 
)ENGINE=INNODB CHARSET=utf8 
-- 删除 
drop table [if exists] `name` 
-- 查看 
show tables 
-- 查看数据库中所有的表 
describe student --展示数据库中表的字段信息 
-- 修改    
# 改表名 
alter table `name` rename as `replaceName`     
# 改字符集 
alter table `name` default charset utf8    
# 改引擎 
alter table `name` engine InnoDB
```

## **字段的操作**

```mysql
-- 增加 
alter table `tableName` add `columnName` dataType() 
-- 删除 
alter table `tableName` drop `columnName`   
-- 修改    
# 修改字段名字 
alter table `name` change `oldname` `newName` dataType()     
# 修改数据类型 
alter table `name` modify `columnName` dataType()     # 设置自增 
alter table `name` modify `columnName` int auto_increment
```



# 数据操作语言（DML）

**DML语言（操作）**

**概念**

DML是Data Manipulation Language的缩写，意思是**数据操纵语言**，是指在SQL语言中，负责对数据库对象**运行数据访问**工作的指令集，以INSERT、UPDATE、DELETE三种指令为核心，分别代表插入、更新与删除。

**操作**

```mysql
-- 插入 
insert into `name`(`field1`,...) values ('',...) 
-- 修改 
UPDATE `student` SET `name`='gentleman' WHERE id<>4 UPDATE `student` SET `birthday`=CURRENT_TIME WHERE pwd='222222' && `name`='lady' 
-- 删除    
# 删除数据 
delete from `name` where 条件   
delete from `name` where id=4    
# 清空数据 
delete from `name` -- 清空数据，计数器不变 
truncate `name` -- 清空数据，改变计数器
```



# 数据查询语言（DQL）

**DQL查询数据**

DQL:Data Query Language 数据查询语言

------

## **select语法**

```mysql
SELECT [ALL | dinstinct] {* | table.* | table.field1[AS alias1][,table.field2[AS alias2]][,...]} 
FROM table_name [AS table_alias] 
[LEFT |RIGHT | INNER JOIN table_name2] 
-- 联合查询 
[WHERE ...] 
-- 指定条件 
[GROUP BY ...] 
-- 指定结果按照哪几个字段来分组 
[HAVING ] 
-- 过滤分组的记录满足的次要条件 
[ORDER BY ...] 
-- 指定查询记录按若干个条件排序 
[LIMIT {[OFFSET,]ROW_COUNT | row_countoffset OFFSET}] -- 指定查询记录范围
```

**执行顺序**

from -> on -> join -> where -> group by -> having -> select -> distinct -> order by ->  limit

通过where，从from子句中找出**元组**，按select选出元组中的属性值形成**结果表**。

group by按列名的值进行**分组**，该属性列值相等的元组为一个组（通常在每组中会作用**聚合函数**）。having进一步筛选，只有满足指定条件的组才进行输出。

order by将结果表按照列名的值进行升序或降序排序。

------

## **单表查询**

**选择列**

```mysql
-- 查询所有列信息 
SELECT * FROM student 
-- 查询特定信息 
SELECT `studentno`,`sex` FROM student 
-- 查询计算过的信息 
SELECT `name`, 2022 - `age` FROM student -- 计算学生出生年份  
-- 给字段取别名 
SELECT `studentno` AS `学号` FROM student 
-- 选择目标可以算术表达式、字符串常量、函数等 
SELECT CONCAT('姓名：',studentname) AS 新名字， 'Year of Birth' FROM student
```

**过滤元组**

```mysql
-- 消除重复行 
SELECT DISTINCT Sno FROM student 
-- where语句 
select `studentno` from student where id>4
```

**where查询条件**

- 比较大小

- 确定范围：（Not） Between .... And ...

- 确定集合：In

- 字符匹配：Like、Escape

- - Like匹配串通配符：%代表任意长度字符串、_代表任意单个字符
  - 若查询字符串本身就含有通配符%或_，就需要使用Escape对通配符进行转义

**排序**

order by：对查询结果按照一个或多个属性列的升序或降序排列

ASC：升序（默认情况：升序）

DESC：降序

二级排序

SELECT employee_id, salary, department_id FROM employees ORDER BY department_id DESC, salary ASC

**聚集函数**

COUNT(*)                          -- 统计元组个数 COUNT([DISTINCT|ALL] <列名>)      -- 统计一列中值的个数 SUM([DISTINCT|ALL] <列名>)        -- 计算一列值的总和 AVG()、MAX()、MIN()同理  

**Group By子句**

将查询的结果按照列的值进行分组，值相等为一组

作用：细化聚集函数的作用对象

-- 查询每门课的学生人数 SELECT Cno, Sno FROM SC GROUP BY Cno

**having**

having短语指定筛选条件

SELECT Sno FROM SC GROUP BY Sno HAVING COUNT(*) > 3

**where和having的区别**

- where作用于表或视图
- having作用于组（条件表达式可以是聚集函数）

**区别1：**

where可以直接使用表中的字段作为筛选条件，但不能使用分组中的聚合函数作为筛选条件；

having需要与group by配合使用，可以把分组中的聚合函数和分组字段作为筛选条件。

**区别2：**

如果需要通过连接从关联表中获取需要的数据，where是先筛选后连接，having是先连接后筛选。

------

## **连接查询**

**等值、非等值连接**

连接条件（连接谓词）：通过where子句连接两个表的条件

等值连接：连接运算符为=

非等值连接：连接运算符不为=

**嵌套循环连接：**从连接表中选择第一个元组，逐一查找被连接表中满足条件的元组，满足条件则拼接元组成为结果表中的一个新的元组。

自然连接：在等值连接中把目标列重复属性列去掉

**自身连接**

-- 自连接 SELECT f.`categoryname` AS '父栏目', s.`categoryname` AS '子栏目' FROM `category` AS f, `category` AS s WHERE f.`categoryid` = s.`pid`

**联表查询**

​    ![0](https://note.youdao.com/yws/res/1999/WEBRESOURCEfbba40f4201c77330eee533dd9e4ec75)

SELECT s.`StudentNo`, `StudentName`, `Sex`, `StudentResult` FROM student AS s LEFT JOIN result AS r ON s.`StudentNo` = r.`StudentNo` WHERE r.`StudentResult` IS NULL

**on和where的区别**

on：作用在生成临时表时，无论表达式是否为真，都会返回左表的元组

where：作用在生成临时表后，对元组进行过滤操作

**多表连接**

多表连接：两个表以上的连接

连接方式：通常是先进行两个表的连接操作，再将结果与第三个表进行连接

------

## **子查询（嵌套查询）**

查询块：一个select-from-where语句

嵌套查询：将一个查询块嵌套在l另一个查询块的where子句或having短语的条件中

**分类**

1. 返回结果的条目数

单行子查询 vs 多行子查询

1. 子查询是否依赖于父查询

相关子查询（子查询的查询条件依赖于父查询） vs 不相关子查询（子查询的查询条件不依赖于父查询）

**in谓词的子查询**

in限定查询所在的范围

一般形式：

SELECT .. FROM .. WHERE .. IN    (        SELECT ..        FROM ..        WHERE ...                )

**比较运算符的子查询**

一般形式：

SELECT .. FROM .. WHERE .. >=    (        SELECT ..        FROM ..        WHERE ...                )

当子查询返回多个结果时需要使用比较运算符，如

SELECT .. FROM .. WHERE .. >= ANY -- A表示大于等于任意一个值即成立 -- ALL 表示大于等于所有值才成立    (        SELECT ..        FROM ..        WHERE ...                )

**集合查询**

由于select语句的查询结果是元组的集合，因此可以结果可进行集合操作。

集合操作主要包括：并操作（union）、交操作（intersect）、差操作（except）

union将多个select语句的查询结果合并到同一个集合中。

**基于派生表的查询**

子查询不仅可以出现在where子句中，还可以出现在from子句中，此时的子查询生成的派生表成为查询对象。

**分页和排序**

```mysql
-- asc升序、desc降序 
SELECT `StudentName`, `StudentResult`  FROM `result` r INNER JOIN `student` s WHERE r.`StudentNo` = s.`StudentNo` 
-- 排序 
ORDER BY `StudentResult` ASC    
-- 分页   
limit 1,2    -- 起始位置，页面大小
```





# 事务

事务是一组操作的集合，是不可分割的工作单位。

事务会把所有操作作为一个整体向系统提交或撤销操作请求，这些操作要么同时成功，要么同时失败



典型示例：银行转账

A->B:1000元

B->A:1000元



## **ACID原则**

**原子性（Atomicity）**

多个步骤一起成功或者一起失败

**一致性（Consistency）**

事务符合逻辑运算

**隔离性（Isolation）**

数据库为每个用户开启的事务之间相互隔离

**永久性（Durability）**

事务一旦提交就不可逆



## **隔离导致的问题**

**脏读**

一个事务读取了另外一个事务未提交的数据

**不可重复读**

在一个事务内，读取表中的某一行数据，多次读取结果不同

**虚读（幻读）**

在一个事务内，读取到了别的事务插入的数据

**命令行操作**

```mysql
-- 关闭自动提交 
SET autocommit = 0; 
-- 开始事务 
START TRANSACTION 
-- 操作数据 
UPDATE account SET money = money - 500 WHERE `name` = 'A' UPDATE account SET money = money + 500 WHERE `name` = 'B' 
-- 提交事务 
COMMIT; 
-- 回滚 
ROLLBACK; 
-- 恢复默认值 
SET autocommit = 1;
```



# 索引

Mysql的索引是在存储引擎层实现的，不同的存储引擎有不同的结构

## 索引数据结构

| 索引                |                                                | InnoDB        | MyISAM | Memory |
| :------------------ | ---------------------------------------------- | :------------ | :----- | :----- |
| B+Tree              | 最常见的索引类型，大部分引擎都支持B+树索引     | 支持          | 支持   | 支持   |
| Hash                | 用哈希表实现，只能精确匹配，不支持范围查询     | 不支持        | 不支持 | 支持   |
| R-Tree(空间索引)    | MYISAM的特殊索引类型，主要用于地理空间数据类型 | 不支持        | 支持   | 不支持 |
| Full-text(全文索引) | 建立倒排索引，快速匹配文档                     | 5.6版本后支持 | 支持   | 不支持 |

### 二叉树

![二叉树](https://dhc.pythonanywhere.com/media/editor/%E4%BA%8C%E5%8F%89%E6%A0%91_20220316153214227108.png)

### 红黑树

红黑树，又叫二叉平衡树

二叉树的缺点可以用红黑树来解决：
![红黑树](https://dhc.pythonanywhere.com/media/editor/%E7%BA%A2%E9%BB%91%E6%A0%91_20220316163142686602.png)
红黑树也存在大数据量情况下，层级较深，检索速度慢的问题。

### B-Tree

为了解决上述问题，可以使用 B-Tree 结构。
B-Tree (多路平衡查找树) 以一棵最大度数（max-degree，指一个节点的子节点个数）为5（5阶）的 b-tree 为例（每个节点最多存储4个key，5个指针）

![B-Tree结构](https://dhc.pythonanywhere.com/media/editor/B-Tree%E7%BB%93%E6%9E%84_20220316163813441163.png)

> B-Tree 的数据插入过程动画参照：https://www.bilibili.com/video/BV1Kr4y1i7ru?p=68
> 演示地址：https://www.cs.usfca.edu/~galles/visualization/BTree.html

### B+Tree

结构图：

![B+Tree结构图](https://dhc.pythonanywhere.com/media/editor/B+Tree%E7%BB%93%E6%9E%84%E5%9B%BE_20220316170700591277.png)

> 演示地址：https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html

与 B-Tree 的区别：

- 所有的数据都会出现在叶子节点
- 叶子节点形成一个单向链表

MySQL 索引数据结构对经典的 B+Tree 进行了优化。在原 B+Tree 的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的 B+Tree，提高区间访问的性能。

![MySQL B+Tree 结构图](https://dhc.pythonanywhere.com/media/editor/%E7%BB%93%E6%9E%84%E5%9B%BE_20220316171730865611.png)

### Hash

哈希索引就是采用一定的hash算法，将键值换算成新的hash值，映射到对应的槽位上，然后存储在hash表中。
如果两个（或多个）键值，映射到一个相同的槽位上，他们就产生了hash冲突（也称为hash碰撞），可以通过链表来解决。

![Hash索引原理图](https://dhc.pythonanywhere.com/media/editor/Hash%E7%B4%A2%E5%BC%95%E5%8E%9F%E7%90%86%E5%9B%BE_20220317143226150679.png)

特点：

- Hash索引只能用于对等比较（=、in），不支持范围查询（betwwn、>、<、…）
- 无法利用索引完成排序操作
- 查询效率高，通常只需要一次检索就可以了，效率通常要高于 B+Tree 索引

存储引擎支持：

- Memory
- InnoDB: 具有自适应hash功能，hash索引是存储引擎根据 B+Tree 索引在指定条件下自动构建的

**索引**

定义：索引是帮助MySQL高效获取数据的数据结构



## **索引的分类**

### **主键索引（primary key）**

唯一的标识，主键不可重复，只能有一个列作为主键

### **唯一索引（UNIQUE key）**

避免重复的列出现

### **常规索引（key/index）**

默认，index，key关键字设置

### **全文索引（FullText）**

快速定位位置



## Innodb中索引存储

![image-20221120220849425](C:\Users\ALIENWARE\AppData\Roaming\Typora\typora-user-images\image-20221120220849425.png)

回表查询：根据二级索引，查询出主键索引，再从主键索引中查询数据

| 分类                        | 含义                                                     | 特点               |
| --------------------------- | -------------------------------------------------------- | ------------------ |
| 聚集索引（Clustered Index） | 将数据存储与索引放到一起，索引结构的叶子节点保存行数据   | 必须有，且只有一个 |
| 二级索引（Secondary Index） | 将数据与索引分开存储，索引结构的叶子节点关联的是对应主键 | 可以存在多个       |

- 如果存在主键，主键索引就是聚集索引
- 如果不存在主键，将使用第一个唯一索引作为聚集索引
- 如果没有主键，则InnoDB将自动生成一个rowId作为隐藏的聚集索引

![image-20221120222628491](C:\Users\ALIENWARE\AppData\Roaming\Typora\typora-user-images\image-20221120222628491.png)

## **索引操作**

```mysql
-- 显示所有的索引信息 
SHOW INDEX FROM student 
-- 增加索引 
ALTER TABLE student ADD FULLTEXT INDEX studentname(studentname) 
CREATE [UNIQUE|FULLTEXT] INDEX id_app_user_name ON app_user(`name`,...) -- 一个索引可以关联多个字段 
-- 删除索引 
drop index index_name on table_name ;  
alter table table_name drop index index_name ;  
alter table table_name drop primary key ;  
-- 分析sql执行状况 
EXPLAIN SELECT * FROM student 
EXPLAIN SELECT * FROM student WHERE MATCH(studentName) AGAINST('')
```

**索引原则**

- 索引不是越多越好
- 数据少尽量不要索引

**数据结构**

- hash类型
- Btree



## SQL性能分析

- 查看执行频次

查看数据库insert、update、delete、select的访问频次

```mysql
show [global|session] status like 'Com_______'
```



- 慢查询日志

慢查询日志记录了所有执行时间超过指定参数的所有SQL语句的日志

```mysql
-- 查看慢查询日志是否开启
show variables like 'slow_query_log'

-- 开启慢查询日志
set global slow_query_log='ON'
```

配置文件修改（/etc/my.conf）

```
# 开启慢查询日志
slow_query_log=1

# 设置慢查询日志的时间（/s）
long_query_time=2
```

linux服务器慢查询日志文件：/var/lib/mysql/localhost-slow.log



- profile详情

```mysql
-- 查看数据库是否支持profile
select @@have_profiling

-- 查看profiling是否开启
select @@profiling

-- 开启profiling
set profiling=1

-- 查看profiles
show profiles

-- 查看指定profile
show profile for query query_id
```



- explain执行计划

  - id

    select查询的序列号，表示查询中执行的select子句或操作表的顺序（id相同，执行顺序从上到下；id不同，值越大，越先执行）

  - select_type

    表示select的类型，常见的有simple、primary、union、subquery等

  - type

    表示连接类型，性能由好到差依次为null、system、const、eq_ref、ref（非唯一性索引）、range、index、all（表示全表扫描）

  - possible_key

    显示可能应用的索引，一个或多个

  - key

    实际使用的索引

  - key_len

    索引长度

  - rows

    Mysql认为要执行查询的行数

  - filtered

    表示返回结果的行数占需读取行数的百分比

  

## 索引使用规则



### 最左前缀法则

如果查询关联多列，要遵循最左前缀法则，查询从索引的最左列开始，并且不跳过索引中的列。如果跳过某一列，将导致索引部分失效。



### 范围查询

联合索引中，出现范围查询（<，>），范围查询右侧的索引失效



### 索引列运算

在索引列中进行运算，将导致索引失效



### 字符串不加引号

字符串类型字段使用时，如果不加引号，索引将失效

```mysql
select * from `user` where name = 张三
```



### 模糊查询

如果是尾部模糊匹配，索引不会失效，但如果是头部模糊匹配，索引将失效

```mysql
-- 索引生效
explain select * from `user` where profession like '软件%'

-- 索引失效
explain select * from `user` where profession like '%工程'
```



### 数据分布影响

如果Mysql评估使用索引比全表扫描更慢，则不使用索引



### SQL提示

SQL提示，是优化数据库的重要手段，是通过在SQL语句中加入一些人为的提示来达到优化操作的目的。

```mysql
-- 使用索引
explain select * from `user` use index(idx_user_pro) where profession = '软件工程'

-- 忽略索引
explain select * from `user` ignore index(idx_user_pro) where profession = '软件工程'

-- 强制索引
explain select * from `user` force index(idx_user_pro) where profession = '软件工程'
```



### 前缀索引

当字段类型为字符串（varchar、text）时，需要建立索引的字符串长度过大，查询时，会浪费大量磁盘IO，影响效率。此时可以只将字符串的一部分前缀建立索引，从而节约索引空间，提高索引效率

语法：

```mysql
-- 提取字符串前n个字符作为索引
create index idx_tableName on table_name(column(n))
```

前缀长度选择

```mysql
select count(distinct email)/count(*) from tb_user

select count(distinct substring(email,1,5))/count(*) from tb_user
```



### 覆盖索引与回表查询

覆盖索引：返回的数据被包含在索引中，不需要回表查询

回表查询：返回的数据需要在二级索引的基础上，再通过主键进行聚集索引



# SQL优化

## 插入数据

insert优化

- 批量插入

- 手动提交事务

  ```mysql
  start transaction;
  insert into ...
  insert into...
  commit;
  ```

- 大批量插入数据

  ```mysql
  mysql --local-infile -u root -p
  
  -- 开启本地加载文件导入数据的开关
  select @@local_infile
  set global local_infile = 1
  
  -- 执行命令，加载数据到表中
  load data local infile '/root/sql.log' into table `user` fields terminated by ',' lines terminated by '\n'
  ```

  



# 视图、存储过程、触发器

## 视图

视图是一种虚拟存在的表，视图中的数据不在数据库表中实际存在，行和列来自定义视图的查询中使用的表，并且是再使用视图时动态生成的。

视图只保存了查询的SQL逻辑，不保存查询结果

```mysql
-- 视图的创建
create [or replace] view view_name[(列名列表)] as select... [with|cascaded|local|check option] 

-- 查询
select * from view_name

-- 修改
create or replace view ...
alter view view_name as select... [with|cascaded|local|check option]

-- 删除
drop view [if exists] view_name
```



- 视图的检查选项

当使用with check option子句创建视图时，mysql会通过视图检查正在更改的每个行，使其符合视图的定义，mysql允许基于另一个视图创建视图，它还会检查依赖视图中的规则以保持一致性，mysql提供两个选项：cascaded和local，默认为cascaded





## 变量

### 变量作用

在存储过程和函数中，可以是用变量来保存查询或计算的中间结果，或者输出查询的数据



### 变量的分类

#### 系统变量

系统变量由系统定义，属于服务器层面，启动MySQL服务，生成MySQL服务实例期间，MySQL将为MySQL服务器内存中的系统变量赋值，这些变量定义了当前MySQL服务实例的属性特征

系统变量是Mysql服务器提供的变量，分为全局变量（Global）、会话变量（Session）

```mysql
-- 查看系统变量
show [session|global] variables
show [session|global] variables like ''
select @@[session|global].系统变量名

-- 设置系统变量
set [session|global] 系统变量名=值
set @@[session|global].系统变量名=值
```



#### 用户变量

用户变量是用户根据自己需求定义的变量

根据作用范围，可以分为会话用户变量和局部变量

- 会话用户变量：作用域和会话变量一样，对当前连接会话有效，会话用户变量的格式为：'@变量名'
- 局部变量：只在begin和end语句块中有效



**会话用户变量**

```mysql
-- 赋值
set @var_name = expr
set @var_name := expr 

select @var_name := expr
select 字段名 into @var_name from table_name

-- 使用
select @var_name
```



**局部变量**

局部变量是根据需要定义、在局部范围内生效的变量，访问之前，需要declare声明。可用作存储过程内的局部变量和输入参数，局部变量的作用范围是在begin...end块内

declare的方式声明的局部变量必须声明在begin中的首行

```mysql
-- 声明
declare 变量名 变量类型 [default ...]

-- 赋值
set 变量名 = 值
set 变量名 := 值
select 字段名 into 变量名 from table_name
```



## 存储过程

> 基本概念

概念：存储过程是经过**预先编译**并存储在数据库中的一段SQL语句的集合，调用存储过程可以简化应用开发人员工作，减少数据库和应用服务器之间的传输，提高数据处理效率

思想：数据库SQL语言层面的代码封装与重用



> 优势

- 高效（存储过程编译一次过后，就会保存到数据库，每次调用时都能直接执行）

- 对SQL语句进行封装、复用性高（一个存储过程往往对应一个具体的功能，每次需要完成此功能时，就可以重复调用该存储过程）
- 减少网络交互，提升效率（远程调用时，不需要传输大量sql语句，只需要传输存储过程名和参数）
- 安全性高（存储过程封装了具体SQL操作，并且存储过程的操作权限可以分配给特定用户，更加安全）
- 可维护性高（当功能发生变化时，修改存储过程更方便）



> 创建和调用语句

创建并调用存储过程

```mysql
-- 创建
create procedure procedure_name([参数列表]) -- 参数格式为： IN|OUT|INOUT 参数名 参数值类型
begin
	-- sql语句
end;

-- 调用
call procedure_name([参数列表]);
```

参数类型

| 类型  | 含义                                   | 备注           |
| ----- | -------------------------------------- | -------------- |
| in    | 参数作为输入                           | 参数的默认模式 |
| out   | 参数作为输出                           |                |
| inout | 既可以作为输入参数，又可以作为输出参数 |                |



创建存储过程实例：查询员工表中的最低工资记录

```mysql
-- 创建过程
delimiter //
create procedure show_min_salary(OUT ms double) 
begin
	select min(salary) into ms from emp;
end //
delimiter ;

-- 调用过程
call show_min_salary(@ms);

-- 查看变量
select @ms;
```

<u>tips: 在命令行中，执行创建存储过程的SQL时，需要通过关键字delimiter指定SQL语句的结束符</u>



存储过程语句查看与删除

```mysql
-- 查看
show create procedure procedure_name

-- 删除
drop procedure if exists procedure_name
```



- case

```mysql
-- 语法一
case case_value
	when when_value1 then statement1
	when when_value2 then statement2
end case;

-- 语法二
case when condition1 then statement1
	when condition2 then statement2
end case;
```



- while

```mysql
while condition1 do 
	-- SQL语句
end while;

-- 例：从1到n累加和
create procedure cal_sum(in num int, out res int)
begin
	declare sum int default 0;
	declare i int default 0;
	while i <= num do
		set sum := sum + i;
		set i := i + 1;
	end while;
	set res = sum;
end
```



- repeat

repeat...until:直到...退出循环

```mysql
repeat 
	-- SQL语句
until condition1
end repeat;
```



- loop

  实现简单循环，如果不加退出循环的条件，则loop为死循环

  - leave：退出循环
  - iterate：作用在循环中，跳过当前循环，进入下一次循环

```mysql
[begin_label:] LOOP
	-- SQL逻辑
end loop [end_label];

-- 例
create procedure procedure_loop(in num int, out res int)
begin
	declare i int default 0;
	declare sum int default 0;
	sum_loop: loop
		if i > num then 
			leave sum_loop;
		end if;
		set sum := sum + i;
		set i := i + 1;
	end loop sum_loop;
	set res = sum;
end
```

​	

- cursor（游标）

游标是用来存储查询结果集的数据类型，在存储过程和函数中可以使用游标对结果集进行循环处理。游标的使用：游标声明、open、fetch、close

```mysql
-- 声明游标
declare cursor_name cursor for select...

-- 打开游标
open cursor_name

-- 获取游标记录
fetch cursor_name into variable1[,variable2,...]

-- 关闭游标
close cursor_name
```



- handler（条件处理程序）

条件处理程序，可以用来定义在流程控制结构执行过程中遇到问题时相应的处理步骤

```mysql
declare handler_action handler condition_value [,condition_value]... statement;

handler_action
	continue: 继续执行当前程序
	exit: 终止执行当前程序
condition_value
	sqlstate sqlstate_value: 状态码，如'02000'
	sqlwarning: 所有以01开头的sqlstate代码的简写
	not found: 所有以02开头的sqlstate代码的简写
	sqlexception: 所有没被sqlwarning或not found捕获的sqlstate代码的简写
```



```mysql
CREATE DEFINER=`root`@`localhost` PROCEDURE `procedure_cursor`()
begin
	declare id varchar(4);
	declare name varchar(15);
	declare customers_cursor cursor for select cid, cname from customers;
	declare exit handler for sqlstate '02000' close customers_cursor;
	drop table if exists customers_name;
	create table customers_name (
		cid varchar(4),
		cname varchar(15)
	);
	open customers_cursor;
	while true do 
		fetch customers_cursor into id, name;
		insert into customers_name values(id, name);
	end while;
	close customers_cursor;
end
```



## 存储函数

存储函数是有返回值的存储过程，存储函数的参数只能是in类型的

```mysql
create function function_name([参数列表]) 
returns type [characteristic]
begin 
	-- sql语句
	return ...;
end;

characteristic:
	deterministic: 相同输入参数总是产生相同结果
	no sql: 不包含sql语句
	reads sql data: 包含读取数据的语句，但不包含写入数据的语句
```



## 触发器

> 基本概念

触发器是一种特殊的存储过程，它在特定的数据库表的特定行为上触发，具体的行为是在insert/update/delete操作前后，触发并执行触发器中定义的sql语句集合。

使用触发器可确保数据的完整性，完成日志记录、数据校验等操作。



在MySQL数据库中，触发器只支持行级触发，不支持语句级触发。



> 触发器类型

| 触发器类型 | new和old                                                 |
| ---------- | -------------------------------------------------------- |
| insert类型 | new表示新增数据                                          |
| update类型 | old表示修改之前的数据，new表示将要修改或已经修改后的数据 |
| delete类型 | old表示将要删除或已经删除的数据                          |



> 创建、查看、删除语句

```mysql
-- 创建
create trigger trigger_name
before[after] insert[update][delete]
on table_name for each row -- 行级触发器
begin
	trigger_stmt;
end;

-- 查看
show triggers;

select * from information_schema.triggers where trigger_name = '';

-- 删除
drop trigger [schema_name.]trigger_name; -- 没有指定schema_name，默认为当前数据库
```





# 锁

- 介绍

锁是计算机协调多个进程或线程并发访问某一资源的机制

- 分类
  - 全局锁：锁定数据库中所有表
  - 表级锁：每次操作锁住整张表
  - 行级锁：每次操作锁住对应行数据



## 全局锁

对整个数据库





# E-R模型

## E-R模型

E-R模型（Entity-Relationship model）

实体是一种物体或概念，包含两种属性：

1. 存在

2. 可区分



实体的分类：

强实体：可以单独存在

弱实体：存在依赖于强实体或其他实体



## 数据库设计

> 数据库设计理念：
>
> 将真实世界的信息映射到抽象的概念和关系上，便于操作

E-R模型是实现上述理念的一种方法



### 阶段

1. 概念设计
2. 逻辑设计
3. 物理设计





# 权限管理

一、 创建用户

```mysql
-- 语法
create user 用户名 identified by '密码'

-- 例子
create user zhangsan identified by '123456'
```

新创建的用户，默认没有任何权限



二、 分配权限

```mysql
-- 语法
grant 权限 on 数据库.数据库表 to '用户'@'主机名'

-- 所有权限
grant all on *.* to 'zhangsan'@'%'

-- 特定权限
grant select,insert,update,delete,create,drop on *.* to 'jack'@ '%'
```



三、 收回权限

```mysql
-- 语法
revoke 权限 on 数据库.数据库表 from '用户'@'主机名'

-- 例子
revoke all on *.* from 'zhangsan'@'%'
```



