---
title: mysql事务的隔离级别
tags:
  - mysql
categories:
  - mysql
date: 2020-08-29 15:16:23
---

事务的四个特性是什么，四种隔离级别分别能够解决并发事务中的哪些问题。

<!--more-->

# 1. 事务的四个特性(ACID)

**● 原子性**（Atomicity）**：**操作这些指令时，要么全部执行成功，要么全部不执行。只要其中一个指令执行失败，所有的指令都执行失败，数据进行回滚，回到执行指令前的数据状态。

**● 一致性**（Consistency）**：**事务的执行使数据从一个状态转换为另一个状态，但是对于整个数据的完整性保持稳定。

```
拿转账来说，假设用户A和用户B两者的钱加起来一共是20000，那么不管A和B之间如何转账，转几次账，
事务结束后两个用户的钱相加起来应该还得是20000，这就是事务的一致性。
```

**● 隔离性**（Isolation）**：**隔离性是当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离。即要达到这么一种效果：对于任意两个并发的事务T1和T2，在事务T1看来，T2要么在T1开始之前就已经结束，要么在T1结束之后才开始，这样每个事务都感觉不到有其他事务在并发地执行。

**● 持久性**（Durability）**：**当事务正确完成后，它对于数据的改变是永久性的。

```sql
例如我们在使用JDBC操作数据库时，在提交事务方法后，提示用户事务操作完成，
当我们程序执行完成直到看到提示后，就可以认定事务以及正确提交，
即使这时候数据库出现了问题，也必须要将我们的事务完全执行完成，
否则就会造成我们看到提示事务处理完毕，但是数据库因为故障而没有执行事务的重大错误。
```

# 2. 并发事务导致的问题

在许多事务处理同一个数据时，如果没有采取有效的隔离机制，那么并发处理数据时，会带来一些的问题。

**脏读：**脏读是指在一个事务处理过程里读取了另一个未提交的事务中的数据。

**不可重复读：**一个事务两次读取同一行的数据，结果得到不同状态的结果，中间正好另一个事务更新了该数据，两次结果相异。

**幻读：**一个事务执行两次查询，第二次结果集包含第一次中没有或某些行已经被删除的数据，造成两次结果不一致，只是另一个事务在这两次查询中间插入或删除了数据造成的。**幻读是事务非独立执行时发生的一种现象。

```
不可重复读的重点是修改:
同样的条件, 你读取过的数据, 再次读取出来发现值不一样了

幻读的重点在于新增或者删除
同样的条件, 第1次和第2次读出来的记录数不一样
```

# 3. 隔离级别

**Read uncommitted（最低级别，任何情况都无法保证。）**

读未提交，顾名思义，就是一个事务可以读取另一个未提交事务的数据。

**Read committed（可避免脏读的发生。）**

读提交，顾名思义，就是一个事务要等另一个事务提交后才能读取数据。

**Repeatable read（可避免脏读、不可重复读的发生。）**

重复读，就是在开始读取数据（事务开启）时，不再允许修改操作

**Serializable（可避免脏读、不可重复读、幻读的发生。）** 序列化

Serializable 是最高的事务隔离级别，在该级别下，事务串行化顺序执行，可以避免脏读、不可重复读与幻读。但是这种事务隔离级别效率低下，比较耗数据库性能，一般不使用。

| 隔离级别 | 异常情况 |            |      |
| -------- | -------- | ---------- | ---- |
| 读未提交 | 脏读     | 不可重复读 | 幻读 |
| 读已提交 |          | 不可重复读 | 幻读 |
| 可重复读 |          |            | 幻读 |
| 序列化   |          |            |      |

# 4. mysql事务测试

## 4.1 准备

1、打开mysql的命令行，将自动提交事务给关闭

```sql
--查看是否是自动提交 1表示开启，0表示关闭
select @@autocommit;
--设置关闭
set autocommit = 0;
```

2、数据准备

```sql
--创建数据库
create database tran;
--切换数据库 两个窗口都执行
use tran;
--准备数据
 create table psn(id int primary key,name varchar(10)) engine=innodb;
--插入数据
insert into psn values(1,'zhangsan');
insert into psn values(2,'lisi');
insert into psn values(3,'wangwu');
commit;
```

## 4.2 测试脏读

```mysql
set session transaction isolation level read uncommitted;
A:start transaction;
A:select * from psn;
B:start transaction;
B:select * from psn;
A:update psn set name='msb';
A:selecet * from psn
B:select * from psn;  --读取的结果msb。产生脏读，因为A事务并没有commit，读取到了不存在的数据
A:commit;
B:select * from psn; --读取的数据是msb,因为A事务已经commit，数据永久的被修改
```

## 4.3 测试不可重复读

当使用read committed的时候，就不会出现脏读的情况了，但会出现不可重复读的问题

```mysql
A:start transaction;
A:select * from psn;
B:start transaction;
B:select * from psn;
--执行到此处的时候发现，两个窗口读取的数据是一致的
A:update psn set name ='zhangsan' where id = 1;
A:select * from psn;
B:select * from psn;
--执行到此处发现两个窗口读取的数据不一致，B窗口中读取不到更新的数据
A:commit;
A:select * from psn;--读取到更新的数据
B:select * from psn;--也读取到更新的数据
--发现同一个事务中多次读取数据出现不一致的情况
```

## 4.4 测试幻读

当使用repeatable read的时候，就不会出现不可重复读的问题，但是会出现幻读的问题。

```mysql
A:start transaction;
A:select * from psn;
B:start transaction;
B:select * from psn;
--此时两个窗口读取的数据是一致的
A:insert into psn values(4,'sisi');
A:commit;
A:select * from psn;--读取到添加的数据
B:select * from psn;--读取不到添加的数据
B:insert into psn values(4,'sisi');--报错，无法插入数据
--此时发现读取不到数据，但是在插入的时候不允许插入，出现了幻读，设置更高级别的隔离级别即可解决
```

---

Tips：

```sql
大多数数据库默认的事务隔离级别是Read committed，比如Sql Server , Oracle。Mysql的默认隔离级别是Repeatable read。

隔离级别的设置只对当前链接有效。
对于MySQL命令窗口而言，一个窗口就相当于一个链接，当前窗口设置的隔离级别只对当前窗口中的事务有效；
对于JDBC操作数据库来说，一个Connection对象相当于一个链接，而对于Connection对象设置的隔离级别只对该Connection对象有效，与其他链接Connection对象无关。

设置数据库的隔离级别一定要是在开启事务之前。
```

