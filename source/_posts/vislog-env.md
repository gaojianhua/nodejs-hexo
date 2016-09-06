title: 乡村销客日志离线处理之--spark开发环境Eclipse
date: 2016-08-09 22:22:13
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

### 说明
由于公司历史原因 
本文使用Eclipse 和 gradle 简介spark开发 环境搭建 , 这里不涉及到spark的安装步骤,
想学习spark的部署请看其他文章().

### 新建gradle项目
首先Eclipse 要安装好插件,了解gradle的简单使用, 可以参考以前的文章(http://robingao.xyz/2016/03/27/gradle/)

删除项目之后重新导入.

### 配置build.gradle 

主要是添加关于spark 的开发依赖包

```bash
buildscript {
	repositories {
	  mavenCentral()
	  mavenLocal()
	  jcenter()
	}	
	dependencies {
	  classpath 'org.hidetake:gradle-ssh-plugin:2.0.0'
	}
}
/****引入gradle插件*****************/
apply plugin: 'java'
apply plugin: 'eclipse'
//apply plugin: 'scala'
/*********ssh自动部署********************/
apply from: "ssh.gradle"

/****jdk版本*****************/
sourceCompatibility = 1.8
String output = new Date().format('yyyyMMddHHmm')
version = '1'
// War 或jar 包名称
//war.baseName = 'spark-test'
jar.baseName = 'spark'

/****编译编码，解决Gradle编译时出现： 编码GBK的不可映射字符*****************/
[compileJava, compileTestJava]*.options*.encoding = 'UTF-8'
/****maven仓库*****************/
repositories {
	maven {url "http://maven.aliyun.com/nexus/content/groups/public/"}
    maven {url "http://svn.etena.cn:8081/content/groups/public/"}
    //maven {url "http://maven.aliyun.com/nexus/content/groups/public/"}
    maven {url "http://repository.ow2.org/nexus/content/repositories/public/"}
}
/****依赖关系*****************/
dependencies {

    // https://mvnrepository.com/artifact/org.apache.spark/spark-core_2.10
	compile group: 'org.apache.spark', name: 'spark-core_2.10', version: '1.6.2'
	// https://mvnrepository.com/artifact/org.apache.spark/spark-sql_2.11
	compile group: 'org.apache.spark', name: 'spark-sql_2.10', version: '1.6.2'
    // https://mvnrepository.com/artifact/org.apache.spark/spark-hive_2.10
	compile group: 'org.apache.spark', name: 'spark-hive_2.10', version: '1.6.2'	
}
```

### 配置ssh.gradle

ssh 插件主要是上传jar包,执行linux命令 ,跑spark的任务,可以根据自己的需求调整

```bash
/****引入gradle插件*****************/
//https://gradle-ssh-plugin.github.io/
apply plugin: 'org.hidetake.ssh'
/************ssh自动部署*****************/
remotes {
    internal {
        host = '192.168.0.61'
        user = 'root'
        password = 'password'
        //identity =  identity = file("${System.properties['user.home']}/.ssh/test/id_rsa")
    }
    product {
        host = '114.215.137.243'
        user = 'root'
        //password = 'password'
        identity =  identity = file("${System.properties['user.home']}/.ssh/id_rsa")
    }
}
ssh.settings {
  knownHosts = allowAnyHosts
  //dryRun = true
}
task jar2LogAnalysis1yesterday(dependsOn: jar) << {
    ssh.run {
        session(remotes.internal) {
        	execute 'rm -rf /data/sparkjar/spark-1.jar'
        	execute 'rm -rf /data/china'
            put from: jar.archivePath.path, into: '/data/sparkjar'
            println '--**jar  upload successed...'
            //execute '  /data/opt/spark-1.5.0-bin-hadoop2.6/' +jar.archiveName + ' '
			execute '/data/opt/spark-1.6.2-bin-hadoop2.6/bin/spark-submit  --name "WordCount"  --master spark://192.168.0.61:7077 --executor-memory 1G --class et.log.vis.WordCount  /data/sparkjar/spark-1.jar hdfs:///user/root/access.log  /data/china ' 
        	//execute 'cat /data/china/part-00000 '
        }
    }
}



task jar1LogClean(dependsOn: jar) << {
    ssh.run {
        session(remotes.internal) {
        	execute 'rm -rf /data/sparkjar/spark-1.jar'
        	execute 'rm -rf /data/china'
            put from: jar.archivePath.path, into: '/data/sparkjar'
            println '--**jar  upload successed...'
			execute '/data/opt/spark-1.6.2-bin-hadoop2.6/bin/spark-submit  --name "LogNginxClean"  --master spark://192.168.0.61:7077 --executor-memory 1G --class et.log.vis.LogNginxClean  /data/sparkjar/spark-1.jar hdfs:///user/root/access.log  /data/china ' 
        	//execute 'cat /data/china/part-00000 '
        }
    }
}


```

### 新建java类,

开始spark编程之旅吧!!!!

### 执行spark任务

ssh中的每一个任务都是一个spark任务,执行一个gradle 任务,就执行了一个spark任务了

### 调试

查看spark任务执行的日志, 访问spark集群的8080 端口, 

查看历史任务和正在进行的任务

![查看任务](/uploads/sparklog1.png "查看任务") 

查看日志

![查看日志](/uploads/sparklog2.png "查看日志") 


/*参考资料*/
