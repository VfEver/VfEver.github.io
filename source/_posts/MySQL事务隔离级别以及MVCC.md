---
title: MySQL事务隔离级别以及MVCC
date: 2019-04-16 00:20:13
categories: 技术
tags:
      - MySQL
      - 教程
---
#### MySQL事务隔离级别
前文提到了MySQL的四种隔离级别：  
- READ UNCOMMITTED（读取未提交）
- READ COMMITTED（读取已提交）
- REPEATABLE READ（可重复读）
- SERIALIZABLE（串行化）  
接下来将对这些不同的隔离级别分别进行实验，测试不同隔离级别下对事务的影响以及最佳实践。  

#### 常用SQL命令
在进行不同隔离级别实验前，先进行一些常用sql预热准备。  
查看当前数据库隔离级别sql：
```java
mysql>select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
1 row in set (0.00 sec)
```
更改当前session的事务隔离级别：
```java
mysql> set session transaction isolation level read committed;#设置隔离级别为读已提交
Query OK, 0 rows affected (0.02 sec)
mysql> select @@tx_isolation;
+----------------+
| @@tx_isolation |
+----------------+
| READ-COMMITTED |
+----------------+
1 row in set (0.00 sec)
```
#### 每种隔离级别对事务的影响  
实验数据库信息：  
版本：5.7.9-log  
存储引擎：InnoDB  
表结构：  
```java
mysql> show create table tx_test\G;
*************************** 1. row ***************************
       Table: tx_test
Create Table: CREATE TABLE `tx_test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(18) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
mysql> show create table tx_test_1\G;
*************************** 1. row ***************************
       Table: tx_test_1
Create Table: CREATE TABLE `tx_test_1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `value` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `value` (`value`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
```
数据准备：  
```java
mysql> insert into tx_test(`name`) value ('lu');
Query OK, 1 row affected (0.01 sec)
mysql>  insert into tx_test(`name`) value ('wen');
Query OK, 1 row affected (0.01 sec)
mysql>  insert into tx_test(`name`) value ('wen');
Query OK, 1 row affected (0.01 sec)
mysql> insert into tx_test_1(`value`) value (1);
Query OK, 1 row affected (0.01 sec)
mysql>  insert into tx_test_1(`value`) value (3);
Query OK, 1 row affected (0.01 sec)
mysql> insert into tx_test_1(`value`) value (5);
Query OK, 1 row affected (0.01 sec)
mysql> insert into tx_test_1(`value`) value (6);
Query OK, 1 row affected (0.01 sec)
```
##### 读未提交
在该隔离级别中，事务可以读取其他未提交事务的执行结果。读取未提交的数据，也称之为脏读。实验如下：  
1. 设置当前会话隔离级别为未提交读：
```java
mysql> set session transaction isolation level read uncommitted;
Query OK, 0 rows affected (0.00 sec)
```
2. 开启事务，并在当前事务中进行一次查询（id=1）：  
```java
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
mysql> select * from tx_test where id = 1;
+----+------+
| id | name |
+----+------+
|  1 | lu   |
+----+------+
1 row in set (0.01 sec)
```
3. 打开另一个会话，设置会话隔离级别为未提交读：
```java
mysql> set session transaction isolation level read uncommitted;
Query OK, 0 rows affected (0.00 sec)
```
4. 开启第二个事务，并修改id=1的name值，但是不提交：
```java
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
mysql> update tx_test set name='luge' where id = 1;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
5. 回到第一个窗口，再进行一次id=1的查询：
```java
mysql> select * from tx_test where id = 1;
+----+------+
| id | name |
+----+------+
|  1 | luge |
+----+------+
1 row in set (0.00 sec)
```
可以看到，id=1的name值已经由**lu**修改为**luge**。事务1读取了事务2执行但是未提交的结果。  

##### 读已提交  
在读已提交的隔离级别当中，当前事务可以读取其他事务已经提交的执行结果。这就出现了不可重复读的问题，也就是在同一个事务当中两次或多次执行同一个sql，得到的结果不一致。实验如下：
1. 设置当前会话隔离级别为读已提交：
```java
mysql> set session transaction isolation level read committed;
Query OK, 0 rows affected (0.00 sec)
```
2. 开启事务，并在当前事务中进行一次查询（id=1）：  
```java
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
mysql> select * from tx_test where id = 1;
+----+------+
| id | name |
+----+------+
|  1 | lu   |
+----+------+
1 row in set (0.01 sec)
```  
3. 打开另一个会话，设置会话隔离级别为读已提交：
```java
mysql> set session transaction isolation level read uncommitted;
Query OK, 0 rows affected (0.00 sec)
```
4. 开启第二个事务，并修改id=1的name值，但是不提交：
```java
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
mysql> update tx_test set name='luge' where id = 1;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
5. 回到第一个窗口，再进行一次id=1的查询：
```java
mysql> select * from tx_test where id = 1;
+----+------+
| id | name |
+----+------+
|  1 | lu   |
+----+------+
1 row in set (0.00 sec)
```  
可以看到，id=1的name值还是**lu**，也就是这个隔离级别消除了未提交读带来的脏读。  

6. 此时回到第二个窗口，进行一次事务的提交：
```java
mysql> commit;
Query OK, 0 rows affected (0.01 sec)
```
7. 切换至第一个窗口，再进行一次id=1的查询：
```java
mysql> select * from tx_test where id = 1;
+----+------+
| id | name |
+----+------+
|  1 | luge |
+----+------+
1 row in set (0.00 sec)
```
此时可以看到，在第二个事务对id=1的修改提交（commit）之后，第一个事务能够看到第二个事务提交的内容，读取了第二个事务提交的内容。此时也带来了另一个一致性问题：也就是在同一个事务中，两次请求读取的数据不一致，这就是不可重复读。

##### 可重复读  
可重复读可以理解为，在同一个事务当中，多次执行同一个sql，返回的结果是一致的，不会受到其他事物的影响。实验如下：
1. 设置当前会话隔离级别为可重复读：
```java
mysql> set session transaction isolation level repeatable read;
Query OK, 0 rows affected (0.00 sec)
```
2. 开启事务，并在当前事务中进行一次查询（id=1）：  
```java
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
mysql> select * from tx_test where id = 1;
+----+------+
| id | name |
+----+------+
|  1 | luge |
+----+------+
1 row in set (0.01 sec)
```  
3. 打开另一个会话，设置会话隔离级别为可重复读：
```java
mysql> set session transaction isolation level repeatable read;
Query OK, 0 rows affected (0.00 sec)
```
4. 开启第二个事务，并修改id=1的name值，但是不提交：
```java
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
mysql> update tx_test set name='lulu' where id = 1;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
5. 回到第一个窗口，再进行一次id=1的查询：
```java
mysql> select * from tx_test where id = 1;
+----+------+
| id | name |
+----+------+
|  1 | luge |
+----+------+
1 row in set (0.00 sec)
```  
可以看到，id=1的name值还是**luge**，第二个事务未提交的修改数据对其他事务不可见。
6. 此时回到第二个窗口，进行一次事务的提交：
```java
mysql> commit;
Query OK, 0 rows affected (0.01 sec)
```
7. 切换至第一个窗口，再进行一次id=1的查询：
```java
mysql> select * from tx_test where id = 1;
+----+------+
| id | name |
+----+------+
|  1 | luge |
+----+------+
1 row in set (0.00 sec)
```
此时，在第一个事务当中，相同条件的sql执行出来的结果是一致的，即使另一个事务已经进行了commmit操作。在这个事务隔离级别当中（repeatable read），同一个事物执行相同的sql，肯定会得到相同的结果，实现原理就是通过MVCC实现的。可重复读虽然解决了同一个事务不可重复读的问题，但是同时也引发了另外一个问题——幻读。实验如下：


1. 开启第一个事务，做一次查询：
```java
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
mysql> select * from tx_test;
+----+------+
| id | name |
+----+------+
|  1 | lulu |
|  2 | wen  |
|  3 | wen  |
|  4 | shi  |
+----+------+
4 rows in set (0.00 sec)
```
2. 开启第二个事务，做一次数据插入并提交：
```java
mysql> begin;
Query OK, 0 rows affected (0.00 sec)  
mysql>  insert into tx_test value(5, 'gou');
Query OK, 1 row affected (0.01 sec)
mysql> commit;
Query OK, 0 rows affected (0.02 sec)
```
3. 返回第一个事务，同样做一个与事务二相同的sql插入。插入之前，先查询一边：
```java
mysql> select * from tx_test;
+----+------+
| id | name |
+----+------+
|  1 | lulu |
|  2 | wen  |
|  3 | wen  |
|  4 | shi  |
+----+------+
4 rows in set (0.00 sec)
mysql>  insert into tx_test value(5, 'gou');
ERROR 1062 (23000): Duplicate entry '5' for key 'PRIMARY'
```
此时可以发现，查询的时候并没有出现**id=5**的记录，当试图插入的时候，报了唯一键冲突。这就与预期结果不一致了，明明在这个事务里面查询没有这条记录，插入却报错，反馈已经存在，刚才查询的结果就像幻觉一样。这就是幻读。  
对于范围查询，是可以避免幻读这个问题的。InnoDB存储引擎在repeatable read事务隔离级别下，使用next-key lock算法避免了幻读的产生。现在实验如下：  


1. 开启第一个事务，做一次**value=3**的查询：
```java
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
mysql> select * from tx_test_1 where value = 3 for update;
+----+-------+
| id | value |
+----+-------+
|  2 |     3 |
+----+-------+
1 row in set (0.00 sec)
```
2. 开启第二个事务，做一次**id=4**的插入：
```java
mysql> begin ;
Query OK, 0 rows affected (0.01 sec)
mysql> insert into tx_test_1(`value`) value (4);
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```
此时第二个事物的insert会被阻塞住，直到超时回滚。第一个事务通过select...for update的方式，对**value=3**加了Record Lock和Next-Key Lock。可以看到前面的建表语句（tx_test_1）,列**value**是辅助索引，对于辅助索引，select...for update方式会对记录添加行锁和间隙锁。对此，对唯一主键列**id**（id=2）添加了行锁，对辅助索引列**value**（value=3）添加了（3，5）的Next-key lock，所以第二个事务进行**value=4**的插入的时候，由于锁的存在导致insert失败。这样就避免了在RR隔离级别下可能会出现的幻读。  
这里需要注意一点的问题就是，select...for update会给唯一主键列所在的行添加行锁，如果where条件为辅助索引，则会对辅助索引添加next-key lock（间隙锁），锁住下一个值区间。  

##### 串行化
串行化是最高的事务隔离级别，每个事务在执行的时候都会加上相应的锁，使之不可能冲突。实验如下：  
1. 设置隔离级别为串行化，开启第一个事务，并且做一次查询：
```java
mysql>  set session transaction isolation level serializable;
Query OK, 0 rows affected (0.00 sec)
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
mysql> select * from tx_test;
+----+-------+
| id | name  |
+----+-------+
|  1 | lulu  |
|  2 | wen   |
|  3 | wen   |
|  4 | shi   |
|  5 | gou   |
|  6 | right |
|  7 | right |
|  9 | wrong |
+----+-------+
8 rows in set (0.00 sec)
```
2. 开启第二个事务，先查询，再尝试一次插入：
```java
mysql>  set session transaction isolation level serializable;
Query OK, 0 rows affected (0.00 sec)
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
mysql> insert into tx_test value(100, 'change');
```  
此时这个insert语句会阻塞住直到第一个事务提交commit或者回滚rollback释放锁。由此可以看到，在串行化隔离级别上，事务之间的读取是互相不受影响的，添加的共享锁。但是事务的插入是排他锁，必须串行执行，这样就完全实现了事物之间的隔离属性。  

#### MVCC——多版本并发控制
##### 概念
MVCC (Multiversion Concurrency Control)，即多版本并发控制技术,它使得大部分支持行锁的事务引擎，不再单纯的使用行锁来进行数据库的并发控制，取而代之的是把数据库的行锁与行的多个版本结合起来，只需要很小的开销,就可以实现非锁定读，从而大大提高数据库系统的并发性能。
##### 原理  
如上面隔离级别分析，在可重复读（RR）隔离级别下，通过MVCC避免了提交读情况下的不可重复读问题，下面来仔细分析下MVCC是如何实现在同一个事务中可重复读的。  
MVCC是通过保存数据在某个时间点的快照来实现重复读的，也就是说，保存了每条记录的变更之前的版本，并通过版本号实现记录和控制。InnoDB的MVCC是通过在每行记录后面两个隐藏的列来实现。这两个列，分别保存了这个行的创建版本号，一个保存的是行的删除版本号，指系统版本号（可以理解为事务的ID），每开始一个新的事务，系统版本号就是自动递增，事务开始时刻的系统版本号会作为事务的ID。  
##### 增删改查如何实现MVCC  
1.（事务T1）插入一条数据（INSERT），记录的版本号即当前事务的版本号。假设事物的id为1.

id | name | create version | delete version
-|-|-|-
1 | test | 1 |  

2.（事务T2）对这条数据进行更新（UPDATE），采取的动作时先标记为已删除，删除的版本号就是事务的id，同时insert一条新的记录。比如此时事务的id为2，把name修改为change。  

id | name | create version | delete version
-|-|-|-
1 | test | 1 | 2
1 | change | 2 |
3.（事务T3）删除（delete）操作的时候，就把事务id作为删除版本号。比如此时事物的id为3.  

id | name | create version | delete version
-|-|-|-
1 | test | 1 | 2
1 | change | 2 | 3  

4.查询（SELECT）操作，在查询时要符合以下两个条件的记录才能被事务查询出来：  
4.1.创建版本号小于等于当前事物的版本号（事务id）。  
4.2.删除版本号未指定或者大于当前事务版本号。即确保事务读取到的行，在事务开始之前未被删除。  
如事务T1在T3操作完成之后，继续原条件查询的话，会匹配出创建版本小于等于T1版本号id=1且删除版本号大于本身版本号的那一条记录。也就是：  

id | name | create version | delete version
-|-|-|-
1 | test | 1 | 2

##### 数据读取相关
快照读：读取的是快照版本，也就是历史版本。  
当前读：读取的是最新版本
