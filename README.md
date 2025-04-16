# Database-Systems_CMU15445
This is the CMU15445-Database Systems course project, consisting of five lab assignments. 

在本课程项目中，需要完成关系数据库bustub的磁盘页面缓冲池、索引功能、查询执行功能以及并发控制功能。并在课程中学习关如下知识：

关系数据库的基本概念，以及基于SQLite简易关系数据库的SQL语句语法及其实践；
关系数据库中页面存储、页面布局及元组布局等数据库物理存储模型，以及DBMS中的页面缓冲池的功能及原理；
关系数据库中索引的功能及分类，以及基于B+-tree和可扩展哈希表的可磁盘备份索引的具体实现及并发控制；
关系数据库中元组排序、聚合、连接功能的具体实现和性能分析，连接、聚合、插入等物理查询计划的具体执行策略，以及基于启发式和成本模型的查询计划优化方法；
关系数据库中事务的基本概念及其所应达到的标准ACID(原子性、一致性、隔离性、持久性)的定义及要求；
关系数据库中并发控制协议（即执行多个事务的方式）的悲观及乐观实现策略，如两阶段锁、基于时间戳并发控制、多版本并发控制；
关系数据库中保证原子性、隔离性、持久性的崩溃恢复算法的策略，包括日志、检查点、重做和撤销操作；

In this course project, it is required to complete the implementation of key functionalities in the relational database Bustub, including disk page buffer pool, indexing, query execution, and concurrency control. The course covers the following topics:

- Basic concepts of relational databases, as well as SQL syntax and practical applications based on SQLite, a simple relational database;
- Physical storage models in databases, such as page storage, page layout, and tuple layout, as well as the functionality and principles of the page buffer pool in a DBMS;
- The functionality and categories of indexes in relational databases, along with the implementation and concurrency control of disk-backed indexes based on B+-tree and extendible hashing;
- Implementation and performance analysis of tuple sorting, aggregation, and join functionalities in relational databases, execution strategies for physical query plans like joins, aggregations, and insertions, as well as query optimization methods based on heuristics and cost models;
- Basic concepts of transactions in relational databases and the standards they should meet (ACID: Atomicity, Consistency, Isolation, Durability) along with their definitions and requirements;
- Pessimistic and optimistic concurrency control protocols (i.e., methods for executing multiple transactions) in relational databases, such as two-phase locking, timestamp-based concurrency control, and multi-version concurrency control;
- Crash recovery algorithms to ensure atomicity, isolation, and durability in relational databases, including strategies like logging, checkpoints, redo, and undo operations.
