#### MySQL相关：
1. 检查MySQL是否安装成功，安装成功可以正常输出
```
[root@host]# mysqladmin --version
```
2. 启动MySQL
```
service mysqld start
```
如果是MariaDB
```
systemctl start mariadb  #启动MariaDB
systemctl stop mariadb  #停止MariaDB
systemctl restart mariadb  #重启MariaDB
systemctl enable mariadb  #设置开机启动
```
3. 连接本地MySQL
```
mysql -u root -p
```
4. 连接远程MySQL
```
mysql -h host -u root -p
```
5. 首次安装MySQL后，默认root密码为空，通过以下命令设置密码
```
[root@host]# mysqladmin -u root password "new_password";
```
6. 查看MySQL版本
```
SELECT VERSION();
```
#### 数据库相关：
1. 创建数据库
```
CREATE DATABASE my_database;
```
2. 查看所有数据库
```
SHOW DATABASES;
```
3. 使用某个数据库
```
use my_database;
```
4. 查看当前数据库
```
SELECT DATABASE();
```

5. 查看当前用户
```
SELECT USER();
```
6. 选定数据库情况下在用户表中添加用户
```
INSERT INTO user 
  (host, user, password, 
   select_priv, insert_priv, update_priv) 
   VALUES ('localhost', 'guest', 
   PASSWORD('guest123'), 'Y', 'Y', 'Y');
```
然后刷新权限，就不用等MySQL重启才能生效了
```
FLUSH PRIVILEGES;
```
第二种添加用户的方法就是下面的GRANT语句方法

5. 设置字符集
```
set names utf8;
```

#### 表相关：(注意:所有的数据库名，表名，表字段都是区分大小写的。)
1. 创建表
```
CREATE TABLE pet (name VARCHAR(20), owner VARCHAR(20),
    -> species VARCHAR(20), sex CHAR(1), birth DATE, death DATE);
```
2. 查看所有表 
```
SHOW TABLES;
```
3. 查看表结构
```
DESCRIBE pet;
```
4. 删除表
```
DROP TABLE pet;
```
5. 查看表详细信息
```
SHOW TABLE STATUS LIKE 'table_name'\G;
```
#### 数据相关：(注意:所有的数据库名，表名，表字段都是区分大小写的。)
1. 插入数据
```
INSERT INTO table_name(field1, field2,...fieldN) 
    VALUES 
    (value1,value2,...valueN);
```
如果数据是字符型，必须使用单引号或双引号。

字段名可以省略。

对于AutoIncrement的字段可以不用设置值。

同时插入多条数据：
```
INSERT INTO table_name(field1, field2,...fieldN) 
    VALUES 
    (value11, value12,...value1N),
    (value21, value22,...value2N);
```


2. 查询数据
```
SELECT column_name, column_name2
FROM table_name
[WHERE Clause]
[LIMIT N] [OFFSET M];
```

3. WHERE Clause
WHERE表达式可以用于SELECT、UDDATE、DELETE语句。
```
WHERE [BINARY] condition1 [AND [OR]] condition2...
```
逻辑判断：=等于、<>或!=不等于、>、<、>=、<=、LIKE。

字段值如果是字符串需要使用单引号或者双引号括起来。

WHERE表达式在比较的时候是忽略大小写的，如果要区分大小写，需要加上BINARY关键字。

LIKE表示含有指定字符，通常搭配%通配符使用。正则表达式中的通配符是*。如果没有搭配%使用，则LIKE和=是一样的。注意通配符摆放的位置表示匹配任意多个字符。

示例：

```
SELECT * FROM runoob_tbl WHERE runoob_author LIKE '%com';
```


4. 改
```
UPDATE table_name SET field1 = new_value1, field2 = new_value2,... [WHERE Clause];
```
new_value可以是表达式，比如
```
UPDATE students SET age = age + 1;
UPDATE students SET student_name = REPLACE(student_name, '小明','大明');
```

REPLACE函数用来替换某个字符串中的特定子字符串。

5. 删
```
DELETE FROM table_name [WHERE Clause];
```
如果没有指定WHERE表达式，则会删除该表中所有记录。

6. UNION
```
SELECT expression1, expression2, ... expressionN
FROM table_name
[WHERE CLAUSE]
UNION [ALL]
SELECT expression1, expression2, ... expressionN
FROM table_name2
[WHERE CLAUSE]
[ORDER BY column_name [ASC [DESC]]];
```
UNION默认会删去重复记录，如果要显示所有记录，需要使用UNION ALL。

