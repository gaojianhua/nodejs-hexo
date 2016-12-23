title: gradle used
date: 2016-03-27 19:42:21
tags:

- 笔记
categories: gradle
---

### gradle简介
 Gradle是一个基于Apache Ant和Apache Maven概念的项目自动化建构工具。它使用一种基于Groovy的特定领域语言来声明项目设置，而不是传统的XML。既保持了Maven的优点，又通过使用Groovy定义的DSL[2]，克服了 Maven中使用XML繁冗以及不灵活等缺点。
 当前其支持的语言限于Java、Groovy和Scala，计划未来将支持更多的语言。
<!-- more -->
#### 功能
 1. gradle对多工程的构建支持很出色，工程依赖是gradle的第一公民。
 2. gradle支持局部构建。
　3. 支持多方式依赖管理：包括从maven远程仓库、nexus私服、ivy仓库以及本地文件系统的jars或者dirs
 4. gradle是第一个构建集成工具（the first build integration tool），与ant、maven、ivy有良好的相容相关性。
 5. 轻松迁移：gradle适用于任何结构的工程（Gradle can adapt to any structure you have.）。你可以在同一个开发平台平行构建原工程和gradle工程。通常要求写相关测试，以保证开发的插件的相似性，这种迁移可以减少破坏性，尽可能的可靠。这也是重构的最佳实践。
 6. gradle的整体设计是以作为一种语言为导向的，而非成为一个严格死板的框架。
 7. 免费开源
#### gradle提供了什么
1.一种可切换的，像maven一样的基于约定的构建框架，却又从不锁住你（约定优于配置）
Switchable, build-by-convention frameworks a la Maven. But we never lock you in!
2. 强大的支持多工程的构建
3. 强大的依赖管理（基于Apache Ivy），提供最大的便利去构建你的工程
Language for dependency based programming
4. 全力支持已有的Maven或者Ivy仓库基础建设
5. 支持传递性依赖管理，在不需要远程仓库和pom.xml和ivy配置文件的前提下
6 基于groovy脚本构建，其build脚本使用groovy语言编写
7 具有广泛的领域模型支持你的构建A rich domain model for describing your build.
#### 使用工具
1. IntelliJ IDEA 当前最新版本15.0.3
2. Eclipse
3. Android Studio
等


### eclipse 中gradle插件的使用

#### 安装gradle插件
	由于公司历史的原因,开发工具一直用Eclipse进行,故以下介绍基于Eclipse

	help>Eclipse marketplace
	在find处输入 gradle 点击右边的go 按钮
	查询结果中找到Gradle(sts) intergration fro Eclipse ****.RELEASE 点击install 或者直接拖到Eclipse中安装


#### 使用
	创建gradle项目, new > other > gradle(sts) project

	目录结构
	 src/main/java         --开发源码包
	 src/main/resoureces   --开发资源包
	 src/test/java         --测试源码包
	 src/test/resources    --测试资源包
	 build                 --build存放目录包
	 src                   --build源码包
	 build.gradle          --gradle任务配置文件

	本公司员工可以用svn检出平台sin项目 
	http://103.10.84.37/svn/SIN/trunk 作为参考项目此项目为多项目
	检出之后先从Eclispe中删除,让后import 作为gradle项目, 
	选择sin项目之后,点击build model ,成功之后选择所有子项目 finish

