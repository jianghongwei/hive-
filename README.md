# hive-
### map端内存设置

set mapred.job.map.memory.mb=6144;

set mapreduce.map.java.opts=-Xmx4915m;

###  小文件处理

1 map 输入合并小文件参数

每个Map最大输入大小(这个值决定了合并后文件的数量)

set mapreduce.input.fileinputformat.split.maxsize=20000000 20M

set mapreduce.input.fileinputformat.split.minsize=20000000 20M

一个节点上split的至少的大小(这个值决定了多个DataNode上的文件是否需要合并)

set mapred.min.split.size.per.node=100000000;

一个交换机下split的至少的大小(这个值决定了多个交换机上的文件是否需要合并)  

set mapred.min.split.size.per.rack=100000000;

执行Map前进行小文件合并

set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;

2 map输出和reduce输出合并相关参数

设置map端输出进行合并，默认为true

set hive.merge.mapfiles = true

设置reduce端输出进行合并，默认为false

set hive.merge.mapredfiles = true

设置合并文件的大小

set hive.merge.size.per.task = 256000000

当输出文件的平均大小小于该值时，启动一个独立的MapReduce任务进行文件merge。

set hive.merge.smallfiles.avgsize=16000000

### mapjoin

默认true开启自动mapjoin优化 

hive.auto.convert.join=false

mapjoin表大小

hive.mapjoin.smalltable.filesize=25000000（25MB）

是否忽略MAPJOIN标记

hive.ignore.mapjoin.hint=false

在基于输入文件大小的前提下将普通JOIN转换成MapJoin，并判断是否将多个MJ合并成一个

hive.auto.convert.join.noconditionaltask=true

多个MJ合并成一个MJ时，其表的总的大小须小于该值

hive.auto.convert.join.noconditionaltask.size=100000000 （100MB）

### hive smb join

smb 会将两个表按照 join 的字段将表分割成桶，这样在join的时候相同的数据必定在同一个桶中，减少了join的IO开销

set hive.auto.convert.sortmerge.join=true;

set hive.optimize.bucketmapjoin = true;

set hive.optimize.bucketmapjoin.sortedmerge = true;

大表选择策略默认是根据当前表的平均分区大小来选择

set hive.auto.convert.sortmerge.join.bigtable.selection.policy = org.apache.hadoop.hive.ql.optimizer.TableSizeBasedBigTableSelectorForAutoSMJ;

org.apache.hadoop.hive.ql.optimizer.AvgPartitionSizeBasedBigTableSelectorForAutoSMJ (default)

org.apache.hadoop.hive.ql.optimizer.LeftmostBigTableSelectorForAutoSMJ

org.apache.hadoop.hive.ql.optimizer.TableSizeBasedBigTableSelectorForAutoSMJ

### map端group

是否在map端进行group操作

hive.map.aggr = true

在 Map 端进行聚合操作的条目数目

hive.groupby.mapaggr.checkinterval = 100000

### distribute by

distribute by 可以按照参照的列将hash值相等的数据分配到同一个reduce 可以为后续的查询加快检索速度

### 数据倾斜

数仓经常会出现某个reduce执行很久，原因就是大部分数据都被分配到了这个reduce中处理，出现长尾导致整个job进度变慢

首先尽量将count distinct 转成 group 之后在count 这样不会出现倾斜

如果某个表很小可以用上面的mapjoin解决

除此之外通过下面的参数，会将 group by 拆分成两次执行减少数据倾斜

hive.groupby.skewindata=true

以上是group时候出现的倾斜

join的时候也会出现倾斜

skew join 

hive会根据表中某个值的数量将其存放到内存中进行mapjoin操作

set hive.optimize.skewjoin = true

某个列的数量阈值，超过此阈值将会将包含此值的所有数据单独摘出来进行mapjoin

set hive.skewjoin.key = skew_key_threshold （default = 100000）

### 改善reduce端处理数据量

每个reduce处理的字节数默认1G，推荐

hive.exec.reducers.bytes.per.reducer

reduce最多执行数量 不推荐使用

mapred.reduce.tasks
