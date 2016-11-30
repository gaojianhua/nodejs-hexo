title: 乡村销客离线计算--ambari-spark(1)
date: 2016-09-29 22:12:11
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

###  提交spark job 到 yarn

__spark run on yarn :__ 

http://spark.apache.org/docs/latest/running-on-yarn.html

__submit application:__ 
http://spark.apache.org/docs/latest/submitting-applications.html

submit to master
```bash
	./bin/spark-submit \
	--name "WordCount" \
	--master spark://192.168.0.61:7077 \
	--executor-memory 1G \
	--class et.log.vis.WordCount \
	spark-1.jar /data/china/china.txt /data/china.txt
```

submit to yarn 
```bash
	spark-submit  --name "SaleAndTaskJob" \
	--master yarn --executor-memory 1G \
	--class et.theme.vis.SaleAndTaskJob  \
	/data/sparkjar/spark-1.jar hdfs:///user/root/access.log  /data/china
```


### spark 读写 mysql 


Advanced spark-env增加配置 

```bash
export SPARK_CLASSPATH=$SPARK_CLASSPATH:/usr/hdp/mysql-connector-java-5.1.25-bin.jar
```
上传mysql jar 包到 安装 spark amb docker  容器的目录中


### spark 读写 hive


sql中写 库名.表名

```java
String sql = " select * from hdfstest.bd_product "; 
DataFrame logAll= hiveContext.sql(sql);
```

### spark  DataFrame  sql 和 join 应用

```java
String sql = " select l.request_time , l.request_path, l.app_flag, "
	+ "substring(l.request_time, 1, 10), l.user_id ,l.corp_id  "
	+ "from log_nginx_access_wzf l "
	+ " where l.request_time > \"2016-08-25\" " ;
// 从hive 中读取数据到 DataFrame
DataFrame logAll= hiveContext.sql(sql);
System.out.println("------*****count=="+logAll.count());
// 清洗 ,增加schame
JavaRDD<NginxLog> logs = logAll.javaRDD().map(new Function<Row, NginxLog>() {
	static final long serialVersionUID = 1L;
	@Override
	public NginxLog call(Row v) throws Exception {
	NginxLog pvlog = new NginxLog();
	pvlog.setRequest_time(v.getString(3));//日期
	pvlog.setRequest_path(v.getString(1));//路由
	return pvlog;
	}
});
//创建带 增加schame 的DataFrame
DataFrame timec = hiveContext.createDataFrame(logs, NginxLog.class);
//打印schame 信息
timec.printSchema();
// 注册临时表
timec.registerTempTable("log");
//查询临时表 日期大于25日
DataFrame timec1 = hiveContext.sql("select request_time , count(request_path)  from log where request_time > \"2016-08-25\" group by  request_time order by request_time ");
timec1.printSchema();
//查询临时表 日期大于28日
DataFrame timec2 = hiveContext.sql("select request_time , count(request_path)   from log where request_time > \"2016-08-28\" group by  request_time order by request_time ");
timec2.printSchema();
// 合并2个DataFrame  相当与 left join
//joinType - One of: inner, outer, left_outer, right_outer, leftsemi.
DataFrame join = timec1.join(timec2,
timec2.col("request_time").equalTo(timec1.col("request_time")),
"left_outer");

```


/*参考资料*/
http://robingao.xyz/2016/08/29/ambari/
