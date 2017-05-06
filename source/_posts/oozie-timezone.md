title: oozie更改时区--乡村销客大数据
date: 2017-01-07 22:12:19
tags:
- 乡村销客
- 大数据
categories: oozie
---
 
# 1.乡村销客
__ 乡村销客官网 __:http://www.vilsale.com
乡村销客是面向化肥行业的企业互联网营销工具。通过“移动应用+云计算+应用市场”的互联网领先技术，帮助化肥生产销售企业快速实现  
<!-- more -->
乡村销客基于软件即服务的互联网理念，创建国内第一个专注__ 肥料行业的SAAS平台 __，为肥料企业打造性价比最高的企业互联网营销工具。
  
-----------这是广告结束的分割线--------------------------
 
-----
# 2.oozie 更改时区
 
oozie　使用　默认UTC时区，在开发oozie任务时必须在期望执行的时间上减去8小时，很不习惯，记录下修改时区的配置操作。
 
１．首先　oozie-site.xml 　添加如下配置　
 
<property>
 <name>oozie.processing.timezone</name>
 <value>GMT+0800</value>
</property>
ambari 中　点击oozie-->config-->
Custom oozie-site 中添加　oozie.processing.timezone　值为　GMT+0800
 
重启　oozie
 
２.Oozie web ui，
    settings--> Timezone
    设置timezone 为CSA(Asia/Shanghai)
 
３．任务开发过程中，
有时间参数的注意格式，如：
 
start_date=2016-02-15T08:28+0800
 
若是coordinator应用，需注意coordinator.xml文件timezone属性值如下timezone="Asia/Shanghai"
