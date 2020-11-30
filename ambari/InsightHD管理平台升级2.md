<h1><center>InsightHD管理平台小版本升级手册</center></h1>

<center><h2><font  color="#FF0000">高危操作</font></h2></center>
<center><h4><font  color="#FF0000">升级到新版本InsightHD并验证集群功能正常后才能拆除旧集群的管理节点</font></h4></center>

### ***此手册适用于在新管理节点部署新版本InsightHD替换旧管理节点的老版本InsightHD***

* 安装包准备
  
    将新版本的insight安装包'InsightV4.6.5-release.tar'上传到新的manager节点/opt/目录下并解压。

* 管理节点部署
    * 配置deploy.xlsm文件
  
        *1. nodes表中只填写manager的信息,NTPROLE栏中选择ntp_server1*

        *2.conf表中控制节点填写新的manager节点信息，MySQL配置中只填写db_master为新manager节点*

    * 执行一键部署程序

* 时间同步
  
  * 所有节点同新管理节点时间同步(在所有节点执行)
    
    `ntpdate xxx.xxx.xxx.xxx`

  * 修改所有节点/etc/ntp.conf文件，用新的管理节点域名替换旧管理节点的域名

* 元数据迁移（非必须）
  * 登陆旧集群主MySQL数据库，备份ambari，dataspace，hive，hue，insight_ml，ke，oozie，mysql，ranger，ranger_audit，rangerkms，superset。并将备份出的sql文件拷贝的新的manager节点上。（以ambari数据库为例）
  
    `1. mysqldump -uroot -pbigdata123 ambari>/opt/ambari.sql`

    `2. scp /opt/ambari.sql root@xxx.xxx.xxx.xxx:/opt`

  * 登陆新manager节点搭建的数据库，导入数据（以导入ambari数据库为例）：

    `1.use ambari;`

    `2.source /opt/ambari.sql;`

  * 从MySQL备份配置,登陆原集群从数据库：

    `1. stop slave;`

    `2. CHANGE MASTER TO MASTER_HOST='xxx.xxx.xxx.xxx',MASTER_USER='replication',MASTER_PASSWORD='bigdata123',MASTER_LOG_FILE='mysql-bin.xxxx',MASTER_LOG_POS=xxx;`

    *MASTER_HOST:填写新管理节点的IP地址*

    **登陆原管理节点的主MySQL数据库，执行show master status;命令查看File和Position对应的值**

    *MASTER_LOG_FILE：File对应的值*

    *MASTER_LOG_POS：Position对应的值*

    `3. start slave;`

    `4. 检测同步状态：show slave status\G`

* 免密配置
  
  * 完善所有节点hosts信息
  
  * 在新的管理节点生成密钥

    `ssh-keygen`

  * 拷贝到其余节点

    `ssh-copy-id -i .ssh/id_rsa.pub root@xxx.xxx.xxx.xxx`

* 更新旧集群ambari-server
  
    * 停止旧版本的ambari-server 
    * 修改所有节点/etc/ambari-agent/conf/ambari-agent.ini文件，将文件中hostname改为新管理节点的域名
    * 重启新管理节点ambari-server

