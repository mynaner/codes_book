# SQL 入门

常见关系数据库

* SQLserver
* Oracle
* mySQL
* DB2
* SQLite
* ... 


## SQL语言

* Structured Query Language，结构化查询语言
* 非过程性语言
* 为了加强sql的语言能力，各个厂商增强了过程性语言的特征
    * 如oracle的PL/SQL 过程性处理能力
    * SQL Server，Sybase的T-SQL
* SQL是用来存取关系数据库的语言，具有查询，操纵，定义和控制关系型数据库的四方面功能

## SQL 分类

* DDL (数据定义语言) Data Definition Language
    * 用来定义数据库的对象，如数据表，视图，索引等
* DML (数据操作语言-数据处理语言) Data Manipulation Language
    * 在数据库表中更新，增加和删除记录
    * 如update，insert，dalete --增删改
* DCL (数据控制语句) Data Control Language
    * 指用于设置用户权限和控制事务语句
    * 如grant，revoke，if .. else，while，begin transaction
* DQL (数据查询语句) Data Query Language
    * select



### 基本语法

```cmd
进入数据库
mysql -uroot -p000000 //  mysql -u用户名 -p密码
```

数据库操作
```sql
-- 创建语法
-- create database 数据库名字;
create database web;
-- create database 数据库名字 CHARACTER SET 字符编码;
CREATE DATABASE web_01 CHARACTER SET UTF8;
-- CREATE DATABASE 数据库名字 CHARACTER SET 字符编码 COLLATE 字符比较方式;
CREATE DATABASE web_02 CHARACTER SET UTF8 COLLATE utf8_bin;

-- 查看，删除数据库
-- 查看当前服务器所有数据库
SHOW DATABASES;
-- 显示数据库创建语句
-- SHOW CREATE DATABASE 数据库名;
SHOW CREATE DATABASE web_02;
-- 删除数据库语句
-- DROP DATABASE 要删除的数据库名;
DROP DATABASE web_01; 
-- 修改数据库
-- 切换数据库
-- use 数据库名
use web;
-- 查看当前的数据库
select database(); 


ALTER DATABASE `web`
DEFAULT CHARACTER SET utf8
DEFAULT COLLATE utf8_general_ci;

```



表操作
```sql
-- 创建表 
-- 列名 列名 数据类型

create TABLE user(
    `id` int,
    `name` varchar(100),
    `password` varchar(100),
    `birthday` date
);

create TABLE users(
    id int,
    name varchar(100),
    password varchar(100),
    birthday date
);
-- 并设置字符集校验规则
-- create TABLE user(
--     `id` int,
--     `name` varchar(100),
--     `password` varchar(100),
--     `birthday` date
-- )character set 字符集 collate 校对规则 ;

```

常用数据类型

* 字符串型 : 
    * VARCHAR   0-65535 字节  变长字符串 必须指定大小 VARCHAR(大小)
    * CHAR  0-255 字节  定长字符串 
* 大数据型 ：
    * BLOB 0-65535字节	二进制形式的长文本数据
    * TEXT 0-65535字节	长文本数据
* 数值型：
    * TINYINT   1 字节		小整数值
    * SMALLINT  2 字节		大整数值
    * INT,      4 字节  	大整数值
    * BIGINT    8 字节		极大整数值
    * FLOAT,    4 字节	    单精度 浮点数值
    * DOUBLE    8 字节      双精度浮点数值
* 逻辑型：BIT 数据类型保存位字段值，并且支持MyISAM、MEMORY、InnoDB和BDB表。
* 日期型：
    * DATE  YYYY-MM-DD	日期值
    * TIME  HH:MM:SS	时间值或持续时间
    * DATETIME  YYYY-MM-DD HH:MM:SS	混合日期和时间值
    * TIMESTAMP YYYYMMDD HHMMSS	混合日期和时间值，时间戳


```sql
-- 删除指定表
-- drop table 表名;
drop table employees;
-- 查看当前数据库所有的表
show tables;
-- 查看指定数据库的所有表
-- show tables from 表名;
show tables from web;
-- 查看表结构
-- desc 表名字;
desc employe;
-- 实例
CREATE TABLE employe(
    id INT,
    name VARCHAR(100),
    gender VARCHAR(100),
    birthdy DATE,
    entry_date DATETIME,
    job CHAR,
    salary TINYINT,
    resume TEXT
);
```

### 单表字段约束

* 定义主键约束
    * primary key 不允许为空，不允许重复
        
