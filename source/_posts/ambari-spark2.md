title: 乡村销客离线计算--ambari-spark(3)
date: 2016-10-17 22:13:01
tags:
- 乡村销客
- 大数据
- spark
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

###  spark sql  join  应用

### Hive SQL: Both left and right aliases encountered in JOIN


mysql中的sql 

```sql
select * from bd_region reg 
join sale_info s on reg.region_treecode like concat(reg.region_treecode ,'%') ;
```


转化为hive 中的sql

```sql
select * from bd_region 
join sale_info on bd_region.corp_id = sale_info.corp_id and bd_region.region_type = 0 
where sale_info.reg_treecode like concat(bd_region.region_treecode,'%') ;
```


/*参考资料*/

http://stackoverflow.com/questions/36015035/hive-sql-both-left-and-right-aliases-encountered-in-join

