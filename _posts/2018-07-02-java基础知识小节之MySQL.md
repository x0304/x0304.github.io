layout: post
title: "java基础知识小节之MySQL"
date: 2018-07-02 
description: "Java基础知识 前言 基础语法"

## tag: MySQL 

## 前言

根基不牢地动山摇!基础是我们能稳步发展的保障,他是一个平台,只有很好的掌握了基础知识,我们的发展才不至于出现片面或大的漏洞.

现在我想将我以前学过的知识点结合看到的一些资料在这里做个总结,争取做到每日一更.

 

## 知识点一 ： 基本概念

- ### 数据库基础:

   InnoDB是一个可靠的事务处理引擎，不支持全文本搜索

   MyISAM是一个性能极高的引擎，支持全文本搜索，不支持事务处理

- ### 数据库(database):

  保存有组织的数据的容器(通常是一个文件或一组文件)

- ### 表(table):

  某种特定类型数据的结构化清单

- ### 模式(schema):

  关于数据库及表的布局及特性的信息

- ### 列(column):

  表中的某个字段，所有表都是由一个或者多个列组成

- ### 数据类型(datatype):

   所容许的数据的类型，每个表列都有相应的数据类型，它限制该列中存储的数据

- ### 行(row):

  表中的一个数据

- ### 主键(primary key):

  一列或一组列，其值能够唯一区分表中的每一行

------

## 知识点二： 复制原理与流程

### 基本原理流程：

 主：binlog线程，记录下所有改变了数据库数据的语句，放进master上的binlog中；

从：io线程，在使用 start slave之后，负责从master上拉取binlog内容，放进自己的relay log中；

从：sql执行线程，执行relay log中的语句；

------

## 知识点三： mysql中myisam与innodb区别

- InnoDB支持事务，而myISAM不支持事务

-    InnoDB支持行级锁，而MyISAM支持表级锁

-    InnoDB支持MVCC，而MyISAM不支持

-    InnoDB支持外键，而MyISAM不支持

  #### InnoDB引擎的四大特性：

  插入缓存（insert buffer），二次写（double write），自适应哈希索引（ahi），预读（read ahead）

------

## 知识点四： InnoDB的事务与日志的实现方式

### 1. InnoDB的事务

- 错误日志
- 查询日志
- 慢查询日志
- 二进制日志
- 中继日志
- 事务日志

### 2. 事务的4种隔离级别：

- 读未提交 RU
-  读已提交 RC
- 可重复读 RR
- 串行

### 3.事务是如何通过日志来实现的：

- 事务日志是通过redo和innodb的存储引擎日志缓冲（InnoDB log buffer）来实现的，当开始一个事务时，会记录该事务的lsn（log sequence number）号，当事务执行时，会往InnoDB存储引擎的日志缓存里面啊插入事务日志
- 当事务提交时，必须将存储引擎的日志缓冲写入磁盘（通过 innodb_flush_log_trx_commit来控制）也就是写数据前，需要先写日志，这种方式称为“预写日志方式”

------

## 知识点五： Mysql中binlog的几种日志录入格式及其区别

1. ### Statement 每一条会修改数据的sql都会记录在binlog中

- 优点 不需要记录每一行的变化，减少了binlon的日志量，节约了IO，提高性能

相比row能节约多少性能与日志量，这个取决于应用的sql情况，正常同一条记录修改或者插入row格式所产生的日志量，但是考虑到如果带条件的update操作，以及整表删除，alter删除，row格式表等操作，row格式会产生大量日志文件

因此在考虑是否使用row格式日志时应该根据应用的实际情况，其所产生的日志量会增加多少，以及带来的IO性能问题

- 缺点 由于记录的只是执行语句，为了这些语句能在slave上正确运行，因此还必须记录每条语句在执行的时候的一些相关信息，以保证所有语句能在slave得到和在master端执行时候相同的结果，

 另外mysql的复制就像一些特定函数功能，slave可与master上要保持一致会有很多相关问题（如sleep（）函数，last_insert_id()，user-defined function(udf)会出现问题）使用以下函数的语句也无法复制LOAD_FILE()     UUID()   USER()    FOUND_ROWS()  SYSDATE()

2. ### row  不记录sql语句上下文相关信息，仅保存哪条记录被修改，基于数据行的模式

- 优点  binlog中可以不记录执行的sql语句的上下文相关的信息，仅需要记录那一条记录被修改成什么了。所以rowlevel的日志内容会非常清楚地记录下每一行数据修改的细节，而且不会出现某些特定情况下的存储过程，或function以及trigger的调用和触发无法被正确复制的问题
- 缺点  所有的执行语句当记录到日志中的时候，都将以每行记录的修改来记录，这样可能会产生大量的日志内容，比如一条update 语句会修改大量的内容，则binlog中每一条修改都会有记录，这样造成binlog日志量很大
- 特别是alert table之类的语句时 会修改整个表结构，每条 记录也会跟着修改，那么该表每一条记录都会记录到日志中

3. ### Mixedlevel  是以上两种level的混合使用 

- 一般的语句修改使用statement格式保存在binlog中，如一些函数，statement无法完成主从复制，则采用row格式保存binlog，mysql会根据执行的每一条sql语句来区分对待记录的日志形式，也就是在statement和row之间选择一种
- 在新版本的mysql中对row做了优化，并不是所有的修改都会以row来记录，像遇到表结构变更时就会以statement模式来记录，至于update和dalete等修改数据的语句，还是会记录所有修改的变更

------



转载请注明原地址，王旭的博客：[https://x0304.github.io/archive/](https://x0304.github.io/archive/) 谢谢！