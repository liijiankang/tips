# 下载并解压tar包
# 配置环境变量
```
    export ZOOKEEPER=/opt/apache-zookeeper-3.5.8-bin/
    export PATH=$PATH:$ZOOKEEPER/bin
```
# 配置zoo.cfg文件
```
    * 配置数据目录
    dataDir=$PATH/data
    *添加节点
    server.1=node1:2888:3888
    server.2=node2:2888:3888
    server.3=node3:2888:3888
```
# 将文件分发到别的node上
# 在所有node上创建myid文件,myid文件中保存自己的id号
`touch $PATH/data/myid`
# 启动
`zkServer.sh start`