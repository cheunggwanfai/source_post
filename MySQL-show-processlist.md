---
title: MySQL show processlist
date: 2023-01-09 22:14:33
categories: [Technique]
tags: [mysql]
---

show processlist 可以显示哪些sql线程正在进行。在mysql语句执行优化中，是十分重要的一个命令。


<!--more-->

```
mysql> show processlist;
+------+------+-----------+------+---------+------+-------+------------------+
| Id   | User | Host      | db   | Command | Time | State | Info             |
+------+------+-----------+------+---------+------+-------+------------------+
| 7899 | root | localhost | NULL | Query   |    0 | init  | show processlist |
+------+------+-----------+------+---------+------+-------+------------------+
```

### 各列含义 ###
**Id**
线程id，`kill 线程号 //kill掉线程`

**User**
当前sql语句的用户

**Host**
当前语句从哪个ip端口发出，用于追踪问题语句的用户

**db**
当前语句连接的是哪个数据库

**Command**
当前连接执行的命令，一般是Sleep(休眠)，Query(查询)，连接(Connect)

**Time**
线程处在当前状态的持续时间，单位为秒

**State**
当前连接的sql语句的状态，state只是语句执行中的某一个状态。以查询为例，可能经过 `copying to tmp table`，`sorting result`，`sending data` 等状态才可以完成。

**Info**
显示当前的sql语句