如果表2中select的column是表1没有的，则不会显示表2独有的字段。
7. 排序

```
Order By column1, column2 [ASC [DESC]]
```

默认是升序排列ASC


8. GROUP BY column_name

将数据表依据一个或多个字段进行分组，相同值为一组。可以select COUNT、SUM、AVG等函数表达式

使用COUNT(*)计算每个分组中元素个数。

```
SELECT name, COUNT(*) FROM   employee_tbl GROUP BY name; // 依据name来分组，列出这些分组的name和Count。筛选出的结果就有个COUNT(*)字段。
```

使用SUM(signin) as sign_count作为一个新字段
```
SELECT name, SUM(signin) as sign_count FROM   employee_tbl GROUP BY name; // 依据name来分组，列出这些分组的name和每个分组里所有signin的和。
```

使用WITH ROLLUP计算所有分组的总和，结果多出一个name为NULL，sign_count为所有组总和的记录，因为无法表示所以为NULL。
```
SELECT name, SUM(signin) as sign_count FROM   employee_tbl GROUP BY name WITH ROLLUP; // 依据name来分组，列出这些分组的name和每个分组里所有signin的和。
```


这个时候我们可以把name这个列修改成为coalesce(name, '总数')，这样如果name非空则显示name，如果name是空的则显示'总数'。
```
SELECT coalesce(name, '总数'), SUM(signin) as sign_count FROM employee_tbl GROUP BY name WITH ROLLUP;
```

coalesce(a,b,c)表示依先后顺序优先返回非null值，如果都为空才返回null。

9. JOIN

联合多表查询，以a表某个字段=b表某个字段为标准来查询。

- [INNER] JOIN: 列出a表某字段的值与b表某字段的值相等 的记录。
- LEFT JOIN: 列出a表某字段的所有值，如果b表某字段没有对应值的话就显示null。
- RIGHT JOIN:列出b表某字段的所有值，如果a表某字段没有对应值的话就显示null。

```
mysql> use RUNOOB;
Database changed
mysql> SELECT * FROM tcount_tbl;
+---------------+--------------+
| runoob_author | runoob_count |
+---------------+--------------+
| 菜鸟教程  | 10           |
| RUNOOB.COM    | 20           |
| Google        | 22           |
+---------------+--------------+
3 rows in set (0.01 sec)
 
mysql> SELECT * from runoob_tbl;
+-----------+---------------+---------------+-----------------+
| runoob_id | runoob_title  | runoob_author | submission_date |
+-----------+---------------+---------------+-----------------+
| 1         | 学习 PHP    | 菜鸟教程      | 2017-04-12      |    
| 2         | 学习 MySQL  | 菜鸟教程      | 2017-04-12      | 
| 3         | 学习 Java   | RUNOOB.COM    | 2015-05-01      |
| 4         | 学习 Python | RUNOOB.COM    | 2016-03-06      |
| 5         | 学习 C      | FK            | 2017-04-05      |
+-----------+---------------+---------------+-----------------+
```

首先是INNER JOIN取交集
```
SELECT a.runoob_id, a.runoob_author, b. runoob_count FROM runoob_tbl a INNER JOIN tcount_tbl b ON a.runoob_author = b.runoob_author;
```
列出A表格的runoob_id、runoob_author以及B表格的runoob_count字段，A表格是runoob_tbl，B表格是tcount_tbl，两个表以A表格的runoob_author字段与B表格的runoob_author字段取交集进行查询。

所以结果是列出A中有的，B中也有的：
```
+-------------+-----------------+----------------+
| a.runoob_id | a.runoob_author | b.runoob_count |
+-------------+-----------------+----------------+
| 1           | 菜鸟教程        | 10             |
| 2           | 菜鸟教程        | 10             |
| 3           | RUNOOB.COM      | 20             |
| 4           | RUNOOB.COM      | 20             |
+-------------+-----------------+----------------+
```

其实WHERE语句查询多个表的情况下也等价于这个取交集：
```
mysql> SELECT a.runoob_id, a.runoob_author, b.runoob_count FROM runoob_tbl a, tcount_tbl b WHERE a.runoob_author = b.runoob_author;
+-------------+-----------------+----------------+
| a.runoob_id | a.runoob_author | b.runoob_count |
+-------------+-----------------+----------------+
| 1           | 菜鸟教程    | 10             |
| 2           | 菜鸟教程    | 10             |
| 3           | RUNOOB.COM      | 20             |
| 4           | RUNOOB.COM      | 20             |
+-------------+-----------------+----------------+
```