#### 配置build.gradle
```bash
buildscript {
	repositories {
	  jcenter()
	}	
	dependencies {
	  classpath 'org.akhikhl.gretty:gretty:+'
	}
}

buildscript {
  repositories {
    jcenter()
  }
  dependencies {
    classpath 'org.hidetake:gradle-ssh-plugin:2.0.0'
  }
}

/****引入gradle插件*****************/
apply plugin: 'java'
apply plugin: 'eclipse'	
/****jdk版本*****************/
sourceCompatibility = 1.7

/****maven仓库*****************/
repositories {
    maven {    	
        url "http://103.10.84.37:8081/nexus/content/groups/public/"
    }
    maven {    	
        url "http://repository.ow2.org/nexus/content/repositories/public/"
    }
}
/****编译编码，解决Gradle编译时出现： 编码GBK的不可映射字符*****************/
[compileJava, compileTestJava]*.options*.encoding = 'UTF-8'

/****依赖关系*****************/
dependencies {
	compile "et.sin:sin-cache:1.0.8",
			"et.sin:sin-resteasy:1.0.8"
	runtime "et.sin:sin-codeseed:1.0.8"	
	testCompile	"junit:junit:4.9",
				"org.springframework:spring-test:4.2.4.RELEASE"		
}

/****生成jar*****************/
version = '1.0'//发布版本
jar {
	manifest {
        attributes 'Created-By':'eteng-etd',//声明该文件的生成者
        'Implementation-Title': 'Singularity', 
        'Implementation-Version': version
    }
	//jar包包含测试代码，用于其它项目测试调用
	from(sourceSets.test.allJava)
}
/****生成源码jar*****************/
task sourceJar(type: Jar) { 
    from sourceSets.main.allJava//主体源代码
    from sourceSets.test.allJava//测试源代码
    classifier "sources"
}
/***生成javadoc,命令行参数配置**/
javadoc {
	source = [sourceSets.main.allJava,sourceSets.test.allJava] //需要生成javadoc的源码包
	classpath=configurations.testCompile
    options.encoding = "UTF-8"
    options.charSet = "UTF-8"
    options.links = ["http://docs.oracle.com/javase/8/docs/api/"]
    options.memberLevel="PACKAGE"
}
//生成javadoc jar
task javadocJar(type: Jar, dependsOn: javadoc) {
	from javadoc.destinationDir
    classifier = 'javadoc'	    
}

task codeseed(type: JavaExec, dependsOn: 'classes') {
    description '运行代码生成器'
    classpath = sourceSets.main.runtimeClasspath
    main = "et.common.codeseed.CodeSeed"
}

/****引入gretty插件*****************/	
apply plugin: 'org.akhikhl.gretty'	
gretty {
    port = 8080
    servletContainer = 'tomcat8'
    //contextPath ="/sin"
	extraResourceBase   project.getProjectDir().getAbsolutePath() + "/webapp" 	    
    contextConfigFile = project.getProjectDir().getAbsolutePath() + "/webapp/WEB-INF/ROOT.xml"    
    enableNaming=true //启用JNDI
}
```
此平台推荐配置 ,请参考平台demo项目中的build.gradle
初级入门可使用以上配置运行一个web项目了, 客官请往下看
>
>
>

#### build.gradle详解
看完builld.gradle配置 ,我们知道, 其实 这个文件就是配置引入那些 gradle 的插件,自定义任务和第三方jar包依赖关系

/******引入java插件*******/

apply plugin: 'java' 

引入插件之后 ,相当于 引入了一定任务,执行之后帮我们做一定的操作
例如引入java之后,  在gradle task 视图中我们选择 project 为自己的项目
就可看到相关任务

还可以设置一些必要的配置信息
/*******jdk版本********/
sourceCompatibility = 1.7
/****maven仓库下载依赖jar用*****/
repositories {
}

还有配置关键的jar包依赖
/**********依赖关系***********/
dependencies {
	compile "et.sin:sin-cache:1.0.8",
			"et.sin:sin-resteasy:1.0.8"
	runtime "et.sin:sin-codeseed:1.0.8"	
	testCompile	"junit:junit:4.9",
				"org.springframework:spring-test:4.2.4.RELEASE"		
}

compile 代表编译时依赖
runtime 代表运行时依赖
testCompile junit 测试时的依赖

更详细的请看http://wiki.jikexueyuan.com/project/gradle/dependency-management-basics.html 当然也可以看官网的配置

/**********自定义任务********************/
task codeseed(type: JavaExec, dependsOn: 'classes') {
    description '运行代码生成器'
    classpath = sourceSets.main.runtimeClasspath
    main = "et.common.codeseed.CodeSeed"
}


#### 启动tomcat容器
	我们使用了gretty插件,配置如下
buildscript {
	repositories {
	  jcenter()
	}	
	dependencies {
	  classpath 'org.akhikhl.gretty:gretty:+'
	}
}

/****引入gretty插件*****************/	
apply plugin: 'org.akhikhl.gretty'	
gretty {
    port = 8080
    servletContainer = 'tomcat8'
    //contextPath ="/sin"
	extraResourceBase   project.getProjectDir().getAbsolutePath() + "/webapp" 	    
    contextConfigFile = project.getProjectDir().getAbsolutePath() + "/webapp/WEB-INF/ROOT.xml"    
    enableNaming=true //启用JNDI
}


