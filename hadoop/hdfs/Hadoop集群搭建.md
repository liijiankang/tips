# 配置主机名及hosts文件
```
    略
```
# 配置免密登陆
```
    1.生成密钥文件
        ssh-keygen -t rsa
    2.manager到其余节点免密登陆 
        ssh-cory-id -i ~/.ssh/id_rsa.pub root@IP
```
# 安装jdk
```
    略
```
# 下载并解压Hadoop安装包
```
    略
```
# 配置Hadoop
**HDFS**
* hadoop-env.sh
```
# The java implementation to use.
//添加JAVA_HOME
export JAVA_HOME=/opt/java/jdk1.8.0_121
# The jsvc implementation to use. Jsvc is required to run secure datanodes
```
* core-site.xml
```
    <property>
                <name>fs.defaultFS</name>
                <value>hdfs://node1:9000</value>
        </property>
        <property>
                <name>io.file.buffer.size</name>
                <value>131072</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/usr/temp</value>
        </property>
        <property>
                <name>hadoop.proxyuser.root.hosts</name>
                <value>*</value>
        </property>
        <property>
                <name>hadoop.proxyuser.root.groups</name>
                <value>*</value>
        </property>
```
* hdfs-site.xml
```
    <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>node1:9001</value>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/usr/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/usr/dfs/data</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>
        <property>
                <name>dfs.webhdfs.enabled</name>
                <value>true</value>
        </property>
        <property>
                <name>dfs.permissions</name>
                <value>false</value>
        </property>
        <property>
                <name>dfs.web.ugi</name>
                <value> supergroup</value>
        </property>
```
* mapred-site.xml
```
    <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>node1:10020</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>node1:19888</value>
        </property>
```
**YARN**
* yarn-env.sh
```
    # some Java parameters
    export JAVA_HOME=/opt/java/jdk1.8.0_121
```
* yarn-site.xml
```
    <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>node1:8032</value>
        </property>
        <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value>node1:8030</value>
        </property>
        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>node1:8031</value>
        </property>
        <property>
                <name>yarn.resourcemanager.admin.address</name>
                <value>node1:8033</value>
        </property>
        <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value>node1:8088</value>
        </property>
```
**slaves**
```
    node1
    node2
    node3
```
**拷贝Hadoop安装目录到别的节点上**
# 格式化namenode
```
    bin/hdfs namenode -format <cluster_name>
```
# 启动集群
* 启动namenode

    `bin/hdfs --daemon start namenode`

* 启动datanode

    `bin/hdfs --daemon start datanode`

* 启动resourcemanager

    `yarn --daemon start resourcemanager`

* 启动nodemanager

    `bin/yarn --daemon start nodemanager`
```
    sbin/start-dfs.sh
    sbin/start-yarn.sh
```    
# 远程调试

`export HADOOP_NAMENODE_OPTS="-agentlib:jdwp=transport=dt_socket,address=8888,server=y,suspend=y"`
# 开启HA
**namenode**
* core-site.xml
```
    <configuration>
        <property> 
                <name>fs.defaultFS</name>
                <value>hdfs://ns</value>
        </property>
        <property>
                <name>ha.zookeeper.quorum</name>
                <value>zk1:2181,zk2:2181,zk3:2181</value>
        </property>
        <property>
                <name>io.file.buffer.size</name>
                <value>131072</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/usr/temp</value>
        </property>
        <property>
                <name>hadoop.proxyuser.root.hosts</name>
                <value>*</value>
        </property>
        <property>
                <name>hadoop.proxyuser.root.groups</name>
                <value>*</value>
        </property>
</configuration>

```
* hdfs-site.xml
```
    <configuration>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/usr/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/usr/dfs/data</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>3</value>
        </property>
        <property>
                <name>dfs.webhdfs.enabled</name>
                <value>true</value>
        </property>
        <property>
                <name>dfs.permissions</name>
                <value>false</value>
        </property>
        <property>
                <name>dfs.web.ugi</name>
                <value>supergroup</value>
        </property>
        <property>
                <name>dfs.nameservices</name>
                <value>ns</value>
        </property>
        <property>
                <name>dfs.ha.namenodes.ns</name>
                <value>nn1,nn2</value>
        </property>
        <property>
                <name>dfs.namenode.rpc-address.ns.nn1</name>
                <value>node1:9000</value>
        </property>
        <property>
                <name>dfs.namenode.http-address.ns.nn1</name>
                <value>node1:9870</value>
        </property>
        <property>
                <name>dfs.namenode.rpc-address.ns.nn2</name>
                <value>node2:9000</value>
        </property>
        <property>
                <name>dfs.namenode.http-address.ns.nn2</name>
                <value>node2:9870</value>
        </property>
        <property>
                <name>dfs.namenode.shared.edits.dir</name>
                <value>qjournal://node1:8485;node2:8485;node3:8485/ns</value>
        </property>
        <property>
                <name>dfs.ha.automatic-failover.enabled</name>
                <value>true</value>
        </property>
        <property>
                <name>dfs.client.failover.proxy.provider.ns</name>
                <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailOverProxyProvider</value>
        </property>
        <property>
                <name>dfs.ha.fencing.methods</name>
                <value>sshfence(root:2022)</value>
        </property>

        <property>
                <name>dfs.ha.fencing.ssh.private-key-files</name>
                <value>/root/.ssh/id_rsa</value>
        </property>

```
**resourcemanager**
* yarn-site.xml
```
    <configuration>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
                <name>yarn.resourcemanager.ha.enabled</name>
                <value>true</value>
        </property>
        <property>
                <name>yarn.resourcemanager.cluster-id</name>
                <value>ljk</value>
        </property>
        <property>
                <name>yarn.resourcemanager.ha.rm-ids</name>
                <value>rm1,rm2</value>
        </property>
        <property>
                <name>yarn.resourcemanager.hostname.rm1</name>
                <value>node1</value>
        </property>
        <property>
                <name>yarn.resourcemanager.hostname.rm2</name>
                <value>node2</value>
        </property>
        <property>
                <name>yarn.resourcemanager.recovery.enabled</name>
                <value>true</value>
        </property>
        <property>
              <name>yarn.nodemanager.local-dirs</name>
              <value>/usr/yarn/local</value>
        </property>
        <property>
              <name>yarn.resourcemanager.store.class</name>
              <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
        </property>
        <property>
              <name>yarn.resourcemanager.zk-address</name>
              <value>zk1:2181,zk2:2181,zk3:2181</value>
        </property>
</configuration>

```
**启动顺序**
```
    1.启动zk
    2.启动jn集群
        hadoop-daemons.sh start journalnode
    3.格式化zkfc,在zookeeper中创建ha节点
        hdfs zkfc -formatZK
    4.格式化namenode
        在active节点上执行： hdfs namenode -format
        如果是非HA->HA执行：bin/hdfs namenode -initializeSharedEdits
    5.启动namenode
        在active节点执行：hadoop-daemon.sh start namenode
        在standby节点上执行：hdfs namenode -bootstrapStandby hadoop-daemon.sh start namenode
    6.启动datanode
    7.启动zkfc
        在每个namenode上启动：hadoop-daemon.sh start zkfc
    8.启动yarn
        在active节点启动：sbin/start-yarn.sh
        在backup节点启动：yarn-daemon.sh start resourcemanager
```