```sql
-- 定义主键约束
-- primary key 不允许为空，不允许重复
CREATE TABLE employes(
    id INT PRIMARY KEY,
    ... 
);
-- 查看表结构
desc employes;
-- 删除主键
alter table employes drop primary key; 
-- 添加主键（当该表没有主键的时候，否则应该先删除主键）
-- alter table employes add primary key(作为主键的列名);
alter table employes add primary key(id);

-- 主键自动增长
-- AUTO_INCREMENT定义列为自增的属性，一般用于主键，数值会自动加1。
-- ALTER TABLE web.`user` MODIFY COLUMN id int(11) AUTO_INCREMENT; //修改为自增
-- 创建时候
CREATE TABLE employees(
    id INT PRIMARY KEY AUTO_INCREMENT, 
    ...
); 
-- 唯一约束
CREATE TABLE employe(
    id INT PRIMARY KEY AUTO_INCREMENT, 
    name VARCHAR(100) unique
);
-- 非空约束
CREATE TABLE employes(
    id INT PRIMARY KEY AUTO_INCREMENT, 
    name VARCHAR(100) unique null,
    gender VARCHAR(100) not null
);
```

### 修改表结构
```sql
-- 修改表名
-- rename table 原表名 to 修改后表名;
rename table employes to user;
-- 修改表的字符集
alter table employes character set utf8;
-- ALTER TABLE 语法  追加，修改或删除列
-- ALTER TABLE user add aa VARCHAR(100),add aaa VARCHAR(100);

-- tbl_name 表名字
ALTER TABLE tbl_name  
    alter_specification [, alter_specification] ...
-- 可以接多个 alter_specification 并用逗号隔开
--例 ：
    ALTER TABLE user add aa VARCHAR(100),add aaa VARCHAR(100);   
alter_specification:    
    -- add 追加
    ADD  column_definition
        -- 例
        ALTER TABLE user add ccc
    ADD  column_definition [FIRST | AFTER col_name ]
        -- FIRST 在第一个列的前面加入,  
        ALTER TABLE user add ccc int FIRST;
        -- AFTER col_name 在指定列后面追加
        ALTER TABLE user add ddd int after ccc;
 
  | ADD INDEX [index_name] [index_type] (index_col_name,...)
        -- 添加索引 INDEX  
        ALTER TABLE user add INDEX (ddd);  

  | MODIFY column_definition [FIRST | AFTER col_name]  
        -- 修改字段类型
        -- 重新定义该字段属性
        --  ALTER TABLE testalter_tbl MODIFY c CHAR(10);  
        alter table user MODIFY id int not null;
  | DROP col_name
        -- 删除字段 
        alter table user drop ddd;
  | DROP PRIMARY KEY
        -- 删除主键 
        -- alter table user drop primary key; 该操作，主键不能为自增 
  | DROP INDEX index_name
        -- 删除索引  
  | CHANGE  old_col_name column_definition
        [FIRST|AFTER col_name]
        -- 改变列名
        alter table user change id userid int;
```


```sql
-- results、server 、systemmysql有六处使用了字符集，分别为：client 、connection、database、。
-- client是客户端使用的字符集。 
-- connection是连接数据库的字符集设置类型，如果程序没有指明连接数据库使用的字符集类型就按照服务器端默认的字符集设置。       
-- database是数据库服务器中某个库使用的字符集设定，如果建库时没有指明，将使用服务器安装时指定的字符集设置。    
-- results是数据库给客户端返回时使用的字符集设定，如果没有指明，使用服务器默认的字符集。       
-- server是服务器安装时指定的默认字符集设定。       
-- system是数据库系统使用的字符集设定。（utf-8不可修改）

-- 通过修改my.ini 修改字符集编码
```


## 数据库CRUD语句
 
* INSERT 语句   （增加数据）
```sql 
-- 在values中列出的数据位置必须与被加入的列的排列位置相对应。
-- 字符和日期型数据应包含在单引号中

-- 查看mysql编码格式
show variables like 'character%';

-- 一，单条插入  INSERT INTO VALUE
INSERT INTO user (name,password,birthday) value ("张三","dadfsdfd","2018-9-10");
-- 二，单条或多条插入  INSERT INTO VALUES
INSERT INTO user (name,password,birthday) values ("张三","dadfsdfd","2018-9-10"),("李四","dadfsdfd","2018-9-10");
-- 三，INSERT INTO SET // 不能批量增加数据
INSERT INTO user set name="王五",password="dengxin",birthday="2018/12/18"; 
-- 四，复制两表之间 Insert into Table2(field1,field2,...) select value1,value2,... from Table1
-- Insert into Table2 select  *  from Table1
```

