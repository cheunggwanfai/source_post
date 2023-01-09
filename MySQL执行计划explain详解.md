---
title: MySQL执行计划explain详解
date: 2023-01-09 22:18:03
categories: [Technique]
tags: [mysql]
---

说到MySQL语句优化，必然就会想到explain，那么，今天我们来详细讲解下explain。


<!--more-->


**语法**
```
    mysql> explain select id from user;
    +----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
    | id | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
    +----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
    |  1 | SIMPLE      | user  | index | NULL          | PRIMARY | 4       | NULL |    8 | Using index |
    +----+-------------+-------+-------+---------------+---------+---------+------+------+-------------+
```
他的用法很简单，直接在select语句前面加上explain就可以了。

**参数解释**

**1. id**
id是SELECT 查询的标识符，每个 SELECT 都会自动分配一个唯一的标识符。该值可能为NULL，如果这一行用来说明的是其他行的联合结果。

**2. select_type**
SELECT 查询的类型，取值可以有一下几种：

- (1)  SIMPLE  简单查询，不包含子查询、联合查询

- (2)  PRIMARY  复杂查询中最外层的查询

- (3)  DERIVED  包含在from子句中的子查询，eg：explain select id from (select ...)
```
    mysql> explain select uid from (select * from user) b;
    +----+-------------+------------+------+---------------+------+---------+------+------+-------+
    | id | select_type | table      | type | possible_keys | key  | key_len | ref  | rows | Extra |
    +----+-------------+------------+------+---------------+------+---------+------+------+-------+
    |  1 | PRIMARY     | <derived2> | ALL  | NULL          | NULL | NULL    | NULL |    8 | NULL  |
    |  2 | DERIVED     | user       | ALL  | NULL          | NULL | NULL    | NULL |    8 | NULL  |
    +----+-------------+------------+------+---------------+------+---------+------+------+-------+
```
- (4) UNION  在UNION查询中第二个和随后的SELECT查询

- (5) UNION RESULT  UNION查询的结果，id列为NULL
```
    mysql> explain select uid from user union select uid from score;
    +----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
    | id | select_type  | table      | type | possible_keys | key  | key_len | ref  | rows | Extra           |
    +----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
    |  1 | PRIMARY      | user       | ALL  | NULL          | NULL | NULL    | NULL |    8 | NULL            |
    |  2 | UNION        | score      | ALL  | NULL          | NULL | NULL    | NULL |    8 | NULL            |
    | NULL | UNION RESULT | <union1,2> | ALL  | NULL          | NULL | NULL    | NULL | NULL | Using temporary |
    +----+--------------+------------+------+---------------+------+---------+------+------+-----------------+
```

- (6) SUBQUERY  子查询中的第一个SELECT，不在from子句中
```
    mysql> explain select uid from user where uid = (select uid from score);    
    +----+-------------+-------+------+---------------+------+---------+------+------+-------------+
    | id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
    +----+-------------+-------+------+---------------+------+---------+------+------+-------------+
    |  1 | PRIMARY     | user  | ALL  | NULL          | NULL | NULL    | NULL |    8 | Using where |
    |  2 | SUBQUERY    | score | ALL  | NULL          | NULL | NULL    | NULL |    8 | NULL        |
    +----+-------------+-------+------+---------------+------+---------+------+------+-------------+
```

- (7) DEPENDENT UNION  UNION中第二个或者后面的查询语句，取决于外面的查询

- (8) DEPENDENT SUBQUERY  子查询中的第一个SELECT，取决于外面的查询，即子查询依赖外面的查询结果
```
mysql> explain select uid from user where uid in(select uid from score union select uid from user);
+----+--------------------+------------+------+---------------+------+---------+------+------+-----------------+
| id | select_type        | table      | type | possible_keys | key  | key_len | ref  | rows | Extra           |
+----+--------------------+------------+------+---------------+------+---------+------+------+-----------------+
|  1 | PRIMARY            | user       | ALL  | NULL          | NULL | NULL    | NULL |    8 | Using where     |
|  2 | DEPENDENT SUBQUERY | score      | ALL  | NULL          | NULL | NULL    | NULL |    8 | Using where     |
|  3 | DEPENDENT UNION    | user       | ALL  | NULL          | NULL | NULL    | NULL |    8 | Using where     |
| NULL | UNION RESULT       | <union2,3> | ALL  | NULL          | NULL | NULL    | NULL | NULL | Using temporary |
+----+--------------------+------------+------+---------------+------+---------+------+------+-----------------+
此SELECT语句可写成，select uid from user a where exists(select uid from score where score.uid = a.uid union select uid from user where user.uid = a.uid)
```

**3. table**
涉及到的表或派生表
\<derived N> N为id值，指该id值对应的那一步操作的结果
<union M, N> union的结果

**4. type**
用来说明表与表之间是如何关联操作的，有没有使用索引。

- (1) system  表仅有一行且满足条件

- (2) const  针对主键或唯一索引的等值查询，最多只返回1条数据

- (3) eq_ref  主键或唯一索引的所有部分都被使用 

- (4) ref  使用普通索引或唯一索引的部分前缀，可用于=，\<=>操作符的带索引的列 

- (5) fulltext  按全文索引进行搜索 

- (6) range  使用索引进行范围查询，ref字段为NULL(MySQL无法使用范围列后面的其他索引) 

- (7) index  全索引扫描 

- (8) all  全表扫描

 性能比较：system > const > eq_ref > ref> range> index > all 

**5. possible_keys**
能够用到的索引

**6. key**
真正使用的索引

> possible_keys 揭示了哪个索引能有助于高效查找
> key 揭示了哪个索引可以最小化查询成本

**7. key_len**
索引字节数
 char(n) n字节
 varchar(n) utf8 (3n+2)字节
 utf8mb4 (4n+2)字节
 tinyint 1字节
 smallint 2字节
 mediumint 3字节
 int 4字节
 bigint 8字节
 date 3字节
 timestamp 4字节
 datetime 8字节

**8. rows**
检查的行数，是一个预估值(取第1页和最后一页的数据，以及往右再取8个页的数据，计算这10页的平均值就是所要检查的行数。可以查看MySQL里面InnoDB的数据结构：https://blog.csdn.net/bohu83/article/details/81086474)

**9. ref**
使用了哪列或常数与key列一起从表中选择行

**10. Extra**
额外信息
using filesort MySQL需要额外的排序，不能通过索引顺序达到排序
using index 覆盖索引扫描，查询可在索引树中查找所需的数据，不需要扫描表数据文件
using temporary 查询使用到临时表
using where MySQL在存储引擎检索行后再进行过滤，先读数据，再按where条件检查，符合留下，不符丢弃

**explain限制**
- (1) 不会告诉你存储过程，触发器会如何影响查询
- (2) 不会告诉你MySQL在执行计划中所做的特定的优化
- (3) 不支持存储过程
