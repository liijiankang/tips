# 初始步骤
    需要在集群每个节点上部署Alluxio二进制服务端包
# 配置Alluxio
    通过修改conf/alluxio-site.properties来配置Alluxio使用底层存储系统

## 基本配置
* 将hdfs根目录映射到Alluxio

    `hdfs://localhost:8020`
* 将hdfs子目录映射到Alluxio

    `hdfs://localhost:8020/alluxio/data`
## HDFS namenode HA模式
* 拷贝配置文件

    `hdfs-site.xml和core-site.xml文件需要放置在${ALLUXIO_HOME}/conf目录下`

    `或者在conf/alluxio-site.properties文件中将alluxio.underfs.hdfs.configuration指向hdfs-site.xml或者core-site.xml。例如：alluxio.underfs.hdfs.configuration=/path/to/hdfs/conf/core-site.xml:/path/to/hdfs/conf/hdfs-site.xml`
* 目录映射
    * 将HDFS的根目录映射到Alluxio

        `hdfs://nameservice/`
    * 把HDFS目录/alluxio/data映射到Alluxio
        `hdfs://nameservice/alluxio/data`

        `例如：alluxio.master.mount.table.root.ufs=hdfs://nameservice/`
## 用户/权限映射
**为了确保文件/目录的权限信息，即HDFS上的用户，组和访问模式，与Alluxio一致，(例如，在Alluxio中被用户Foo创建的文件在HDFS中也以Foo作为用户持久化)，用户需要以以下方式启动:**
* HDFS超级用户。即，使用启动HDFS namenode进程的同一用户也启动Alluxio master和worker进程。也就是说，使用与启动HDFS的namenode进程相同的用户名启动Alluxio master和worker进程。
* HDFS超级用户组的成员。编辑HDFS配置文件hdfs-site.xml并检查配置属性dfs.permissions.superusergroup的值。如果使用组（例如，“hdfs”）设置此属性，则将用户添加到此组（“hdfs”）以启动Alluxio进程（例如，“alluxio”）;如果未设置此属性，请将一个组添加到此属性，其中Alluxio运行用户是此新添加组的成员。
## 安全认证模式下的HDFS
**Alluxio支持安全认证模式下的HDFS作为底层文件系统，通过Kerberos认证。**
### Kerberos配置
* Hadoop

    `export HADOOP_OPTS="$HADOOP_OPTS -Djava.security.krb5.realm=<YOUR_KERBEROS_REALM> -Djava.security.krb5.kdc=<YOUR_KERBEROS_KDC_ADDRESS>"`
* Spark

    `SPARK_JAVA_OPTS+=" -Djava.security.krb5.realm=<YOUR_KERBEROS_REALM> -Djava.security.krb5.kdc=<YOUR_KERBEROS_KDC_ADDRESS>"`
* Alluxio Shell

    `ALLUXIO_JAVA_OPTS+=" -Djava.security.krb5.realm=<YOUR_KERBEROS_REALM> -Djava.security.krb5.kdc=<YOUR_KERBEROS_KDC_ADDRESS>"`

### Alluxio服务器Kerberos认证 
**在alluxio-site.properties文件配置下面的Alluxio属性：**
```
    alluxio.master.keytab.file=<YOUR_HDFS_KEYTAB_FILE_PATH>
    alluxio.master.principal=hdfs/<_HOST>@<REALM>
    alluxio.worker.keytab.file=<YOUR_HDFS_KEYTAB_FILE_PATH>
    alluxio.worker.principal=hdfs/<_HOST>@<REALM>
```
# 使用HDFS在本地运行Alluxio
```
    $ ./bin/alluxio format
    $ ./bin/alluxio-start.sh local
    该命令应当会本地启动一个Alluxio master和一个Alluxio worker，可以在浏览器中访问http://localhost:19999查看master Web UI。
```