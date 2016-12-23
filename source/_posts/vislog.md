title: 乡村销客离线计算--流程设计
date: 2016-08-06 20:42:21
tags:
- 乡村销客
- 大数据
- 离线计算
categories: 日志处理
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
 
----广告结束----

### 背景简介
 乡村销客应用,主要分为手机客户端, 电脑端web应用, tv大数据分析三块 ,应用采用前后太分离的架构方式,
服务器提供restfull 接口API 供前台调用,  nginx做软件负载
nginx 日志格式化, 注意每个变量之间只有一个空格,不要多

    log_format  main  '$remote_addr $http_e_corpId $http_e_userId [$time_local] "$request" '
        	'$status $body_bytes_sent "$http_referer" '
        	'"$http_user_agent" $request_time vis-main $http_x_real_ip $upstream_response_time';

*字段解释说明:

     '$remote_addr       : 访问ip
     $http_e_corpId      : 公司id 可能为空, http请求中设置 请求头E-corpId 
     $http_e_userId      : 用户id 可能为空, http请求中设置 请求头E-userId 
     [$time_local]       : 请求ts
     "$request" '        : 请求详情
     '$status            : 请求返回状态码
     $body_bytes_sent    : 请求返回值大小
     "$http_referer" '   : 跳转页面来源
     '"$http_user_agent" : 浏览器user_agent
     $request_time       :  请求处理耗时
     vis-main            : 传入日志应用的标识
     $http_x_real_ip'    :  反向代理时, 设置访问真实ip -- 可以不设置 ---暂时没用这个


### 具体处理流程:

主要分为四个步骤, 以存储HDFS为例 流程如下图所示

1. 日志数据接入  
	数据接入采用Flume agent 上传nginx 日志, 用Flume 为controller 接受分发日志, 可以存储到
	HDFS中.  
	具体可参考美团日志收集系统的架构设计:  http://tech.meituan.com/mt-log-system-arch.html

2. 日志数据清洗 
日志清洗 主要采用spark 的定时任务,清洗出有效数据,并保存到hive数据仓库中存储 . 

3. 日志数据主题挖掘分析  
根据业务需要, 用spark的定时任务,计算出每个主题所需要的统计数据,保存到 mysql 或者hive 中, 供可视化使用

4. 可视化化 
更具主题显示 图标信息, 可以用zeppelin 或者自建应用展现



![离线日志处理流程](/uploads/NginxAccessLog.png "离线日志处理流程")  

在线原图 : https://www.processon.com/view/link/57a80262e4b0fb4c10d00947

### 

__原文链接: __http://robingao.xyz/2016/08/06/vislog/ 或者 https://github.com/gaojianhua/nodejs-hexo


/*参考资料*/
