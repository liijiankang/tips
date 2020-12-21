<h1><center>大数据安全防护-Kerberos</center></h1>

&emsp;&emsp;大数据作为企业转型升级的重要支撑性技术，在数据采集、加工、存储、聚合、交换、应用等诸多环节存在安全防护需求。随着数据驱动创新战略的提出，数据已成为一种重要的生产要素，数据安全程度将对企业转型升级的成败产生重大的影响。企业在使用信息平台进行管理和对外提供服务时，要制定技术和管理措施，推进数据全生命周期过程的安全防护，提升数据防窃取、防丢失的能力，为成功实现数字化转型提供技术支撑。IBM研究报告数据泄露事件给企业造成的平均成本为 386 万美元，智能技术将数据泄露成本降低了一半。


![example](picture/1.png)

<h5><center>Kerberos:希腊神话中的人物,是一条守护地狱之门的三头保卫神犬</center></h5>

<br/>
<br/>



>&emsp;&emsp;Kerberos是一种网络认证协议，其设计目标是通过密钥系统为客户机 / 服务器应用程序提供强大的**认证服务**。该认证过程的实现**不依赖于主机操作系统的认证**，**无需基于主机地址的信任**，**不要求网络上所有主机的物理安全**，并假定网络上**传送的数据包可以被任意地读取、修改和插入数据**。在以上情况下， Kerberos 作为一种可信任的第三方认证服务，是通过传统的密码技术（如：共享密钥）执行认证服务的。

<br/>
<br/>

<h2>概念</h2>

* Authentication：认证。验证身份的过程，证明我是我。
* Authorization：授权。验证是否有权访问的过程。虽然是公司的一员，但是没有访问公司某些信息的权限。

*&emsp;&emsp;认证和授权通常相互结合使用，只有认证没有授权，无法做到细粒度的权限控制；只有授权没有认证无法避免仿冒问题，比如给张三用户赋予了访问HDFS中某些目录的权限，如果不用身份认证，那么别的用户可以仿冒张三用户去访问这些数据。*

* Principal：Kerberos主体。主体是 KDC 可以为其分配票证的唯一标识，可以是用户或者服务，约定主体名称分为三个部分：主名称、实例和域名。

    <div class="indentation_1">

     principal|角色
     |:-:|:-:|
    |nm/sizu05@BIGDATA|服务|
    |nm/sizu06@BIGDATA|服务|
    |hbase-l3kerberos@BIGDATA|用户|
    |hdfs-l3kerberos@BIGDATA|用户|
    </div>

*&emsp;&emsp;服务主体代表的是集群中的服务，用户主体代表的是使用集群服务的用户。*

*&emsp;&emsp;错误示例：*

```
    FLUME -> KAFKA -> SPARK 
    FLUME的jaas文件中配置的是FLUME服务的pricipal.
    KAFKA的jaas文件中配置的是KAFKA服务的principal.
    SPARK中用的是Spark服务的principal.
```

*&emsp;&emsp;正确示例：*

```
    FLUME -> KAFKA -> SPARK 
    整个流程中使用一个用户principal，并给用户赋予KAFKA,HDFS，YARN等服务相应权限。
```

* KDC
* keytab
* krb5.conf
* kdc.conf
* default_realm
* renew_lifetime
* ticket_lifetime


<style>
    .indentation_1 {
    width: auto;
    display: table;
    margin-left: 1cm;
    margin-right: auto;
    }
    .indentation_2 {
    width: auto;
    display: table;
    margin-left: 1.5cm;
    margin-right: auto;
    }
    .indentation_3 {
    width: auto;
    display: table;
    margin-left: 3cm;
    margin-right: auto;
    }
    .center {
    width: auto;
    display: table;
    margin-left: auto;
    margin-right: auto;
    }
</style>

