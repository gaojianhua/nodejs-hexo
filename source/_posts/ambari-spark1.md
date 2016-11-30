title: 乡村销客离线计算--ambari-spark(2)
date: 2016-10-10 20:22:31
tags:
- 开始
- 我
- 日记
categories: spark
---

### 乡村销客
__ 乡村销客官网 __: http://www.vilsale.com 
乡村销客是面向化肥行业的企业互联网营销工具。通过“移动应用+云计算+应用市场”的互联网领先技术，帮助化肥生产销售企业快速实现  
<!-- more -->
1. __ 移动化市场营销及客户拜访，__  
1. __ 解决调度发运响应不畅  __  
1. __ 客户账户对账不准等问题， __  
1. __ 帮助肥料企业营销及客户管理 __”。 

乡村销客基于软件即服务的互联网理念，创建国内第一个专注__ 肥料行业的SAAS平台 __，为肥料企业打造性价比最高的企业互联网营销工具。
 
-----------这是广告结束的分割线--------------------------

###  spark sql  应用

hive , spark sql , oracle 中

分组排序取topN

案例需求
对每个班级内的学生成绩，取出前3名。（分组取topN）

class1 90  
class2 56  
class1 87  
class1 76  
class2 88  
class1 95  
class1 74  
class2 87  
class2 67  
class2 77  
class1 98  
class2 96 


```bash
select className,score from (SELECT className,score, Row_Number() OVER (partition by className ORDER BY score desc ) rank FROM scores ) a where a.rank<=3; 
``` 

实际就是用了 row_number() over (partition by ... order by ...)的函数。同样hive也是支持的



### hive 排名


Hive在0.11.0版本开始加入了row_number、rank、dense_rank分析函数，可以查询分组排序后的top值

说明：
row_number() over ([partition col1] [order by col2])
rank() over ([partition col1] [order by col2])
dense_rank() over ([partition col1] [order by col2])
它们都是根据col1字段分组，然后对col2字段进行排序，对排序后的每行生成一个行号，这个行号从1开始递增
col1、col2都可以是多个字段，用‘,‘分隔
 
区别：
1）row_number：不管col2字段的值是否相等，行号一直递增，比如：有两条记录的值相等，但一个是第一，一个是第二
2）rank：上下两条记录的col2相等时，记录的行号是一样的，但下一个col2值的行号递增N（N是重复的次数），比如：有两条并列第一，下一个是第三，没有第二
3）dense_rank：上下两条记录的col2相等时，下一个col2值的行号递增1，比如：有两条并列第一，下一个是第二

业务实例：
统计每个学科的前三名
select * from (select *, row_number() over (partition by sub order by score desc) as od from t ) t where od<=3;
语文成绩是80分的排名是多少
hive (test)> select od from (select *, row_number() over (partition by sub order by score desc) as od from t ) t where sub=‘chinese‘ and score=80;
分页查询
hive (test)> select * from (select *, row_number() over () as rn from t) t1 where rn between 1 and 5;




/*参考资料*/

http://blog.csdn.net/kwu_ganymede/article/details/50443433



http://www.mamicode.com/info-detail-849458.html
