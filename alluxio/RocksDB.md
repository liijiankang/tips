# 1.0 vs 2.0
    alluxio2.0一个namespace可以支持10亿个文件
    alluxio1.0可以支持2亿个文件，但是jvm需要占用200G的堆内存
# RocksDB
    热数据存储在内存中，冷数据存储在磁盘上。
## 配置
*conf/alluxio-site.properties*

* alluxio.master.metastore
* alluxio.master.metastore.dir
* alluxio.master.metastore.inode.cache.max.size ：1千万个文件大约需要10G内存

    预估工作集的大小，然后设置master的jvm参数。例如：工作集有5百万个文件，可以将master的jvm内存设置为5G，ALLUXIO_MASTER_JAVA_OPTS+=" -Xmx5G"。如果内存充足，可以将其设置为31G，ALLUXIO_MASTER_JAVA_OPTS+=" -Xmx31G"，可以满足大多数工作负载，并且避免在堆达到32GB时切换到64位指针的需要。此配置在conf/alluxio-env.sh中。
## 磁盘存储
    每个文件存储在内存占用大约占用1KB的空间，但是存储在磁盘上约占500byte的空间，所以如果在磁盘上存储10亿个文件，HDD或者SSD容量应该大于500G.