# Scala安装
___
* 解压scala

    `tar –zxvf scala-2.11.8.tgz`

* 配置环境变量
```
vi /etc/profile
export SCALA_HOME=/usr/tools/scala-2.11.8
export PATH=$PATH:$SCALA_HOME/bin
```
* 使环境变量生效

    `source /etc/profile`

* 检查安装成功：

    `scala –version`

    Scala code runner version 2.11.8 -- Copyright 2002-2016, LAMP/EPFL

* 拷贝到子节点上：
    `scp -r scala-2.11.8 root@jokeros2:/usr/tools`
    `scp -r scala-2.11.8 root@jokeros2:/usr/tools`
* 分别配置子节点的环境变量并使其生效.
# Spark安装
___
* 解压spark

    `tar –zxvf spark-1.6.1-bin-hadoop2.6.tgz`

* 配置环境变量

    ```
    export SPARK_HOME=/usr/tools/spark-1.6.1-bin-hadoop2.6
    export PATH=$PATH:$SPARK_HOME/bin
    ```
* 使环境变量生效

    `source /etc/profile`

* Spark配置

    **进入spark的conf目录**
```
    cd /usr/tools/spark-1.6.1-bin-hadoop2.6/conf
    cp  spark-env.sh.template spark-env.sh
    cp  log4j.properties.template log4j.properties
    cp slaves.template slaves
```
    **编辑spark-env.sh**
```
    export SCALA_HOME=/usr/tools/scala-2.11.8
    export JAVA_HOME=/usr/tools/jdk1.7.0_67
    export SPARK_WORKER_MEMORY=1G
    export HADOOP_CONF_DIR=/usr/tools/hadoop-2.6.4/etc/hadoop
```

**编辑slaves**
```
    node1
    node2
    node3
```
**spark拷贝到子节点上然后配置环境变量并使其生效。**
```
    scp -r spark-1.6.1-bin-hadoop2.6 root@jokeros2:/usr/tools
    scp -r spark-1.6.1-bin-hadoop2.6 root@jokeros3:/usr/tools
```
**进入主节点的sbin目录**

    `运行start-all.sh`

**主节点上Master Worker两个进程，子节点上Worker一个进程**
```
    http://node1:8080
    http://node1:4040/jobs
```

# Spark HistoryServer配置
```
    * 在spark-defaults.conf文件中添加以下内容
    spark.eventLog.enabled           true
    spark.eventLog.dir               hdfs://node1.bigdata:9000/historyserver
    spark.eventLog.compress          true
    * 在spark-env.sh文件中添加下面这句配置
    export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=4000 -Dspark.history.retainedApplications=5 -Dspark.history.fs.logDirectory=hdfs://node1.bigdata:9000/sparkhistory"
    * 在hdfs中创建目录/sparkhistory
    *启动
    sbin/start-history-server.sh
```

# Spark HA配置
**Spark-env.sh里面添加配置完成ha：**
```
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=hdm1:2181,hdm2:2181,hdm3:2181 -Dspark.deploy.zookeeper.dir=/usr/hadoop/spark-2.0.1-bin-hadoop2.6/zookeeper"
```
**启动**
```
    在主节点启动：sbin/start-all.sh
    在从节点启动：/sbin/start-master.sh
```
**注意**

`8080端口可能被占用`

# 集群测试

## 示例任务
