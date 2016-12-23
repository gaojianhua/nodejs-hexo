title: 乡村销客离线计算--DataX 测试
date: 2016-09-25 21:22:11
tags:
- 乡村销客
- 大数据
- ETL
categories: ambari
---

# 乡村销客
__ 乡村销客官网 __: http://www.vilsale.com 
乡村销客是面向化肥行业的企业互联网营销工具。通过“移动应用+云计算+应用市场”的互联网领先技术，帮助化肥生产销售企业快速实现  
<!-- more -->
1. __ 移动化市场营销及客户拜访，__  
1. __ 解决调度发运响应不畅  __  
1. __ 客户账户对账不准等问题， __  
1. __ 帮助肥料企业营销及客户管理 __”。 

乡村销客基于软件即服务的互联网理念，创建国内第一个专注__ 肥料行业的SAAS平台 __，为肥料企业打造性价比最高的企业互联网营销工具。
 
-----------这是广告结束的分割线--------------------------


# ** datax ** 


dataxWIKI : https://github.com/alibaba/DataX/wiki/DataX-Introduction

hdfswriter : https://github.com/alibaba/DataX/blob/master/hdfswriter/doc/hdfswriter.md


# hive

 1 进入 amb5 或者amb6

 2 执行 hive 命令 启动 hive

 3 写sql语句
 
 ```sql
show databases;

show tables;

create database IF NOT EXISTS hdfswriter;

use hdfswriter;

create table text_table(
PERSONNEL_CODE  STRING,
PERSONNEL_NAME  STRING,
REGION_CODE  STRING,
REGION_NAME  STRING,
REGION_TREECODE  STRING
)
row format delimited
fields terminated by "\t"
STORED AS TEXTFILE;


desc text_table ; 

insert overwrite table text_table select * from text_table where text_table.personnel_code = '000437';

insert overwrite table t_table1 select * from t_table1 where XXXX; 其中xxx是你需要保留的数据的查询条件。 如果清空表，如下： insert overwrite table t_table1 select * from t ...
```

# mysql--->hive


## mysql-hdfs.json 配置案例

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "mysqlreader", 
                    "parameter": {
                        "username": "e***",
                        "password": "e***",
                         
                        "connection": [
                            {
                                "querySql":["SELECT p.PERSONNEL_CODE,p.PERSONNEL_NAME,reg.REGION_CODE,reg.REGION_NAME,reg.REGION_TREECODE FROM dossier_personnel p  JOIN re_user_regoin r ON r.user_id = p.PERSONNEL_ID JOIN        bd_region reg ON reg.REGION_ID = r.REGION_ID WHERE r.RUG_STATE =0 "],
                                "jdbcUrl": ["jdbc:mysql://192.168.0.57:3306/vil"]
                            }
                        ]
                    }
                }, 
                "writer": {
                    "name": "hdfswriter", 
                    "parameter": {
                        "column": [
                            {
                                "name": "PERSONNEL_CODE",
                                "type": "STRING"
                            }, {
                                "name": "PERSONNEL_NAME",
                                "type": "STRING"
                            }, {
                                "name": "REGION_CODE",
                                "type": "STRING"
                            }, {
                                "name": "REGION_NAME",
                                "type": "STRING"
                            }, {
                                "name": "REGION_TREECODE",
                                "type": "STRING"
                            }
                        ], 
                         
                        "defaultFS": "hdfs://172.17.0.5:8020", 
                        "fieldDelimiter": "\t", 
                        "fileName": "text_table1", 
                        "fileType": "text", 
                        "path": "/apps/hive/warehouse/hdfswriter.db/text_table", 
                        "writeMode": "append"
                    }
                }
            }
        ], 
        "setting": {
            "speed": {
                "channel": "2"
            }
        }
    }
}
```

## 参数说明

* __defaultFS__
描述：Hadoop hdfs文件系统namenode节点地址。格式：hdfs://ip:端口；例如：hdfs://127.0.0.1:9000


* __fileType__
描述：文件的类型，目前只支持用户配置为"text"或"orc"。 
text表示textfile文件格式
orc表示orcfile文件格式



* __path__

描述：存储到Hadoop hdfs文件系统的路径信息，HdfsWriter会根据并发配置在Path目录下写入多个文件。为与hive表关联，请填写hive表在hdfs上的存储路径。例：Hive上设置的数据仓库的存储路径为：/user/hive/warehouse/ ，已建立数据库：test，表：hello；则对应的存储路径为：/user/hive/warehouse/test.db/hello 


* __writeMode__
描述：hdfswriter写入前数据清理处理模式： 
append，写入前不做任何处理，DataX hdfswriter直接使用filename写入，并保证文件名不冲突。
nonConflict，如果目录下有fileName前缀的文件，直接报错。


* __compress__
描述：hdfs文件压缩类型，默认不填写意味着没有压缩。其中：text类型文件支持压缩类型有gzip、bzip2;orc类型文件支持的压缩类型有NONE、SNAPPY（需要用户安装SnappyCodec）。


## 类型转换

目前 HdfsWriter 支持大部分 Hive 类型，请注意检查你的类型。

下面列出 HdfsWriter 针对 Hive 数据类型转换列表:

| DataX 内部类型| HIVE 数据类型    |
| -------- | -----  |
| Long     |TINYINT,SMALLINT,INT,BIGINT |
| Double   |FLOAT,DOUBLE |
| String   |STRING,VARCHAR,CHAR |
| Boolean  |BOOLEAN |
| Date     |DATE,TIMESTAMP |




# 步骤
 1 建表
 2 配置 json
 3 执行导入
 4 hive 查看测试数据

# 定时任务

参考
https://github.com/alibaba/DataX/wiki/%E9%85%8D%E7%BD%AE%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1%EF%BC%88Linux%E7%8E%AF%E5%A2%83%EF%BC%89

会进入已有crontab文件编辑界面，继续增加定时任务即可，本示例增加以下内容,并保存

```bash
0,10,20,30,40,50 * * * *  python /opt/local/datax/bin/datax.py  /opt/local/datax/job/mysql.json  >>/opt/logs/dataxlog.`date +\%Y\%m\%d\%H\%M\%S`  2>&1

```
```bash
python /opt/local/datax/bin/datax.py  /opt/local/datax/job/mysql.json 

```


/*参考资料*/
http://robingao.xyz/2016/09/25/dataxTest/
