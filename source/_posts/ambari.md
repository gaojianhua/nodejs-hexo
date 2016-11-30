title: 乡村销客离线计算--ambari平台搭建
date: 2016-08-29 21:22:11
tags:
- 开始
- 我
- 日记
categories: ambari
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

###  安装docker-engine最新版
1、centos系统更新

	yum -y update
2、添加docker源

	tee /etc/yum.repos.d/docker.repo <<-'EOF'
	[dockerrepo]
	name=Docker Repository
	baseurl=https://yum.dockerproject.org/repo/main/centos/7/
	enabled=1
	gpgcheck=1
	gpgkey=https://yum.dockerproject.org/gpg
	EOF
3、安装docker-engine

	yum -y install docker-engine
4、启动docker

	service docker start

### 安装docker 拓展命令

	yum install wget
	wget -P ~ https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker;
	echo "[ -f ~/.bashrc_docker ] && . ~/.bashrc_docker" >> ~/.bashrc; source ~/.bashrc

可执行：
docker-pid 可以获取某个容器的 PID：docker-pid myjenkins
docker-enter 可以进入容器或直接在容器内执行命令：docker-enter myjenkins

### 装sequenceiq/ambari容器

https://hub.docker.com/r/sequenceiq/ambari/
ambaris: 是hortonworks开发的一个自动部署、管理、监控  HDP（Hortonworks Data Platform， http://hortonworks.com/hdp/，由hadoop生态环境中的一些开源软件组成）的开源软件。与之相对应的有cloudera manager， cloudera 发展时间更长，网上能搜到的资料更多， 但是其开源性好像没有hortonworks好。
Ambari Server与agent 交互从而 管理集群节点，这里agent的注册可以是被动，也就是server 通过ssh连接到agent，然后启动agent 去注册，所以需要两个配置， agent上需要能够ssh 私钥 无密码登陆，而且agent上要配置server的地址。

### 下载镜像

	docker pull sequenceiq/ambari
### 下载ambari-functions脚本文件

	curl -O https://raw.githubusercontent.com/sequenceiq/docker-ambari/master/ambari-functions
### 执行脚本：ambari-functions

	source ambari-functions

source命令：
source命令也称为“点命令”，也就是一个点符号（.）,是bash的内部命令。
功能：使Shell读入指定的Shell程序文件并依次执行文件中的所有语句
source命令通常用于重新执行刚修改的初始化文件，使之立即生效，而不必注销并重新登录。


### 安装ambari-server和ambari-agent集群：

将会启动1个consul server(分布式服务注册和发现consul)、1个Ambari server、2个Ambari agents

	amb-start-cluster
或者 启动5个Ambari agents
	
	amb-start-cluster 6


假如 服务没有启动全,请删掉 安装的docker ,重新执行如上命令

	docker rm dockername
	amb-start-cluster 6

### 创建并启动ambari-shell
创建ambari-shell容器

	docker run  -d --name shell --link amb-server:ambariserver hortonworks/ambari-server

###  进入ambari-shell容器

	docker-enter shell

###  创建蓝图文件two-node-only-zookeeper（仅安装zookeeper服务）

	vi /tmp/two-node-only-zookeeper
{
  "host_groups": [
    {
      "name": "master",
      "components": [
        {
          "name": "ZOOKEEPER_SERVER"
        }      
      ],
      "cardinality": "1"
    },
    {
      "name": "slave_1",
      "components": [
        {
          "name": "ZOOKEEPER_CLIENT"
        }      
      ],
      "cardinality": "1+"
    }
  ],
  "Blueprints": {
    "blueprint_name": "two-node-only-zookeeper",
    "stack_name": "HDP",
    "stack_version": "2.5"
  }
}

### 运行ambari-shell

	export AMBARI_HOST=ambariserver;\
	export PATH=$PATH:/usr/jdk64/jdk1.7.0_67/bin;\
	cd /tmp;\
	./ambari-shell.sh


### 通过ambari-shell创建hadoop集群环境

	blueprint add --file /tmp/two-node-only-zookeeper
	cluster build --blueprint two-node-only-zookeeper
	cluster autoAssign
	cluster create

### 客户端机器添加网络路由
Windows(需要以管理员身份运行cmd): route add 172.17.0.0 mask 255.255.0.0 192.168.0.207


将consul容器作为客户端机器的DNS服务器：netsh interface ip add dns "无线网络连接 2" 172.17.0.2

Linux:  route add -net 172.17.0.0/16 192.168.0.207


route add -net 172.17.0.0 netmask 255.255.0.0 gw 192.168.0.62

8、手动安装其他服务

ZooKeeper： ZooKeeper Server、ZooKeeper Client
HDFS: NameNode、SNameNode、DataNode、HDFS Client
YARN + MapReduce2: ResourceManager、History Server、App Timeline Server、
	NodeManager、YARN Client、MapReduce2 Client
Tez:                Tez Client
Pig:                Pig Client
Slider:             Slider Client
Hiver:              Hive Metastore、HiveServer2、MySQL Server、WebHCat Server、
                     Hive Client、HCat Client
Spark:              Livy Server、Spark History Server、Spark Client
Zeppelin:           Zeppelin Notebook
Oozie:

```bash

amb1:ZooKeeper，hdfs，yarn，mapreduce,namenode,  
 ResourceManager, History Server, App Timeline Server,  
  NodeManager

amb2:oozie，hive ，snamenode,Hive Metastore,  
 HiveServer2, WebHCat Server, Oozie Server,  

amb3:spark，zeppelin，datanode, HDFS Client,  
 Spark History Server, Livy Server, Zeppelin Notebook

amb4:datanode, HDFS Client

amb5:所有client,datanode, tez，pig，slider,flume

hive密码：yiteng

```

###  ambari server管理页面

http://192.168.0.230:8080/  用户密码 admin/admin


###  HDFS NameNode管理界面

http://192.168.0.231:50070/

### MapReduce2 JobHistory 管理页面 

http://192.168.0.231:19888/


### 查找DFS Master配置

登陆ambari server管理界面->HDFS->Configs->Advanced-> Advanced hdfs-site
-> dfs.namenode.rpc-address


### 查找Map/Reduce(v2) Master配置

登陆ambari server管理界面->YARN->Configs->Advanced-> Advanced yarn-site
-> yarn.resourcemanager.scheduler.address

/*参考资料*/
http://robingao.xyz/2016/08/29/ambari/
