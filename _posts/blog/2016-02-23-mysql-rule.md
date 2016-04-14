---
layout: post
title: Mysql Rule
description: Mysql使用规则总结
category: blog
---

## In VS OR

当条件列相同，候选值有很多个时，使用in或者or可以表达相同的意思，但是查询效率有所区别
例如：查询id为1或者2或者3的student时，可以这么写
select * from students where id in (1, 2, 3);
select * from students where id = 1 or id = 2 or id = 3

or的效率为O(n), in的效率为O(Logn)
当条件列为主键或者有索引时，in和or效率几乎一样，当时当条件列没有索引时，in比or的效率快很多

## Exists VS In

这两个语句的含义同or一样，条件列的候选值为一个集合;但是不同的是这里的候选集合是
一个子查询的结果,or操作不能和子查询配合。
in和exists都从一个候选集合中选择数据，但是数据库对这两个语句的解释不一样。

例如
select * from users where address_id in (select id from address);
select * from users where exists (select * from address where users.address_id = address.id);

数据库语句有内外两层查询
exists时，外层查询会逐条遍历users表，然后判断在address表中是否有address.id=user.address_id
当内层判断返回true时，则认为外层的条目为合法的，将外层条目加到结果集中
使用exists时，内层查询不会返回记录，只会返回true或者false
可以看到这里主要能够用到内层循环的索引，外层查询是一次遍历的，所以不会用到索引

in时，现将子查询的结果记录下来，然后外层执行in语句的条件查询；
自查询在执行的时候，会用到连接，即address表和user表会用到基于users.address_id=address.id的连接，
这样就不会使用到

## 中间表

使用group by、order by、limit时，都会产生中间临时表

## char VS varchar

varchar(10)可以存储长度最大为10的字符串，如果插入的字符串长度大于10，则插入失败

## 构建索引和列操作

新建一个列条目，如果该列条目允许为NULL，则默认给所有现有数据新增的列条目的值为NULL
新建一个列条目，如果该列条目不允许为NULL，则默认给所有现有数据新增的列条目的值为空

```
alter table uniq_null_column_test add column address varchar(20) not null;
```

在有多个为NULL值的列中创建唯一性索引可以成功
在有多个为空值的列中创建唯一性索引不成功

```
create unique index addressx on uniq_null_column_test(address);
drop index addressx on uniq_null_column_test;
```

## 组合索引

组合索引匹配最左匹配规则，(a, b, c)索引，当查询条件为a, a/b, a/b/c时可以用到该索引；
如果a是一个常量，是否可以用到这个索引?

在查询中，当中间的某个列的条件为范围查询时，后续的索引条件就用不到索引了
这里的范围查询类似between and 或者函数比较操作符等，但是注意这里的In关键字不属于范围查询

例如查询条件为a = 1 and b < 3 and c = 4，这是只能用到索引(a, b)

## 索引查找和索引排序

Mysql索引既可以用来查找行，也可以用来排序
索引在排序的时候使用也遵循最左匹配规则, order by 子句的顺序必须和索引顺序完全一致，
并且所有列的排序方向都一样时，Mysql才能使用索引做排序

但是有种例外，考虑组合索引(a, b, c)，查询条件中a为一个常量值，order by b, c，这时
排序时可以使用到组合索引

## Mysql缓存

MySQL查询缓存可以跳过SQL解析优化查询等阶段,直接返回缓存结果给用户
MyBatis的一级缓存策略和Mysql缓存策略一样

### 缓存工作流程

1. 服务器接收SQL,以SQL和一些其他条件为key查找缓存表(额外性能消耗)

2. 如果找到了缓存,则直接返回缓存(性能提升)

3. 如果没有找到缓存,则执行SQL查询,包括原来的SQL解析,优化等.

4. 执行完SQL查询结果以后,将SQL查询结果存入缓存表(额外性能消耗)

### 缓存失效

当某个表正在写入数据,则这个表的缓存(命中检查,缓存写入等)将会处于失效状态.在Innodb中,如果某个事务修改了表,则这个表的缓存在事务提交前都会处于失效状态,在这个事务提交前,这个表的相关查询都无法被缓存.

## 分页

limit a, b

数据库分页使用的时候一般会伴随着排序工作

## Left Join VS Inner Join VS Right Join

关联查询, 将两个表连接起来做关联查询；
不管是左关联、右关联、内关联都会返回这两个表的所有信息；
只是左关联，以左边的表为基础，左边的所有
数据项都会返回，符合关联条件的右表条目信息全部返回，不符合关联条件的右表条目用Null值替换
右关联规则同左关联一样，只是数据已右表为基础
内关联会只返回符合关联规则的左集合和右集合的交集，不符合规则的条目不会返回，
因此不会出现用NULL值替换条目内容的现象

## 延迟关联

延迟关联一般处理分页问题.
所谓的分页就是指使用limit加上偏移量，同时配合一定的order by子句，选择查询结果的部分数据。
当order by用到索引时，一般比较快，否则将做大量的文件排序操作。
limit在数据量比较大的时候，就会出现一个严重的问题，例如limit 20000, 10
该语句是获取排序结果集的从20000开始10列条目，数据库在执行的时候会直接获取20010条数据
然后将前20000条数据抛弃，只返回后10条记录，这样代价比较高，可以使用延迟关联的方式解决

select * from a order by id limit 20000, 10

优化为

select * from a left join (select id from a order by id limit 20000, 10) as b on a.id  = b.id

这样Mysql使用覆盖索引先查询结果列的索引位置，然后在根据索引查询整个表，这样会快很多

## 范围查询

where a > 2 and a < 100，如果a列有索引，是否能够用到？

## union all

## count

