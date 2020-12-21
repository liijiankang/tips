
### 集群拓扑

<div class="center">
    <img src="../picture/77.png" width=700 height=250 />
</div>

### 机器配置

* 计算节点

    <div class="indentation_1">

    |内存|AEP|
    |:-:|:-:|
    |384G|512G

    </div>
    

* 存储节点

    <div class="indentation_1">

    |内存|磁盘|
    |:-:|:-:|
    |128G|1T*6|

    </div>

* 管理节点

    <div class="indentation_1">

    |内存|
    |:-:|
    |256G|

    </div>

### 对比测试

</br>

**资源配置**


<div class="indentation_1">

|yarn|alluxio|
|:-:|:-:|
|128G|256G|
</div>

</br>

* Alluxio空间能够缓存所有数据(300G数据、500G数据)

    * 300G数据

        存储节点能够缓存所有数据时，此时spark with alluxio是否有性能提升

    * 600G数据

        存储节点不能缓存所有数据，Alluxio可以缓存所有数据，spark with alluxio性能提升是否明显

* Alluxio空间不足（1T数据）

    Alluxio不能缓存所有数据时，不同的cache策略对性能的影响

* Alluxio+AEP
  * 2T数据，可以缓存所有数据

    * 单层存储

        AEP作为内存使用

        AEP作为SSD使用，Alluxio单层存储配置两种不同的存储介质RAM、SSD

    * 多级存储

        AEP作为SSD使用，第一层配置RAM，第二层配置SSD

  * 4T数据 ，不能缓存所有数据
    







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