# 数据库
## 创建数据库
`create database db;`
`create schema db;`
## 删除数据库
`drop database db;`
## 查看数据库
`show databases;`
## 应用数据库
`use db;`
# 表
## 建表
### 先建表，后加载数据
**step1:建表**
```sql
    create table user_logs_large(
        user_name string,
        action_type string,
        ip_addr string
    )
    row format delimited
    fields terminated by '\t'
    stored as textfile;
```
**step2:导入数据**
```sql
    load data inpath '/user-logs-large.txt' overwrite into table user_logs_large;
```
### 建表同时加载数据
```sql
    create external table if not exists province_user(
        user_id string,
        user_name string,
        province_type int,
        gender_type string,
        province_name string
    )
    row format delimited
    fields terminated by ';'
    location '/external/provinceuser';
```
### 建表同时使用serder正则表达式的形式加载数据
```sql
    create external table access_log(
        host STRING,
        identity STRING,
        auser STRING,
        access_time STRING,
        request STRING,
        status STRING,
        size STRING,
        referer STRING,
        agent STRING
    )
    row format serde 'org.apache.hadoop.hive.serde2.RegexSerDe'
    WITH SERDEPROPERTIES (
    "input.regex" = "([^ ]*) ([^ ]*) ([^ ]*) (-|\\[[^\\]]*\\]) ([^ \"]*|\"[^\"]*\") (-|[0-9]*) (-|[0-9]*)(?: ([^ \"]*|\"[^\"]*\") ([^ \"]*|\"[^\"]*\"))?"
    )
    STORED AS TEXTFILE
    location '/external/access_log_20160429';
```
## 删除表
`drop table province_user;`
## 查询表
`select * from access_log;`
## 复制表
`create table access_log_copy like access_log;`
## 克隆表
`create table access_log_clone as select * from access_log`
## 将表中一部分数据单独存储为别的表
```sql
    create table access_log_s
    stored as orc
    as
    select * 
    from access_log
    where host='27.19.74.143';
```
```sql
    create table access_log_sq
    stored as sequencefile
    as
    select * 
    from access_log
    where host='27.19.74.143'
```
## 查看表结构
`describe access_log`

`describe extended access_log`

`describe formatted access_log`

<html lang="en">
<head>
<meta charset="UTF-8" />

<style type="text/css">
    h1{
        position: relative;
        color: black;
    }
    h1:before{
        content: attr(text);
        position: absolute;
        z-index: 10;
        color:red;
        -webkit-mask:linear-gradient(to left, red, transparent );
    }
</style>
</head>

<body>
    <h1 align="right" text="李建康-inspur">李建康-inspur</h1>
</body>

</html>