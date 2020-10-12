# 采用HortonWorks版本的99sql来衡量Insight集群Spark-SQL模块的性能

## HortonWorks版本99sql下载地址
`https://github.com/hortonworks/hive-testbench/tree/hdp3/spark-queries-tpcds`

## 物理机参数配置
物理机编号|	CPU	|内存	|核数	|硬盘
-|-|-|-|-|
1|	Xeon 6140（2）|	512GB|	18|	12*7T
2|	Xeon 6130（2）|	383GB|	18|	12*7T
3|	Xeon 6130（2）|	383GB|	18|	12*7T
4|	Xeon 6130（2）|	383GB|	18|	12*7T
5|	Xeon 6130（2）|	383GB|	18|	12*7T

## Insight集群环境
Namenode|	Datanode|	可用内存|	可用核数	|Insight版本
-|-|-|-|-|
1号物理机|	2-5号物理机|	1.13TB|	400|	V4旗舰版|

## 配置
**注意：**

    Hive 3.1.0默认开启ACID功能，且新建的内表默认是ACID表（Hive事务表）。但Spark目前还不支持Hive的ACID功能，因此无法读取ACID表的数据。
    为解决此问题，测试前，需对Spark及Hive进行先期配置，使Spark2.3.1能够读取Hive 3.1.0内部表。于Ambari界面修改配置如下
组件	|版本	|默认配置	|修改配置
-|-|-|-|
Spark	|2.3.1|	metastore.catalog.default：spark|	metastore.catalog.default：hive
Hive |3.1.0 | hive.strict.managed.tables=true | hive.strict.managed.tables=false
Hive |3.1.0 | hive.create.as.insert.only=true|hive.create.as.insert.only=false
Hive |3.1.0 |metastore.create.as.acid=true|metastore.create.as.acid=false

## 测试流程
**生成数据**

`hadoop jar target/tpcds-gen-1.1.jar -d /test/tpcds-500g/ -p 60 -s 500`
* -d: 数据目录
* -p: 并行数
* -s: 数据量（单位：GB）

**建表**
* 创建hive外部表

    `hive -f hive-mr-tpcds-external.sql`
* 创建hive内部表

    `hive -f hive-mr-tpcds-drop-create.sql`
* import非分区表数据

    `hive -f hive-mr-tpcds-insert.sql`
* 重新创建分区表且导入数据

    `hive -f hive-mr-tpcds-partition.sql`
* 调参执行99sql指令

    `./spark-sql --driver-memory 20G --executor-memory 30G --num-executors 32 --executor-cores 10 --master yarn --deploy-mode client --database tpcds --conf spark.sql.shuffle.partitions=960 --conf spark.memory.offHeap.enabled=true --conf spark.memory.offHeap.size=30g --conf spark.speculation=true --conf spark.sql.auto.repartition=true -f /hive/sql99.sql > /home/hive/sql1.log 2>&1`

## 100G与500G 数据量Spark-sql参数调优
*100G最优参数*
参数|	最优值
-|-|
driver-memory|	20G
executor-memory|	30G
num-executors	|16
executor-cores	|10
shuffle.partitions|	320
blockdev	|2048
auto.repartition|	TRUE
speculation	|TRUE
memory.offHeap.enabled	|TRUE
memory.offHeap.size	|10G
times	|1023s
*500G最优参数*
参数|	最优值
-|-|
driver-memory	|20G
executor-memory	|30G
num-executors	|32
executor-cores	|10
shuffle.partitions|	960
blockdev	|2048
auto.repartition	|TRUE
speculation	|TRUE
memory.offHeap.enabled|	TRUE
memory.offHeap.size	|30G
times	|1724s

## 调优结果
* 打印日志：tail -f log1.log
* 过滤99sql执行时间：cat sql1.log | grep Fetched | awk ‘{print $3}’ |grep –v INFO
## 影响较大的参数
*100G数据量*
参数|	影响程度|	最优值
-|-|-|
executor-memory|	高	|30g
num-executors|	高	|16
executor-cores|	高|	10
driver-memory|	高|	20g
shuffle.partitions|	高|	320
*500G数据量*
参数|	影响程度|	最优值
-|-|-|
executor-memory|	高	|30g
num-executors	|高|	32
executor-cores|	高|	10
driver-memory	|高	|20g
shuffle.partitions	|高	|640-960
## 其他参数设置
参数|	影响程度
-|-|
offHeap.size=30g（默认值：false）	|一般
speculation=true（默认值：false）	|一般
auto.repartition=true（默认值：false）|	一般
Blockdev=2048（默认值：256）|	一般
broadcast.blocksize=8m（默认值：4m）|	无积极影响
io.compression.codec=zstd（默认值：lz4）|	无积极影响
spark.reducer.maxSizeInFlight=128m	|无积极影响
spark.io.compression.zstd.level=2(默认值:1)|	无积极影响
spark.shuffle.file.buffer=512k(默认值:32k)|	无积极影响
spark.broadcast.checksum=false(默认值:true)|无积极影响
spark.shuffle.service.enabled=true(默认值:false)|无积极影响
spark.sql.autoBroadcastJoinThreshold=20M	|无积极影响
G1GC="-XX:+UseCompressedOops"	|无积极影响
Memory.fraction=0.7(默认值：0.6)	|无积极影响
storageFraction=0.4(默认值：0.5)	|无积极影响
Spark.sql.codegen=true(默认值：false)|无积极影响
driver.cores=5(默认值:：1)	|无积极影响

## 一般性结论
* shuffle.partitons设置为集群可用资源总核心数的2-3倍，提高shuffle read task的并行度，尽量避免计算最慢的task决定整个stage的时间，让运行快的task可以继续领取任务计算直至全部任务计算完毕。
* auto.repartition设置为true。每个stage运行时分区并不完全相同，使用此配置可优化计算后的分区数，避免分区数过大导致单个分区数据量过少，每个task运算分区数据时间过短，从而导致task频繁调度消耗过多时间
* 随着数据量的增大，若要保证99sql结果的最优，需要相应地提高内存及核数的使用量。对于相同数据量，内存及核数需保持在合适的范围内，才能得到最优结果。集群资源过小或过大均无法得到最优结果
* 推测执行、堆外内存、自动重新分区等参数对spark-sql性能均有提高
* 某些参数对部分sql是正面影响，对其它sql为负面影响（如：zxtd）

## 测试问题
* 1T数据量93条有结果，6条没有结果（44，51，58，65，69，75）
* 500G数据量，96条有结果，3条没有结果（58，65，75）
* 100G数据量，99条全部有数据，少数情况下，1条无数据。
* 建表阶段，重新创建分区表（Partition表）且导入数据时，需要手动逐条执行4条建表语句才能创建且导入数据（inventory表，web_sales表，store_sales表，catalog_sales表）。