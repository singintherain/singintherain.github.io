---
layout: post
title: JDBC异常简介
description: 由MyBatis的异常引出的JDBC异常简介
category: blog
---

## JDBC API Overview

The JDBC API is a call-level API for SQL-based database access. It possible does
three things;

* Establish a connection with a database or access any tabular data source
* Send SQL statements
* Process the results

## 正常的一次数据库操作

* 建立连接
* 生成SQL待执行命令,即statement
* 执行这个命令
* 处理执行结果
* 关闭连接

## 建立数据库连接的方式

* DriverManager

这个类会将app与一个URL标识数据源(data source)建立连接，而且会默认自动的从classPath从
查找JDBC 4.0驱动，所以你的app需要预先手动加载JDBC 4.0驱动。

* DataSource

这个方法比较受推崇，它屏蔽了应用和低层数据库连接的细节，你只要提供一个合法的DataSource对象
的属性配置就可以了。

## DataSource连接数据库

### 如何建立连接

### 连接池

管理应用和数据源的之间的连接，避免无限制的创建连接和销毁连接，因为每个连接都是很
耗费资源的。容器在启动时可以预先创建

* 测试

1. 在不使用连接池的情况下，数据库启动是规定一个较小的最大连接数，使用单线程创建多于最大数据库
连接数的连接，查看发生什么情况。

* 测试经历:

混淆了JPA概念, 本来想着JPA是一项类似于MyBatis或者Hibernate的技术，因此查询是否有单独的JPA的Jar包，
仅仅引入改Jar包就可以顺利的访问数据库，做到测试项目的最小化。
但是发现JPA只是一项标准，各个产品都有其实现。

事实上，我只需要一个DataSource就可以了，而DataSource是和Hibernate或者MyBatis是分离的。

** 如何查看Mysql Server现在有多少个connection:

show global status : 查看mysql server所有的状态信息
show status like 'connections': 这种格式可以查看数据库的连接统计
show global variables: 查看mysql server 的参数配置
set global max_connections = 10; 设置mysql server 的最大连接数
show processlist: 查看mysql server 的连接线程信息

### 如何处理sql执行结果

如何判断sql执行成功还是失败，失败一定会抛出异常吗

查询操作使用executeQuery(sql)，返回值为ResultSet
更新操作使用execute(sql)，返回值为boolean变量

如果执行失败，大多情况下都是违背了表的完整性约束，此时会抛出MySQLIntegrityConstraintViolationException
例如，出现重复的key数据，在插入时会抛出该异常

selectOne 和 selectList 的不同仅仅是 selectOne 必须返回一个对象。 如果多余一个, 或者 没有返回 (或返回了 null) 那么就会抛出异常。

在ibatis把本身jdbc规定的insert update delete select抛出的SqlExecption都转换为了RuntimeException，使得所有的操作都抛出非首检异常
这样的话，在使用中需要手动捕获特定的异常

SqlSession在哪个阶段关闭的
新建SqlSession对象-> 设置一个连接Connection -> 开启一个事务
-> 执行操作 -> 事务提交 -> 开启多个事务 ... -> 

数据库中字段定义为not null时，即使在数据库中给了default值，在做sql插入时，如果没有定义值，会抛出数据完整性异常.

ibatis的insert update返回值只是标识修改的行数，insert语句操作后，会直接将生成的
新条目的id赋给传递的对象id

### Transactional

@Transactional注解只是提供元数据，需要在Spring配置文件中通过指定
<tx:annotation-driven transaction-manager="txMansger"/>
来通知Spring容器对标注@Transactional的Bean进行加工处理@Transactional

@Transactional注解可以应用于接口定义和接口方法、类定义和类的public方法上
但Spring建议在业务类上使用@Transactional注解

不能在JUnit中针对Test类使用@Transactional，因为spring对Transactional修饰的bean类创建代理类，而spring无法对Test创建代理类



