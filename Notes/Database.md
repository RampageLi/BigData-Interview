# 数据库

## 1 事务

ACID：原子性、一致性、隔离性、持久性

`redo log` 实现持久性，`undo log` 实现原子性，`锁机制` `MVCC` 实现隔离性

## 2 并发一致性问题

丢失修改、读脏数据、不可重复读、幻影读

## 3 封锁

封锁粒度：行级锁、表级锁【锁开销和并发程度权衡】

读写锁：互斥锁、共享锁

意向锁：想在表中某个数据行加锁

## 4 封锁协议

一级封锁协议：必须加写锁，解决丢失修改

二级封锁协议：必须加读锁，解决读脏数据

三级封锁协议：事务结束才释放读锁，解决不可重复读

## 5 隔离级别

![img](https://camo.githubusercontent.com/1632a88a3a4fa7954026cb939edf2f8a30bb5d60a1bce4921c7e0d0e4d245739/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230373232333430303738372e706e67)

## 6 多版本并发控制

+ 用于实现**提交读**和**可重复读**
+ 写操作更新最新的版本快照，而读操作去读旧版本快照
+ MVCC 规定只能读取已经提交的快照。当然一个事务可以读取自身未提交的快照
+ 修改操作会新增一个版本快照
+ ReadView：保存 TRX_IDs，存放的是未提交的事务

## 7 Next-Key Locks

+ 目的为解决幻影读
+ Record Locks：锁定一个记录的索引
+ Gap Locks：锁定索引之间的间隙
+ 锁定前开后闭区间

## 8 DQL、DML、DDL、DCL

**DQL**

常写的 SQL 语句：

```sql
select
	...
from
	...
where
	...
```

**DML**

增、删、改操作。

**DDL**

创建表，删除表的等操作。

**DCL**

授权、回滚、提交。

## 9 索引优点

保证每行唯一性；

提升检索速度；

避免排序和临时表（B+ Tree 有序）；

顺序IO；

加速表连接。

## 10 索引缺点

增、删、改时，索引需要动态维护；

索引需要占物理空间；

创建、维护索引需要时间。

## 11 索引使用原则

经常查询的列；

WHERE 语句中出现的列；

需要排序的列；

经常连接的列；

特大型表不适合索引（创建、维护成本）。

## 12 覆盖索引

```sql
select 
	username
	,age 
from 
	user 
where 
	username = 'Java' 
	and age = 22
```

查询数据都在叶子节点，无需回表。

## 13 索引如何提升查询速度

MySQL 的基本存储结构是页；

不同数据页构成双向链表，每个数据页的记录是单向链表；

采用非主键查询时，首先定位页，然后查找相应记录；

采用索引后，进行二分查找，降低时间复杂度。

## 14 索引的最左前缀原则

如果联合索引设置为 `index_name (num,name,age)` ，那么当查询的条件有为:num / (num AND name) / (num AND name AND age) 时，索引生效。

```sql
select * from user where name=xx and city=xx ; ／／可以命中索引
select * from user where name=xx ; // 可以命中索引
select * from user where city=xx ; // 无法命中索引 
```

## 15 冗余索引

index(a, b) 和 index(a)，index(a) 就是冗余索引。

## 16 如何创建索引 

```sql
# 主键索引
ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` )  

------------------------二级索引------------------------
# 唯一索引
ALTER TABLE `table_name` ADD UNIQUE ( `column` ) 

# 普通索引
ALTER TABLE `table_name` ADD INDEX index_name ( `column` )

# 全文索引
ALTER TABLE `table_name` ADD FULLTEXT ( `column`) 

# 多列索引
ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )
```

## 17 聚集索引和非聚集索引

**聚集索引**

数据和索引结构放在一起（如主键索引）；

查询速度快；

依赖于有序数据，如果无序，需要在插入时排序；

更新代价大。

**非聚集索引**

数据和索引结构分开存放；

叶子节点存放主键；

因为叶子节点不存放数据，更新代价小；