<h1><center>Ambari后台手动关闭Kerberos</center></h1>
<h3><center>高危操作</center></h3>

## 备份元数据

* 需要备份的表
  
  *  ambari.servicedesiredstate
  *  ambari.hostcomponentdesiredstate
  *  ambari.hostcomponentstate
  *  ambari.servicecomponentdesiredstate
  *  ambari.clusterservices
  *  ambari.clusters
  *  ambari.kkp_mapping_service
  *  ambari.kerberos_keytab_principal
  *  ambari.kerberos_principal
  *  ambari.kerberos_keytab
  *  ambari.clusterconfig

## 修改数据库

* 页面上启动/近用Kerberos相关表

    `update ambari.clusters set security_type='NONE';`

    NONE:启用Kerberos按钮可用

    KERBEROS:禁用Kerberos按钮可用

* 删除服务列表中显示的Kerberos组件
  
  `delete from ambari.servicedesiredstate where service_name='KERBEROS';`

  `delete from ambari.hostcomponentdesiredstate where service_name='KERBEROS';`

  `delete from ambari.hostcomponentstate where service_name='KERBEROS';`

  `delete from ambari.servicecomponentdesiredstate where service_name='KERBEROS';`

  `delete from ambari.clusterservices where service_name='KERBEROS';`

* 禁用集群Kerberos配置
  
  **将ambari.clusterconfig表中type_name="cluster-env"的行中config_data字段中security_enabled的值由true 改为false**

* 重启ambari-server即可

## 回滚

    恢复数据库，重启ambari-server