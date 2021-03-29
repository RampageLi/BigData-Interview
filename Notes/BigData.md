# 大数据技术框架

## 1 性能调优

### 1.1 数据倾斜

 **数据倾斜原理**

相同 key 数据在 shuffle 阶段分配到同一个 reduce task 进行处理，某个 key 所对应的数据远大于其他 key。

**数据倾斜现象**

大部分 task 运行速度很慢，卡在最后几个 task；

报错：JVM Out Of Memory。

**数据倾斜问题定位**

使用命令 `yarn logs -applicationId $application_id` 查看日志；

数据倾斜基本只能是发生 shuffle 操作，找可能产生 shuffle 的算子，如： groupByKey、countByKey、join；

定位 stage，调整对应部分代码。

**解决方法一：聚合数据源**

在 Hive ETL 部分对数据做聚合，比如对同一个 key 的数据，采用 `concat_ws`、 `collect` 等命令拼接为一条字符串，或者直接进行计算；

放粗粒度，比如按照城市、地区对数据进行聚合，尽量减少每个 key 对应的数据量。

**解决方法二：提高 reduce 并行度**

增加 reduce task 的数量，让每个 reduce task 分配到的数据量变少；

给 shuffle 算子 增加一个参数，代表 reduce 端的并行度。

**解决方法三：随机 key 实现双重聚合**

针对 groupByKey 和 reduceByKey；

给 key 添加随机前缀，使相同的 key 变成不同的 key，之后进行局部聚合，然后全局聚合。

**解决方法四：将 reduce join 转为 map join**

将较小的 RDD 进行 broadcast，解决了 join 可能导致的数据倾斜。

较小的 RDD 放在 excutor。

**解决方法五：sample 采用两次倾斜 key 进行两次 join**

将会发生倾斜的 key 单独拉出来，放到一个 RDD 中，使用该 RDD 和别的 RDD 进行 join，key 对应的数据可能会分散到多个 task 进行 join，最后将结果 union。

**解决方法六：采用随机数和扩容表进行 join**

选择较小的一个 RDD，使用 `flatMap` 扩容并给 key 打上随机前缀进行 join；

另外一个 RDD，只做普通 map，且打上随机数。

## 2 Spark 相关知识

### 2.1 Shuffle

**概述**

shuffle write 将数据存储在 **blockManager**，数据位置元信息上报到 **mapOutPutTracker**；

下一个 stage 根据数据位置元信息拉去上个 stage 的输出数据，进行 shuffle read；

shuffle manager 负责 shuffle 的执行。

**Hash based shuffle**

小文件数目 = M * R；

产生很多写小文件对象、读小文件对象；

JVM 堆内存对象过多造成 GC，GC 无法解决，造成 OOM；

**Hash based shuffle （合并机制）**

小文件数目 = C * R；

**Sort shuffle （普通运行机制）**

小文件数目 = M * 2；

map task 计算结果写入内存结构（map or array）；

到达阈值溢写到磁盘，溢写前进行排序分区；

分批（batch）的形式进行溢写（默认是10000条）；

多个小文件合并成大文件，并生成索引文件。

**Sort shuffle （byPass 运行机制）**

shuffleMapTask 的数量小于默认值200，开启 byPass 运行机制；

因为数据量比较小，不需要进行 sort。

## 3 Hive 相关知识

### 3.1 Hive 调优

**使用 `group by` 代替 `count(distinct col)`**

distinct 会将 col 列所有数据保存至内存中，可能发生 OOM。

**避免使用 select ***

占用程序资源，造成资源浪费。

**处理空值**

直接过滤；

给空值赋予随机数。

**不要在 join 后加 where**

两表连接时，采用 `谓词下推` 技术，将 where 放到子查询进行。

**开启并行执行**

set hive.exec.parallel=true。

**使用 tez 引擎**

hive.execution.engine = tez。

**采用本地模式**

hive.exec.mode.local.auto=true。

**使用严格模式**

避免 where 扫描所有分区；

order by 排序后所有数据会发往同一个 reducer 进行处理，必须加 limit。

**合理设置 map 和 reduce 数**

map 启动、初始化时间大于逻辑处理时间，造成资源浪费；

单个 map 记录过多，耗时大；

reduce 过多会造成小文件多，启动、初始化时间过长。

## 4 Hadoop 相关知识

### 4.1 MapReduce 连接两表

在 map 阶段，根据数据来源给每条数据打标，如下所示：

```java
context.write(new Text(id), new Text("#" + city));
```

在 reduce 阶段，根据打的标签放到不同 list：

```java
if (value.startsWith("#")) {
	value = value.substring(1);
	list1.add(value);
} else if (value.startsWith("$")) {
	value = value.substring(1);
	list2.add(value);
}
```

两表相同 key 数据进行笛卡尔积。