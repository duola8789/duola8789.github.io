---
title: MySQL-01 快速上手
top: false
date: 2018-05-29 16:11:00
updated: 2020-08-02 18:41:58
tags:
- MySQL
categories: 全栈开发
---

MySQL-01皮毛。

<!-- more -->

# 1 简介

MySQL是最流行的关系型数据库管理系统

> [关系型数据库](https://baike.baidu.com/item/%E5%85%B3%E7%B3%BB%E5%9E%8B%E6%95%B0%E6%8D%AE%E5%BA%93/8999831?fr=aladdin)，是指采用了关系模型来组织数据的数据库，其以行和列的形式存储数据，以便于用户理解，关系型数据库这一系列的行和列被称为表，一组表组成了数据库。用户通过查询来检索数据库中的数据，而查询是一个用于限定数据库中某些区域的执行代码。
>
> 关系模型可以简单理解为二维表格模型，而一个关系型数据库就是由二维表及其之间的关系组成的一个数据组织。

我们使用关系型数据库管理系统（RDBMS）来存储和管理的大数据量。

MySQL是一种关联数据库管理系统，关联数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。

# 2 安装

## 2.1 linux下的安装

详情见[这里](http://www.runoob.com/mysql/mysql-install.html)。

## 2.2 windows下的安装

[下载地址在这里](https://dev.mysql.com/downloads/mysql/)。我没有选择最新的8.0.21.0，而是选择了5.7.31版本进行安装，主要原因还是为了少踩一些坑

安装时安装类型选择`Developer Default`版本

![](http://image.oldzhou.cn/Fq8LCm_P8bh3hyeSkEMv1u9Scnui)

安装过程可以参考[这篇文章](https://blog.csdn.net/chenriyang0306/article/details/54587034/)。

> 安装过程中遇到了按钮不见的问题，用快捷键`alt+n`下一步，`alx+x`执行，`alt+f`完成

安装完成后可以通过MySQL自带的命令行工启动MySQL，也可以通过图形化工具（例如Navicat）来连接数据库。

> 如果选择加密方式的时候选择了推荐的第一种新的加密方式，在用navicat连接数据库的时候会遇到下面的问题
>
> ```
> Client does not support authentication protocol requested by server; consider upgrading MySQL client
> ```
>
> [解决方法](https://zhuanlan.zhihu.com/p/36087723)：
>
> ```
> alter user '用户名'@localhost IDENTIFIED WITH mysql_native_password by '你的密码';
> ```

# 3 命令行工具

打开MySQL命令行工具，输入密码后：

![image](http://image.oldzhou.cn/18-7-22/35201274.jpg)

## 3.1 创建数据库

### （1）显示数据库

```BASH
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| my                 |
| myfirstdatabase    |
| mysql              |
| performance_schema |
| sakila             |
| sys                |
| test               |
| world              |
+--------------------+
9 rows in set (0.00 sec)
```

### （2）创建数据库

```BASH
mysql> create database zhou;
Query OK, 1 row affected (0.00 sec)
```

### （3）进入数据库

```BASH
mysql> use xiaohutu
Database changed
```

## 3.2 创建表

### （1）显示表

```BASH
mysql> show tables;
Empty set (0.00 sec)
```

### （2）创建表

```BASH
mysql> create table table1(
    -> name varchar(20),
    -> gender char(1),
    -> birth date,
    -> birthaddr varchar(20));
Query OK, 0 rows affected (0.04 sec)
```

`varchar(M)`是一种比`char`更加灵活的数据类型，同样用于表示字符数据，但是`varchar`可以保存可变长度的字符串。

其中`M`代表该数据类型所允许保存的字符串的最大长度，只要长度小于该最大值的字符串都可以被保存在该数据类型中。因此，对于那些难以估计确切长度的数据对象来说，使用`varchar`数据类型更加明智。

### （3）显示表结构

```BASH
mysql> describe table1;
+-----------+-------------+------+-----+---------+-------+
| Field     | Type        | Null | Key | Default | Extra |
+-----------+-------------+------+-----+---------+-------+
| name      | varchar(20) | YES  |     | NULL    |       |
| gender    | char(1)     | YES  |     | NULL    |       |
| birth     | date        | YES  |     | NULL    |       |
| birthaddr | varchar(20) | YES  |     | NULL    |       |
+-----------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```

## 3.3 向表中添加数据

### （1）查看表中数据

```BASH
mysql> select * from table1;
Empty set (0.00 sec)

mysql> select * from table1 where name='Jay';
+------+--------+------------+-----------+
| name | gender | birth      | birthaddr |
+------+--------+------------+-----------+
| Jay  | m      | 1999-01-01 | china     |
+------+--------+------------+-----------+
1 row in set (0.00 sec)

mysql> select name from table1 where name='Jay';
+------+
| name |
+------+
| Jay  |
+------+
1 row in set (0.00 sec)
```

### （2）插入数据

```BASH
mysql> update table1 set name="Jay Chow" where name="Jay";
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

### （3）修改数据
```
mysql> update mytable set birth="1994-03-06" where birth="0194-03-06";
Query OK, 1 row affected (0.14 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

如果`where`查询到多个符合条件的数据，那么会进行批量更改。

# 4 [数据类型](http://www.runoob.com/mysql/mysql-data-types.html)

## 4.1 数值类型

包括严格数值数据类型(`INTEGER`、`SMALLINT`、`DECIMAL和NUMERIC`)，以及近似数值数据类型(`FLOAT`、`REAL`和`DOUBLE PRECISION`)。

## 4.2 日期和时间类型

表示时间值的日期和时间类型为`DATETIME`、`DATE`、`TIMESTAMP`、`TIME`和`YEAR`。

## 4.3 字符串类型

字符串类型指`CHAR`、`VARCHAR`、`BINARY`、`VARBINARY`、`BLOB`、`TEXT`、`ENUM`和`SET`

# 5 主键

 唯一标识表中每行的这个列（或这组列）称为主键（primary key）。没有主键，更新或删除表中特定行很困难，因为没有安全的方法保证只涉及相关的行。

虽然并不总是都需要主键，但大多数数据库设计人员都应保证他们创建的每个表有一个主键，以便于以后数据操纵和管理

## 5.1 主键的设计原则：

- MySQL主键应当是对用户没有意义的。
- MySQL主键应该是单列的，以便提高连接和筛选操作的效率
- 永远也不要更新MySQL主键
- MySQL主键不应包含动态变化的数据，如时间戳、创建时间列、修改时间列等
- MySQL主键应当有计算机自动生成。

## 5.2 主键设计的常用方案

### （1）自增ID

优点：

1. 数据库自动编号，速度快，而且是增量增长，聚集型主键按顺序存放，对于检索非常有利。
2. 数字型，占用空间小，易排序，在程序中传递方便。

缺点：

1. 不支持水平分片架构，水平分片的设计当中，这种方法显然不能保证全局唯一。
2. 表锁
3. 自增主键不连续

### （2）UUID

优点

1. 全局唯一性、安全性、可移植性。
2. 能够保证独立性，程序可以在不同的数据库间迁移，效果不受影响。
3. 保证生成的ID不仅是表独立的，而且是库独立的，在你切分数据库的时候尤为重要

缺点

1. 针对InnoDB引擎会徒增IO压力，InnoDB为聚集主键类型的引擎，数据会按照主键进行排序，由于UUID的无序性，InnoDB会产生巨大的IO压力。InnoDB主键索引和数据存储位置相关（簇类索引），uuid 主键可能会引起数据位置频繁变动，严重影响性能。
2. UUID长度过长，一个UUID占用128个比特（16个字节）。主键索引KeyLength长度过大，而影响能够基于内存的索引记录数量，进而影响基于内存的索引命中率，而基于硬盘进行索引查询性能很差。严重影响数据库服务器整体的性能表现。

# 参考

- [关于mysql8.0安装以及配置navicat的问题@知乎](https://zhuanlan.zhihu.com/p/36087723)
- [MySQL 教程@RUNOOB](http://www.runoob.com/mysql/mysql-tutorial.html)
- [windows下安装MySQL 5.7，创建数据库和数据库表@CSDN](https://blog.csdn.net/chenriyang0306/article/details/54587034)
- [MySQL主键设计@程序员的自我修养](http://www.cnblogs.com/xiekeli/p/5398374.html)