接下来是LEFT JOIN，以A中某字段的所有记录为准，查找B中某字段的相等值，如果没有就为NULL：
```
mysql> SELECT a.runoob_id, a.runoob_author, b.runoob_count FROM runoob_tbl a LEFT JOIN tcount_tbl b ON a.runoob_author = b.runoob_author;
+-------------+-----------------+----------------+
| a.runoob_id | a.runoob_author | b.runoob_count |
+-------------+-----------------+----------------+
| 1           | 菜鸟教程    | 10             |
| 2           | 菜鸟教程    | 10             |
| 3           | RUNOOB.COM      | 20             |
| 4           | RUNOOB.COM      | 20             |
| 5           | FK              | NULL           |
+-------------+-----------------+----------------+
5 rows in set (0.01 sec)
```

然后是RIGHT JOIN，以B中某字段的所有记录为准，查找A中某字段的相等值，如果有多个相等值就都列出来，如果没有就为NULL：
```
mysql> SELECT a.runoob_id, a.runoob_author, b.runoob_count FROM runoob_tbl a RIGHT JOIN tcount_tbl b ON a.runoob_author = b.runoob_author;
+-------------+-----------------+----------------+
| a.runoob_id | a.runoob_author | b.runoob_count |
+-------------+-----------------+----------------+
| 1           | 菜鸟教程        | 10             |
| 2           | 菜鸟教程        | 10             |
| 3           | RUNOOB.COM      | 20             |
| 4           | RUNOOB.COM      | 20             |
| NULL        | NULL            | 22             |
+-------------+-----------------+----------------+
```

10. MySQL NULL值处理
MySQL中NULL与其他任何值包括NULL，相比较都是false，即WHERE NULL = NULL返回false。MySQL当中判断当前值是否为NULL使用IS NULL（当前值为NULL时返回true）、IS NOT NULL（当前值非NULL时返回true）、<=>（比较的两个值均为NULL时返回true）。
11. 正则表达式匹配
```
SELECT * FROM person_tbl WHERE name REGEXP '^st'; // 匹配以st开头的name

SELECT * FROM person_tbl WHERE name REGEXP 'ok$'; // 匹配以ok结尾的name

SELECT * FROM person_tbl WHERE name REGEXP 'nar'; // 匹配包含nar字符串的name

SELECT * FROM person_tbl WHERE name REGEXP '^[aeiou]|ok$'; // 匹配以元音字母开头或者以ok结尾的name
```
12. 事务Transaction
MySQL的事务是一系列insert/update/delete修改数据操作的组合，比如删除一名员工的基本资料、信箱、文章等等。事务必须具有原子性不可拆分。要么全部执行，要么全部不执行。

MySQL中默认一条SQL语句就是一个事务，执行会马上COMMIT，如果要让多条SQL语句组成一个事务，可以使用BEGIN或者START TRANSACTION来开启事务。或者执行命令SET AUTOCOMMIT = 0；这样命令就不会马上执行。

```
mysql> begin;  # 开始事务
Query OK, 0 rows affected (0.00 sec)
 
mysql> insert into runoob_transaction_test value(5);
Query OK, 1 rows affected (0.01 sec)
 
mysql> insert into runoob_transaction_test value(6);
Query OK, 1 rows affected (0.00 sec)
 
mysql> commit; # 提交事务
Query OK, 0 rows affected (0.01 sec)
 
mysql>  select * from runoob_transaction_test;
+------+
| id   |
+------+
| 5    |
| 6    |
+------+
2 rows in set (0.01 sec)
 
mysql> begin;    # 开始事务
Query OK, 0 rows affected (0.00 sec)
 
mysql>  insert into runoob_transaction_test values(7);
Query OK, 1 rows affected (0.00 sec)
 
mysql> rollback;   # 回滚
Query OK, 0 rows affected (0.00 sec)
```
13. ALTER 改变，更改
- 修改表名
```
ALTER TABLE table_name RENAME TO table_name2;
```
- 添加字段
```
ALTER TABLE table_name ADD new_field field_type [FIRST | [AFTER some_field]] [NOT NULL] [DEFAULT default_value];
```
first是添加在表格第一列，after某个列是添加在该列后一列。如果要修改某个已有列的位置，只能先删除再添加到指定位置。

