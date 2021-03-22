# MySQL

## 1 B+ Tree

+ 叶子节点全在同一层，B Tree 的基础上添加叶子节点的顺序访问指针
+ 全称为 balance tree
+ 一个节点的 key 从左到右非递减排序
+ 如果某个指针的左右相邻 key 分别是 key_1 和 key_2，且不为 null，则该指针指向节点的所有 key 大于等于 key_2 且小于等于 key_2

## 2 与红黑树进行比较

1. 更低的树高
2. 磁盘访问原理：一个节点的大小为页大小，一次 I/O 可完全载入一个节点
3. 磁盘预读特性：顺序读取相邻节点

## 3 MySQL 索引

+ B+ Tree 索引
+ 哈希索引
+ 全文索引
+ 空间数据索引

## 4 索引优化

+ 独立的列：不能是表达式的一部分，不能是函数参数
+ 采用多列索引
+ 选择性强的索引放在前面
+ 前缀索引：只索引开始的部分字符
+ 覆盖索引：索引包含想要查询的所有数据

## 5 性能优化

1. 减少请求数据量：只选择特定列，LIMIT限制行，缓存数据
2. 切分大查询
3. 分解大连接查询

## 6 存储引擎

+ InnoDB 和 MyISAM 的比较

  1. 行级锁：InnoDB 支持，MyISAM不支持

  2. 崩溃后修复：MyISAM 执行速度更快，修复过程可能导致数据丢失
  3. 外键：InnoDB 支持
  4. MVCC：InnoDB 支持
  5. 备份：InnoDB 支持在线热备份