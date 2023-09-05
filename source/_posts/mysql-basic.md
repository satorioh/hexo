---
title: MySQL 基础操作
date: 2023-09-05 15:36:59
categories: MySQL
tags:
- mysql
permalink: mysql-basic
---
### 一、库的操作
#### 1.展示服务器连接信息
```mysql
show status;
```

#### 2.显示所有的库
```mysql
show databases;
```
<!--more-->

#### 3.创建库
```mysql
create database db_school charset = utf8;
```

#### 4.切库
```mysql
use db_school;
```

#### 5.删库
```mysql
drop database vsearchlogDB;
```

### 二、表的操作
#### 1.查看所有的表
```mysql
show tables;
```

#### 2.建表
```mysql
create table class
(
    id    int primary key auto_increment,
    name  varchar(30) not null,
    sex   enum ('w','m','o'),
    age   tinyint unsigned,
    score float default 0
);
create table hobby
(
    id     int primary key auto_increment,
    name   varchar(30) not null,
    hobby  set ('sing','dance','draw','swim'),
    level  char(5) comment '评级',
    price  decimal(7, 2) check ( price > 8000 ),
    remark text comment '备注部分'
);
```

#### 3.查看表结构
```mysql
desc class;
```

#### 4.查看表的创建信息
```mysql
show create table class;
```

#### 5.删表
```mysql
drop table class;
```

#### 6.修改表名
```mysql
alter table class rename xxx;
```

#### 7.复制表结构，但不复制数据
```mysql
create table archive_class like class;
```

#### 8.复制表中某一条记录，到新的表
```mysql
insert into archive_class select * from class where id = 1;
```

### 三、记录的操作
#### 1.插入记录
```mysql
# 插入全部字段
insert into class
values (1, 'Lily', 'w', 12, 90),
       (2, 'Lucy', 'w', 18, 76),
       (3, 'Tom', 'm', 17, 83);

# 插入部分字段
insert into class (name, age, score)
values ('billy', 20, 100),
       ('jack', 22, 65);
       
# 查询并插入
insert into archive_class select * from class where id = 1;
```

#### 2.查询记录
```mysql
# 获取当前所在的数据库名称
select database();

# 查询记录
select * from class;

# 查询指定字段
select name, age, score from class;

# where条件
select * from class where sex in ('w');

# null检测
select * from class where sex is null;
```

#### 3.修改记录
```mysql
update class set age=18,score=91 where name="Abby"; 

update class set sex='m' where sex is null; 

# 不加where则全部修改
update class set age=age+1;
```

#### 4.删除记录
注意:delete语句后如果不加where条件,所有记录全部清空
```mysql
delete from class where id=4;
```

### 四、字段（表头）的操作
#### 1.新增字段
```mysql
alter table hobby add phone char(10) # 默认在最后加一列
alter table hobby add phone char(10) first # 在最前面加一列
alter table hobby add phone char(10) after price; # 在指定列后加一列
```

#### 2.修改字段
```mysql
# 修改字段类型和约束
alter table hobby modify phone char(15) not null default "";
```

#### 3.删除字段
```mysql
# 数据也一并删除
alter table hobby drop level;
```

#### 4.替换字段
```mysql
alter table hobby change phone tel char(15) not null default "";
```

### 五、更新和删除的操作原则
- 除非确实打算更新和删除每一行，否则绝对不要使用不带WHERE子句的UPDATE或DELETE语句。
- 保证每个表都有主键，尽可能像WHERE子句那样使用它（可以指定各主键、多个值或值的范围）。
- 在对UPDATE或DELETE语句使用WHERE子句前，应该先用SELECT进行测试，保证它过滤的是正确的记录，以防编写的WHERE子句不正确。
- 使用强制实施引用完整性的数据库，这样MySQL将不允许删除具有与其他表相关联的数据的行。


