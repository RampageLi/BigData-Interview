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