引入插件之后,看到 gradle task视图中多了关于app的任务

appStart
appStop
appStartDebug
appRestart
等等


启动appStart ,tomcat容器启动



#### junit测试
	参考平台 test的目录的写法

基本到此为止......基本可以进行开发测试了
/*当然你可能还不满足于这点东西,,那就继续吧*/
>
>
>
>

### build.gradle 更多高级用法

在大型java项目中,使用的gradle插件增多和任务的增多单个配置就显得臃肿了,
以下介绍一些其他用法:

#### 引入其他.gradle文件
	我们可以分类写.gradle文件,然后在build.gradle中引入即可.
	例如,我可以把关于gretty 的配置写在 gretty.gradle

	在build.gradle 中引入 
	/*********ssh自动部署********************/
	apply from: "ssh.gradle"
	/********gretty插件配置*****************/	
	apply from: "gretty.gradle"

#### 项目中多平台配置文件管理
	配置其他的资源文件, 打包时会自动加入

	例如生产系统的配置和测试系统的配置时不一样的,
	正式发布时,我们把配置指向正式系统, 配置文件会自动覆盖/src/main/resources,中相同的配置文件

	//设置其他资源文件 --打包正式时启用production
	sourceSets {
	    main {
	        resources {
	            //srcDir "config/production"
	            srcDir "src/main/resources"
	        }
	    }
	}

#### ssh自动部署

	配置ssh.gradle  ,在build.gradle中引入 , 依赖于war任务 ,
利用ssh,先打包出项目的war包,  上传到指定的服务器,,然后执行命令解压命令,让后执行重启tomcat的命令.
	配置如下

	```bash
	/****引入gradle插件*****************/
	//https://gradle-ssh-plugin.github.io/
	apply plugin: 'org.hidetake.ssh'

	/*************打包war设置********************/
	//https://dongchuan.gitbooks.io/gradle-user-guide-/content/the_war_plugin/war.html
	// war  或jar包的版本
	version = '1.4'
	// War 或jar 包名称
	String output = new Date().format('yyyyMMddHHmm')
	war.baseName = 'VSA_WEB'+output
	// Web directory, this overrides the default value "src/java/webapp"
	project.webAppDirName = 'webapp'
	//设置其他资源文件 --打包正式时启用production
	sourceSets {
	    main {
	        resources {
	            //srcDir "config/production"
	            srcDir "src/main/resources"
	        }
	    }
	}
	/************ssh自动部署***internal测试*****product正式******/
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


	task sshCheck << {
	    ssh.run {
	        session(remotes.product) {
	            def result = execute 'uname -a'
	            execute 'cat /etc/*-release', ignoreError: true
	        }
	    }
	}
	/***********发布测试************/
	task sshLocalDeploy(dependsOn: war) << {
	    ssh.run {
	        session(remotes.internal) {
	        	execute '/opt/bin/tomcat-sc.sh tomcat-vistv stop'
	            put from: war.archivePath.path, into: '/opt/buildapp'
	            println '--**war upload successed...'
	            execute 'unzip -oq  /opt/buildapp/' +war.archiveName + ' -d /opt/apps/VISTV_WEB'
	            println '--**war unzip successed...'
	            execute '/opt/bin/tomcat-sc.sh tomcat-vistv start'
	            println '--**server restart successed...'
	        }
	    }
	}
	/***********发布测试***************/
	task sshProductDeploy(dependsOn: war) << {
	    ssh.run {
	        session(remotes.product) {
	         	println '--**war starting upload ...'
	            put from: war.archivePath.path, into: '/opt/back_apps'
	            //execute 'unzip -oq  /opt/back_apps/' +war.archiveName + ' -d /opt/apps/VSA_WEB'
	         	//execute '/opt/bin/tomcat-sc.sh tomcat-test start'
	        }
	    }
	}

	```

/*参考资料*/
极客学院-gradle使用教程:[http://wiki.jikexueyuan.com/project/gradle]

美团点评: [http://tech.meituan.com/gradle-practice.html]

 Gradle在大型Java项目上的应用:[http://www.bianceng.cn/Programming/Java/201312/38553_4.htm]