* UPDATE 语句   （更新数据）
```sql 
-- 修改列的全部
UPDATE user SET image='SDFSADFAG阿道夫撒地方';
-- WHERE子句指定应更新哪些行。如没有WHERE子句，则更新所有的行
UPDATE user SET image='SDFSADFAG阿道夫撒地方' WHERE id=3;;
-- 在原值上进行修改
UPDATE user SET id=id+100  WHERE id=6;
```
* DELETE 语句   （删除数据）
```sql
-- 如果不使用where子句，将删除表中所有数据。
-- Delete语句不能删除某一列的值（可使用update）
-- 使用delete语句仅删除记录，不删除表本身。如要删除表，使用drop table语句。
-- 同insert和update一样，从一个表中删除记录将引起其它表的参照完整性问题，在修改数据库数据时，头脑中应该始终不要忘记这个潜在的问题
-- 外键约束
-- 删除表中数据也可使用TRUNCATE TABLE 语句，它和delete有所不同，参看mysql文档。
-- 删除id等于3的数据
delete from user where id=3;
-- 如果不加where删除当前表的所有数据
delete from user;
-- 删除表中数据也可使用TRUNCATE TABLE 语句，它和delete有所不同
TRUNCATE [TABLE] tbl_name
```
* SELECT 语句   （查找数据）
```SQL
SELECT [DISTINCT] *|{column1, column2. column3..} FROM table;
-- select 指定查询哪些列的数据。
-- column指定列名。
-- *号代表查询所有列。
-- from指定查询哪张表。
-- DISTINCT可选，指显示结果时，是否剔除重复数据 
-- * 代表所有， 查询整个user表
SELECT * FROM user;
-- 查询指定列的数据
SELECT id,name from user;
-- 剔除重复数据
select DISTINCT name from user;

-- 查询通过计算出来的数据
SELECT *|{column1｜expression, column2｜expression，..} FROM table;

-- 更改查出来的表头名
SELECT column as 别名 from 表名;

select name as ddd from user;
select name as aaa ,id as ddd from user;

-- WHERE 子句

SELECT column_name,column_name FROM table_name WHERE column_name operator value;

-- 
select id from user where id=1;
select * from user where id>1;

-- =	等于
select * from user where id=1;
-- <>	不等于。注释：在 SQL 的一些版本中，该操作符可被写成 !=
select * from user where id<>1;
select * from user where id!=1;

-- >	大于
-- <	小于
-- >=	大于等于
-- <=	小于等于
select * from user where id>1;
select * from user where id>1 and id<5;
select * from user where id<1 or id>5;
select * from user where not id<5;
-- 空值判断
select * from user where image is null;
-- between and (在 之间的值)	
select * from user where id BETWEEN 1 and 5; 
-- IN	指定针对某个列的多个可能值
select * from user where id in(1,2,3,4,5); 
-- LIKE	搜索某种模式
--  % 表示多个字值，_ 下划线表示一个字符；
--  M% : 为能配符，正则表达式，表示的意思为模糊查询信息为 M 开头的。
--  %M% : 表示查询包含M的所有内容。
--  %M_ : 表示查询以M在倒数第二位的所有内容。
select * from user where id like "%M"

-- order by 子句查询排序结果
SELECT column1, column2. column3..
		FROM	 table;
		order by column asc|desc

-- 列排序
-- Order by 指定排序的列，排序的列即可是表中的列名，也可以是select 语句后指定的列名。
-- Asc 升序、Desc 降序
-- ORDER BY 子句应位于SELECT语句的结尾。

-- 升序
select * from user order by birthday; 和 select * from user order by birthday asc;
-- 降序
select * from user order by birthday desc;
-- 多列排序 (先以birthday排序，在birthday相同的情况下，在进行id排序)
select * from user order by birthday,id desc; 

```

```sql
-- 聚集函数
-- count 
Select count(*)|count(列名) from tablename
		[WHERE where_definition] 
-- 查看总共多少信息
select count(*) from user;
-- 查看id大于5的数据有多少条
select count(*) from user where id>5;
-- COUNT(column_name) 函数返回指定列的值的数目（NULL 不计入）：
select count(id) from user;


-- sum
-- Sum函数返回满足where条件的行的和
Select sum(列名)｛,sum(列名)…｝ from tablename
		[WHERE where_definition] 

-- 如果没有where条件则为所有
select sum(id) from user; 


-- AVG
-- AVG函数返回满足where条件的一列的平均值
Select sum(列名)｛,sum(列名)…｝ from tablename
		[WHERE where_definition]

-- MAX/MIN
-- Max/min函数返回满足where条件的一列的最大/最小值
Select max(列名)　from tablename
		[WHERE where_definition] 


-- 使用group by 子句对列进行分组 用于多表

SELECT column1, column2. column3.. FROM	table;
		group by column having ...
 
-- 使用having 子句 对分组结果进行过滤
-- Having和where均可实现过滤，但在having可以使用聚集函数,having通常跟在group by后，它作用于分组
SELECT site_id, SUM(access_log.count) AS nums
FROM access_log GROUP BY site_id;
```



备份恢复数据库

备份数据库表中的数据

Linux 下 
> mysqldump -uroot -p000000 web > ~/web.sql;

恢复数据库

> 



```sql
-- 多表设计

-- 定义外键约束
  foreign key
  foreign key(ordersid) references orders(id)


-- 多表设计三种关系 一对一，一对多，多对多

```

