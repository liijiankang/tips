<h1><center>InsightHD管理平台小版本升级手册</center></h1>
<center><h2><font  color="#FF0000">高危操作</font></h2></center>

### ***此手册适用于在原有管理节点升级InsightHD管理平台***

<h2>目录</h2>

* 升级前
	 * ambari-server备份
	 * 元数据备份
	 * 准备4.6.5版本的镜像文件
* 升级
	 * 卸载ambari-server
	 * 升级yum源
	 * 下载并配置ambari-server
* 升级后
	 * 登陆密码替换
	 * 启动ambari-server
* 版本回退
	 * 卸载ambari-server
	 * 回退yum源
	 * 下载ambari-server
	 * 元数据恢复
	 * 配置并重启ambari-server
* 关闭Kerberos
    * 删除Kerberos服务
    * 更新数据库
    * 重启ambari-server

<h2>1.升级前</h2>

## &emsp;1.1.ambari-server配置备份

* 需要备份的目录
  
    * /etc/ambari-server/conf/ 

    * /var/lib/ambari-server/resources/common-services/

    * /var/lib/ambari-server/resources/stacks/

* 备份
  
  `cp -r /etc/ambari-server/conf/ /etc/ambari-server/conf.bak`

  `cp -r /var/lib/ambari-server/resources/common-services/ /var/lib/ambari-server/resources/common-services.bak`

  `cp -r /var/lib/ambari-server/resources/stacks/ /var/lib/ambari-server/resources/stacks.bak`

## &emsp;1.2.元数据备份
* 需要备份的数据库：
    * ambari
    * dataspace
    * hive
    * hue
    * insight_ml
    * ke
    * oozie
    * mysql
    * ranger
    * ranger_audit
    * rangerkms
    * superset
* 备份(以ambari数据库为例子，用户：root,密码：bigdata123):
  
  `mysqldump -uroot -pbigdata123 ambari>/opt/ambari.sql`

## &emsp;1.3.准备4.6.5版本的镜像文件
        将4.6.5版本的镜像文件CentOS7-Sirius-Incloud_xxx_4.6.5_xxx.iso上传到集群manager节点/opt/InsightHdInstall/repository/目录下

<h2>2.升级</h2>

* 卸载ambari-server
  
  `yum remove ambari-server`

* 升级yum源
    * 卸载低版本的yum源
  
        `umount /var/www/html/InsightHD/`

    * 清理yum缓存
  
        `yum clean all && rm -rf /var/cache/yum`

    * 挂载新版本的yum源
  
        `mount -o loop -t iso9660  CentOS7-Sirius-Incloud_xxx_4.6.5_xxx.iso /var/www/html/InsightHD/`
  
* 下载并配置ambari-server
  * 下载ambari-server
  
    `yum install ambari-server`

  * 配置ambari-server
  
    `ambari-server setup`
  

>请根据如下样例填写交互信息！

```
[root@sizu05 keytabs]# ambari-server setup
Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
SELinux status is 'disabled'
Customize user account for ambari-server daemon [y/n] (n)? n
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
Do you want to change Oracle JDK [y/n] (n)? y
[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Custom JDK
==============================================================================
Enter choice (1): 2
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /usr/jdk64/java/
Validating JDK on Ambari Server...done.
Check JDK version for Ambari Server...
JDK version found: 8
Minimum JDK version is 8 for Ambari. Skipping to setup different JDK for Ambari Server.
Checking GPL software agreement...
GPL License for LZO: https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html
Enable Ambari Server to download and install GPL Licensed LZO packages [y/n] (n)? n
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? y
Configuring database...
==============================================================================
Choose one of the following options:
[1] - PostgreSQL (Embedded)
[2] - Oracle
[3] - MySQL / MariaDB
[4] - PostgreSQL
[5] - Microsoft SQL Server (Tech Preview)
[6] - SQL Anywhere
[7] - BDB
==============================================================================
Enter choice (3): 3
Hostname (sizu05):sizu05
Port (3306):3306
Database name (ambari):ambari
Username (ambari):ambari
Enter Database Password (bigdata123):bigdata123
Configuring ambari database...
Configuring remote database connection properties...
WARNING: Before starting Ambari Server, you must run the following DDL against the database to create the schema: /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql
Proceed with configuring remote database connection properties [y/n] (y)?
Extracting system views...
.....
Ambari repo file doesn't contain latest json url, skipping repoinfos modification
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully.
```

<h2>3.升级后</h2>

* 登陆密码替换
  * 登陆后台数据库：
  
    `mysql -uroot -pbigdata123`

  * 重置密码(admin@123)
  
    `update ambari.user_authentication set authentication_key='YWRtaW5AMTIz' where user_id=(select user_id from ambari.users where user_name='admin');`

* 重启ambari-server
  
    `ambari-server restart `


<h2>4.版本回退</h2>

* 卸载ambari-server
  
  `yum remove ambari-server`


* 回退yum源
  
    * 卸载v4.6.5镜像文件: 
    
        `umount /var/www/html/InsightHD/  `
    
    * 挂载v4.6.0镜像文件: 
  
        `mount -o loop -t iso9660  CentOS7-Sirius-Incloud_xxx_4.6.0_xxx.iso /var/www/html/InsightHD/`
  
    * 清除yum缓存: 
  
        `yum clean all && rm -rf /var/cache/yum`

* 下载ambari-server
  
  `yum install ambari-server`


* 元数数据恢复,以恢复ambari数据库为例:

    * 登陆后台数据库：
  
        `mysql -uroot -pbigdata123`
  
    * 删除ambari数据库:
  
        `drop database ambari;`
  
    * 导入备份的ambari数据库：
  
        `source /opt/ambari.sql;`

* 配置并重启ambari-server
  * 配置ambari-server
    
    配置方法和升级步骤中的配置方法相同

  * 重启ambari-server

    `ambari-server start`
<h2>5.关闭Kerberos</h2>

* 修改元数据
  `delete from ambari.clusterservices where  service_name='KERBEROS';`
  `update ambari.clusters set security_type='NONE';`
* 重启ambari-server 



