title: 乡村销客离线计算--日志清洗
date: 2016-08-10 20:22:11
tags:
- 开始
- 我
- 日记
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

###  nginx配置

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

每次调用nginx 都会有日志记录,如下:

      192.168.0.53 wuzoufen-f76b-4437-a3bd-9f1ab1343dfc 07aafae5-d5c6-4240-a1ba-0f2785d92e4c [28/Jul/2016:20:41:37 +0800] 
		"GET /vsaapi/leader/homepage?user_id=07aafae5-d5c6-4240-a1ba-0f2785d92e4c&corp_id=wuzoufen-f76b-4437-a3bd-9f1ab1343dfc HTTP/1.0
		" 200 349 
		"-
		" 
		"Mozilla/5.0 (Linux; Android 6.0.1; MI 4W Build/MMB29M) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/43.0.2357.130 Crosswalk/14.43.343.24 Mobile Safari/537.36
		" 0.083 vis-main 192.168.0.156

### 清洗之后日志字段:

		192.168.0.53  --转化成整数 
		wuzoufen-f76b-4437-a3bd-9f1ab1343dfc 
		07aafae5-d5c6-4240-a1ba-0f2785d92e4c 
		2016-08-03 12:12:12 
		/vsaapi/leader/homepage/
		200 
		349 
		Mozilla/5.0 (Linux; Android 6.0.1; MI 4W Build/MMB29M) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/43.0.2357.130 Crosswalk/14.43.343.24 Mobile Safari/537.36
		0.083
		vis-main 

有这些信息,可以做很多的统计分析 , pv ,耗时,功能使用频率使用时间峰值,客户端环境分布, 使用地区分布 等等

### 添加$upstream_response_time

下面介绍下2者的差别：

1、request_time
官网描述：request processing time in seconds with a milliseconds resolution; time elapsed between the first bytes were read from the client and the log write after the last bytes were sent to the client 。
指的就是从接受用户请求的第一个字节到发送完响应数据的时间，即包括接收请求数据时间、程序响应时间、输出
响应数据时间。

2、upstream_response_time
官网描述：keeps times of responses obtained from upstream servers; times are kept in seconds with a milliseconds resolution. Several response times are separated by commas and colons like addresses in the $upstream_addr variable

是指从Nginx向后端（php-cgi)建立连接开始到接受完数据然后关闭连接为止的时间。

从上面的描述可以看出，$request_time肯定比$upstream_response_time值大，特别是使用POST方式传递参数时，因为Nginx会把request body缓存住，接受完毕后才会把数据一起发给后端。所以如果用户网络较差，或者传递数据较大时，$request_time会比$upstream_response_time大很多。

所以如果使用nginx的accesslog查看php程序中哪些接口比较慢的话，记得在log_format中加入$upstream_response_time



### 具体处理流程:

主要分为四个步骤, 以存储HDFS为例 流程如下图所示

1. 日志数据接入  
	数据接入采用Flume agent 上传nginx 日志, 用Flume 为controller 接受分发日志, 可以存储到
	HDFS中.  
	具体可参考美团日志收集系统的架构设计:  http://tech.meituan.com/mt-log-system-arch.html

2. 日志数据清洗 
日志清洗 主要采用spark 的定时任务,清洗出有效数据,并保存到hive数据仓库中存储 . 


![离线日志处理流程](/uploads/NginxAccessLog.png "离线日志处理流程")  


__在线原图 :__ https://www.processon.com/view/link/57a80262e4b0fb4c10d00947


### spark 日志清洗案例 

	public class LogNginxClean {

public static void main(String[] args) {
	if (args.length < 1) {
		System.err.println(" Usage: JavaWordCount <file> <savepath> ");
		System.out.println(" examle: ./bin/spark-submit  --name \"WordCount\"  "
       		+ "--master spark://192.168.0.61:7077 --executor-memory 1G  "
       		+ "--class et.log.vis.WordCount  spark-1.jar "
       		+ "/data/china/china.txt file:///data/china ");
		System.exit(1);
	}
	SparkConf sparkConf = new SparkConf().setAppName("LogNginxClean");
	JavaSparkContext ctx = new JavaSparkContext(sparkConf);
	//读取文件数据
	JavaRDD<String> lines = ctx.textFile(args[0], 1);
    
	/** map 清洗数据*/
	JavaRDD<NginxLog> logtextvos =  lines.map(
		new Function<String, NginxLog>() {
			private static final long serialVersionUID = 1L;
			@Override
            public NginxLog call(String line) {
				NginxLog logtext = transformVO(line);
				return logtext;
             }
         });
		
	//创建 hive 连接
	HiveContext hiveContext  = new HiveContext(ctx);
	//创建 DataFrame 
	DataFrame logs = hiveContext.createDataFrame(logtextvos, NginxLog.class);
	try {
		logs.write().insertInto("log_nginx_access");//追加数据
	} catch (Exception e) {
		e.printStackTrace();
		/**DataFrame保存到hive的表中*/
		logs.write().saveAsTable("log_nginx_access");//新建表,插入数据
	}
	ctx.stop();
}




### 








/*参考资料*/