- 删除字段
```
ALTER TABLE table_name DROP field_name;
```
- 修改字段类型（MODIFY）
```
ALTER TABLE table_name MODIFY field_name new_field_type;
```
- 修改字段名（CHANGE）
```
ALTER TABLE table_name CHANGE old_field_name new_field_name new_field_type;
```
- 修改字段默认值
```
ALTER TABLE table_name MODIFY field_name field_type [NOT NULL] [DEFAULT default_value];

// 或者
ALTER TABLE table_name ALTER field_name SET DEFAULT new_default_value;
```
- 删除字段默认值
```
ALTER TABLE table_name ALTER field_name DROP DEFAULT;
```
- 修改表引擎
```
ALTER TABLE table_name ENGINE = engine_name;
```
引擎名可以是InnoDB、MyISAM之类的。可以通过show engines查询。

14. 临时表

临时表是在当前连接可见的，连接断开后会自动删除。也可以手动drop table。
```
CREATE TEMPORARY TABLE table_name(
    field_name field_type,
);
```

15. 复制表

只复制表结构，不复制表数据
```
CREATE TABLE new_table SELECT * FROM original_table where 1=2;
// 或者
CREATE TABLE new_table LIKE original_table;
```
复制表结构和数据
```
CREATE TABLE new_table SELECT * FROM original_table;

// 或者先复制结构，再复制数据
CREATE TABLE new_table LIKE original_table;

INSERT INTO new_table SELECT * FROM original_table;

// 或者复制结构数据加定义新字段
CREATE TABLE new_table (
    new_field_name new_field_type
) AS (
    SELECT some_columns_or_* FROM original_table [WHERE CLAUSE]
)

```

16. COUNT
```
SELECT COUNT(column_name) FROM table_name; // 返回指定列的值的个数，不包括NULL值

SELECT COUNT(*) FROM table_name; // 返回表中所有记录的条数

SELECT COUNT(DISTINCT column_name) FROM table_name; // 返回指定列的不同值的个数
```
17. d 

#### 用户管理相关：
1. 新增用户：
```
insert into mysql.user(Host,User,Password) values("对应的host","自己设置的用户名",password("自己设置的密码"));

flush privileges;
```
host可以设置localhost表示本机，或者%表示不限，或者指定ip。

2. 修改用户密码：
使用某个数据库后，
```
update user set password=PASSWORD('新密码') where user='用户名，比如说root' and host='主机名';
```
PASSWORD()这个函数是MySQL提供的加密函数<br>
然后刷新权限：
```
FLUSH PRIVILEGES; 
```

2. 新建远程登录用户：
首先要登录进去root账号，然后
```
grant 权限 on 数据库名.表名 to 用户名@'远程ip地址'  identified by '登录密码';
```
权限可以是以下几种<br>
all PRIVILEGES或者
select,insert,update,delete,create,drop任意组合，使用英文逗号分隔

数据库名和表名可以用*来表示通配

ip处填写为%表示不限制ip

最后还要刷新一下权限：
```
flush PRIVILEGS;
```

删除用户

```
DROP USER 'username'@'host'; 
```

### 查询及修改字段类型
查询
```
SELECT column_name,column_comment,data_type FROM information_schema.columns WHERE table_name='表格名'; 
```

增加字段
```
alter table 表格名 add 字段名 字段类型 not Null;
```

修改字段类型
```
alter table 表名 modify column 字段名 类型;
```

修改字段名

```
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 新数据类型;
```
varchar需要设置位数不然执行报错？

#### 常见问题整理
1. mysql数据库的user表里，存在用户名为空的账户即匿名账户，导致登录的时候是虽然用的是root，但实际是匿名登录的，也就是''@'localhost'开头的空字符串。
```
ERROR 1044 (42000): Access denied for user ''@'localhost' to database 'mydb'。
```
解决方法：
```
mysqld --defaults-file="D:\Program Files\MySQL\MySQL Server 5.6\my-default.ini" --console --skip-grant-tables;

mysql -u root mysql;

update user set password=PASSWORD('password') where user='root';

FLUSH PRIVILEGES; 
quit 
```
2. 忘记root密码
解决方法：<br>
（1）终止mysql进程
```
killall -TERM mysqld
```
（2）跳过权限检测启动mysql
```
safe_mysqld --skip-grant-tables &
```
（3）空密码方式以root登录mysql
```
mysql -u root
```
（4）修改root密码
```
mysql> update mysql.user set password=PASSWORD('新密码') where User='root'; 
mysql> flush privileges; 
mysql> quit
```

