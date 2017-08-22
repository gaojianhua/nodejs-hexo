title: sparkt-调优
date: 2017-04-07 21:21:33
tags:
- 乡村销客
- 大数据
categories: spark
---

# spark调优初体验-案例

用户行为数据

<!-- more -->

程序自动运行的模拟数据

数据特征
文本格式
约17GB 400w行,每行1000 列



spark shell 

export
 yarn_conf_dir=/home/hadoop/hadoop-2.7.3/etc/hadoop

 bin/spark-shell --master yarn --deploy-mode client 

 bin/spark-shell --mater yarn-client (从2.1.0开始,不建议使用该方式)


 ## 优化1 : 设置资源
 资源量 
 是否充分使用了集群中的资源
 executor个数,每个execcutor的内存和core等

 增加资源量
 增加executor个数(--num-executors 4)
 增加每个executor可同时运行的task数目 (--executor-cores 2)

 ## 优化2: 调整任务的并行度
多少个并行的任务
Map任务并行度
Reduce 任务并行度

适当调整任务数目
Map 个数: 默认 与 hdfs block/hbase region数目一致


task

executor




 ## 优化3 修改存储格式



spark driver 和executor

每个core 对应一个 task
每个action 对应一个job


spark调优准备: 辅助工具

spark 界面
jstack,jstat,jprofile
查看运行完成的日志

spark调优准备 : spark-history



spark调优思路

1,spark运行环境,存储与计算资源
2,优化Rdd操作符的使用方法
3,参数调优

主要是两个方面: 序列化和缓冲区
spark程序传递擦书方法:
