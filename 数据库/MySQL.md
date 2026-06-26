# MySQL基础

## 一、基础

### 1.1 连接

```mysql
# 启动和关闭mysql服务
net stop mysql;
net start mysql;

# 连接
mysql -h -127.0.01 -P 3306 -u root -p
mysql -u root -p
```

### 1.2 数据库

```mysql
# 创建使用utf8字符集，并带校对规则（utf8_bin 不区分大小写,默认utf8_general_ci不区分大小写）的数据库
create database if no exists da_name CHARACTER SET utf8 COLLATE utf8_bin
# 删除数据库
drop database da_name;
# 查看所有数据库
show databases;
# 查看数据库的信息
show create database da_name;
```

### 1.3 表

#### 1.3.1 创建表

```mysql
# 创建表
create table if not exists t_name(
    field1 datatype [column_constraints],
    field2 datatype [column_constraints],
    field3 datatype [column_constraints],
    ...
    [table_constraints]
)[table_options];
```

datatype (字段类型)

```mysql
整数：tinyint smallint mediumint int bigint
小数: float double decimal[M,D]
文本: char varchar text longtext
二进制: blob longblob
日期: time datetime timestamp year
```

column_constraints (列约束)

```mysql
unsigned
NOT NULL 
NULL
default default_value 
auto_increment
unique
primary key
check(expression)
comment 'string'
```

table_constraints (表级约束)

```mysql
primary key (column1,column2..)
unique key index_name (column1,column2..)
index index_name (column1,column2..)
key index_name (column1,column2..)
```

table_options (表选项)

```mysql
ENGINE = InnoDB/MyISAM/MEMORY/CSV
DEFAULT CHARSET = utf8mb4
collate = utf8mb4_unicode_ci
auto_increment = 1000  				#自增起始值
comment = "表注释信息"
```

#### 1.3.2 修改表

```mysql
# 查看表结构
DESC tb_name;
# 添加列
alter table tb_name add field_name datatype [column_constraints];
# 修改列
alter table tb_name modify field_name datatype [column_constraints];
# 删除列
alter table tb_name drop field_name;
# 修改表名
rename table tb_name to tb_name2;
# 修改表字符集
alter table tb_name character set utf8;
```

### 1.4 CRUD

```mysql
# 增
insert into tb_name (cloumn1,cloumn2..) values(cloumn1,cloumn2..)
# 删
delete from tb_name where [condition]
# 改
update tb_name set cloumn_name = ? ;
# 查
select * from tb_name;
```

### 1.5 运算符

```mysql
between...and...
in(..)
like '%?%'
not like ''
is null
and
or
not
```

### 1.6 关键字

```mysql
where
distinct
order by
having
```

### 1.7 函数

```mysql
# 统计数学
count()
sum()
max()
min()
abs()
floor()
format(number,decimal_places)
# 字符串
substring()
ucase() 
lcase()
length()
replace(str,search_str,replace_str)
# 时间
current_date()					#当前日期
current_time()					#当前时间
date(dateTime)					#返回datetime日期部分
date_add()					    #date + 时间
date_sub()					    #date - 时间
datediff(date1,date2)			 #时间差
now()
```

## 二、事务

### 2.1 隔离级别

```mysql
read uncommitted     脏读、不可重复读、幻读
read committed	     不可重复读、幻读
repeatable read		 幻读
serializable	

如果不考虑隔离，会出现以下问题：
# 脏读(dirty read):
当一个事务读取另一个事务尚未提交的改变(update,insert,delete)时,产生脏读。即本事务开启后，因为别的事务的未提交操作，而影响到本事务
# 不可重复读(nonrepeatable read):
同一查询在同一事务中多次进行,由于其他提交事务所做的修改或删除,每次返回不同的结果集,此时发生不可重复读。即本事务开启后，因为别的事务的修改或删除操作并提交，而影响到本事务
# 幻读(phantom read):
同一查询在同一事务中多次进行,由于其他提交事务所做的插入操作,每次返回不同的结果集,此时发生幻读。即本事务开启后，因为别的事务的插入操作并提交，而影响到本事务
#加锁
当有事务在操作某个表时，该表将不再被其他表操作，直到正在操作的事务
```

### 2.2 基础语法

```mysql
# 查询当前事务隔离级别
SELECT @@SESSION.TX_ISOLATION;-- mysql5.7 
SELECT @@TRANSACTION_ISOLATION;-- mysql8.0

# 设置隔离级别 -- 只在本次会话有效
set transaction isolation level read committed;-- 需小写
# 设置全局的事务隔离级别,该设置不会影响当前已经连接的会话,新会话,将使用新设置的事务隔离级别
set global transaction isolation level read committed;

# 开启事务
START TRANSACTION;

# 设置保存点
SAVEPOINT a;

# 执行dml操作

# 回退操作
ROLLBACK TO a; -- 回退到 a
ROLLBACK; -- 回退到事务开始的状态

# 提交事务
COMMIT;
```

### 2.3 四大特性

- A（原子性）：事务中的操作要么全部完成，要么全部失败

  ```
  当事务开始时，MySQL 会在undo log中记录事务开始前的旧值。如果事务执行失败，MySQL 会使用undo log中的旧值来回滚事务开始前的状态；如果事务执行成功，MySQL 会在某个时间节点将undo log删除。
  ```

- C（一致性）：确保事务从一致性状态到另外一个一致性状态，比如银行转账事务，A给B转账了，应该保持总金额 A + B 是不变的

- I（隔离性）：并发执行的任务是互不干扰的，设置事务隔离状态，默认为可重复读

  ```
  在 MVCC 中，每行记录都有一个版本号，当事务尝试读取记录时，会根据事务的隔离级别和记录的版本号来决定是否可以读取
  ```

- D（持久性）：事务一旦提交了，就一定保存到数据库中，即使数据库崩溃了，修改的数据也不会丢失

  ```
  redo log 是一种物理日志，当执行写操作时，MySQL 会先将更改记录到 redo log 中。当 redo log 填满时，MySQL 再将这些更改写入数据文件中。
  如果 MySQL 在写入数据文件时发生崩溃，可以通过 redo log 来恢复数据文件，从而确保持久性
  ```



