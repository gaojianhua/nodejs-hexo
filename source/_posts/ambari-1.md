title: 乡村销客离线计算--ambari2.4.1和hdp2.5安装部署
date: 2016-10-29 22:12:19
tags:
- 乡村销客
- 大数据
categories: ambari
---

# 1.乡村销客
{% cq %} __ 乡村销客官网 __:http://www.vilsale.com  {% endcq %}
乡村销客是面向化肥行业的企业互联网营销工具。通过“移动应用+云计算+应用市场”的互联网领先技术，帮助化肥生产销售企业快速实现  
<img src="http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/vis.png" class="full-image">
<!-- more -->
1. __ 移动化市场营销及客户拜访，__  
1. __ 解决调度发运响应不畅  __  
1. __ 客户账户对账不准等问题， __  
1. __ 帮助肥料企业营销及客户管理 __”。 

乡村销客基于软件即服务的互联网理念，创建国内第一个专注__ 肥料行业的SAAS平台 __，为肥料企业打造性价比最高的企业互联网营销工具。
 
-----------这是广告结束的分割线--------------------------

-----

# 2.集群环境

censtos7 三台虚拟机  4核4G内存, 50存储
jdk8,mysql5.6

192.168.0.65  h65m h65m.hadoop
192.168.0.66  h66  h66.hadoop
192.168.0.67  h67  h67.hadoop

----

# 3.操作系统环境

## 配置SSH免密码登陆

主节点(h65m)里root用户登录执行如下步骤  

```
ssh-keygen
cd ~/.ssh/
cat id_rsa.pub >>authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

在从节点登录root执行命令

```bash
mkdir ~/.ssh/  
chmod 700 ~/.ssh
```

在主节点,分发主节点里配置好的authorized_keys到各从节点

```bash
scp /root/.ssh/authorized_keys root@192.168.0.66:/root/.ssh/authorized_keys

```

## 创建ambari用户和用户组

只在主节点操作

添加ambari安装、运行用户和用户组，也可以不创建新用户，直接使用root或者系统其他账号

adduser ambari
passwd ambari

## 开启NTP服务

__所有集群上节点__都需要操作

Centos 7 命令

```bash
yum  install ntp
systemctl is-enabled ntpd
systemctl enable ntpd
systemctl start ntpd
```
Centos 6 命令

```bash
yum install ntpd
chkconfig --list ntpd
chkconfig ntpd 
service ntpd start
```

## 检查DNS和NSCD

__所有节点都要设置__

ambari在安装时需要配置全域名，所以需要检查DNS。为了减轻DNS的负担, 建议在节点里用 Name Service Caching Daemon (NSCD)

```bash
vi /etc/hosts
192.168.0.65 h65m h65m.hadoop
192.168.0.66 h66 h66.hadoop
192.168.0.67 h67 h67.hadoop
```

每台节点里配置FQDN，如下以主节点为例

```bash
vi /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=h65m.hadoop
```

## 关闭防火墙

所有节点都要设置

Centos 7 命令

```bash
systemctl disable firewalld
systemctl stop firewalld 
```

Centos 6 命令

```bash
chkconfig iptables off
/etc/init.d/iptables stop
```


## 关闭SELinux

所有节点都要设置

查看SELinux状态：

```bash
sestatus
```
如果SELinux status参数为enabled即为开启状态 
SELinux status: enabled

临时关闭，不用重启机器：

```bash
setenforce 0
```
修改配置文件需要重启机器：

```bash
vi /etc/sysconfig/selinux
SELINUX=disabled
```


----

# 4.制作本地安装源

制作本地源只需在主节点上进行即可

## 安装 Apache HTTP 服务器

安装HTTP 服务器，允许 http 服务通过防火墙(永久)

```bash
yum install httpd
firewall-cmd --add-service=http 
firewall-cmd --permanent --add-service=http
```

添加 Apache 服务到系统层使其随系统自动启动

```bash
systemctl start httpd.service
systemctl enable httpd.service
```

##  安装本地源制作相关工具

```bash
yum install yum-utils createrepo
```

## 下载安装资源

## 下载 Ambari 2.4.1 , HDP 2.5.0 的安装资源
本文使用centos7的安装源

Ambari地址中查找下载路径:
http://docs.hortonworks.com/HDPDocuments/Ambari-2.4.1.0/bk_ambari-installation/content/ambari_repositories.html

hdp地址中查找下载路径: 
http://docs.hortonworks.com/HDPDocuments/Ambari-2.4.1.0/bk_ambari-installation/content/hdp_25_repositories.html

```bash

wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.4.1.0/ambari-2.4.1.0-centos7.tar.gz

wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.5.0.0/HDP-2.5.0.0-centos7-rpm.tar.gz


wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/centos7/HDP-UTILS-1.1.0.21-centos7.tar.gz

```


## 在httpd网站根目录 创建ambari目录

在httpd网站根目录,默认是即/var/www/html/，创建目录ambari， 
并且将下载的压缩包解压到/var/www/html/ambari目录

```bash
cd /var/www/html/
mkdir ambari
cd /var/www/html/ambari/
tar -zxvf ambari-2.2.2.0-centos7.tar.gz
tar -zxvf HDP-2.4.2.0-centos7-rpm.tar.gz
tar -zxvf HDP-UTILS-1.1.0.20-centos7.tar.gz
```
验证httd网站是否可用，可以使用links 命令或者浏览器直接访问下面的地址：


http://192.168.0.65/ambari


## 配置ambari、HDP、HDP-UTILS的本地源


首先下载上面资源列表中的相应repo文件，修改其中的URL为本地的地址，相关配置如下：

ambari.repo
直接执行AMBARI-2.4.1.0/setup_repo.sh 即可生成ambari.repo

hdp.repo
在/HDP/centos7/ 下载 hdp.repo 修改为如下

```bash
#VERSION_NUMBER=2.5.0.0-1245
[HDP-2.5.0.0]
name=HDP Version - HDP-2.5.0.0
baseurl=http://192.168.0.65/ambari/HDP/centos7
gpgcheck=1
gpgkey=http://192.168.0.65/ambari/HDP/centos7/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1


[HDP-UTILS-1.1.0.21]
name=HDP-UTILS Version - HDP-UTILS-1.1.0.21
baseurl=http://192.168.0.65/ambari/HDP-UTILS-1.1.0.21/repos/centos7
gpgcheck=1
gpgkey=http://192.168.0.65/ambari/HDP/centos7/2.x/updates/2.5.0.0/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1
```

将上面的修改过的源放到/etc/yum.repos.d/下面


```bash
yum clean all
yum list update
yum makecache
yum repolist

```


##  安装mysql数据库

Ambari安装会将安装等信息写入数据库，建议使用自己安装的MySQL数据库，也可以不安装而使用默认数据库PostgreSQL

Mysql数据库安装请参考下面文章:

使用docker安装:
https://eteng-wiki.github.io/DB-WIKI/mysql/

或者参考

http://blog.csdn.net/lochy/article/details/51721319

安装完成后创建ambari数据库及用户，记住下面的用户名和密码 后面安装的时候需要用的,
登录root用户执行下面语句

```bash
create database ambari character set utf8 ;  
CREATE USER 'ambari'@'%'IDENTIFIED BY 'Ambari-123';
GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'%';
FLUSH PRIVILEGES;
```
如果要安装Hive，再创建Hive数据库和用户 再执行下面的语句：

```bash
create database hive character set utf8 ;  
CREATE USER 'hive'@'%'IDENTIFIED BY 'Hive-123';
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'%';
FLUSH PRIVILEGES;
```
如果要安装Oozie，再创建Oozie数据库和用户 再执行下面的语句：

```bash
create database oozie character set utf8 ;  
CREATE USER 'oozie'@'%'IDENTIFIED BY 'Oozie-123';
GRANT ALL PRIVILEGES ON *.* TO 'oozie'@'%';
FLUSH PRIVILEGES;
```

## 安装JDK

参考:
http://gjhbiji.mydoc.io/?t=36412

配置java环境变量

```bash
vim /etc/profile
export JAVA_HOME=/opt/java/jdk1.8.0_91
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
source /etc/profile
```


## 安装mysql jdbc 驱动

```bash
yum install mysql-connector-java
```


----

# 5.安装配置ambari  

## 安装Ambari

在主节点执行

```bash
yum install ambari-server
```
## 配置Ambari

```bash
ambari-server setup
```
下面是配置执行流程，按照提示操作

1.检查SELinux是否关闭，如果关闭不用操作

```bash
Using python  /usr/bin/python
Setup ambari-server
Checking SELinux...
SELinux status is 'disabled'
```

2.提示是否自定义设置。输入：y

```bash
Customize user account for ambari-server daemon [y/n] (n)? y
```

3.ambari-server 账号。输入：ambari

```bash
Enter user account for ambari-server daemon (root):ambari
Adjusting ambari-server permissions and ownership...
```
4.检查防火墙，如果关闭则不用操作
```bash
Checking firewall status...
Redirecting to /bin/systemctl status  iptables.service
```

5.设置JDK。输入：3

```bash
Checking JDK...
Do you want to change Oracle JDK [y/n] (n)? y
[] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[] Oracle JDK 1.7 + Java Cryptography Extension (JCE) Policy Files 7
[] Custom JDK
==============================================================================
Enter choice (1): 3
```

6.如果上面选择3自定义JDK,则需要设置JAVA_HOME。输入：/opt/Java/jdk1.8.0_91

```bash
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /opt/java/jdk1.8.0_91
Validating JDK on Ambari Server...done.
Completing setup...
```

7.数据库配置。选择：y

```bash
Configuring database...
Enter advanced database configuration [y/n] (n)? y
```

8.选择数据库类型。输入：3

```bash
Configuring database...
==============================================================================
Choose one of the following options:
[1] - PostgreSQL (Embedded)
[2] - Oracle
[3] - MySQL
[4] - PostgreSQL
[5] - Microsoft SQL Server (Tech Preview)
[6] - SQL Anywhere
==============================================================================
Enter choice (3): 3
```
9.设置数据库的具体配置信息，根据实际情况输入，如果和括号内相同，则可以直接回车。

```bash
Hostname (localhost): 
Port (): 
Database name (ambari): 
Username (ambari): 
Enter Database Password (Ambari-123): 
```

10.提示必须安装MySQL JDBC，回车结束ambari配置

```bash
WARNING: Before starting Ambari Server, you must copy the MySQL JDBC driver JAR file to /usr/share/java.
Press <enter> to continue.
```
11.将Ambari数据库脚本导入到数据库

如果使用自己定义的数据库，必须在启动Ambari服务之前导入Ambari的sql脚本

用Ambari用户（上面设置的用户）登录mysql

```bash
mysql -u ambari -p
use ambari
source /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql
```
## 启动Amabri

执行启动命令，启动Ambari服务

```bash
ambari-server start
```
成功启动后在浏览器输入Ambari地址：

http://192.168.0.65:8080/
出现登录界面，默认管理员账户登录， 账户：admin 密码：admin



----

# 6.安装hdp集群 
  
## 1 出现登录界面，

默认管理员账户登录， 账户：admin 密码：admin

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb1.png "安装hdp集群") 


## 2登陆成功后，如下界面，点击 Launch Install Wizard 

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb2.png "安装hdp集群") 

## 3 安装集群， 设置一个集群名字

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb3.png "安装hdp集群") 

## 4 设置HDP安装源

选择HDP2.5 ,并且设置Advanced Repository Options 的信息，本次使用本地源，所以修改对用系统的安装源为本地源地址。

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb4.png "安装hdp集群") 

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb5.png "安装hdp集群") 


## 6设置集群机器，
选择密钥 id_rsa ,

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb6.png "安装hdp集群") 

## 7确认集群host
确认前面配置集群中hosts列表 中的机器是否都可用，也可以移除相关机器，机器Success后进行下一步操作。

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb7.png "安装hdp集群") 

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb8.png "安装hdp集群") 

## 9 选择安装的服务

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb9.png "安装hdp集群") 

## 10 各个服务Masters 配置，
可以自己选择机器

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb10.png "安装hdp集群") 

## 11服务的Slaves 和 Clients节配置 

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb11.png "安装hdp集群") 

## 12服务的个性化配置

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb12.png "安装hdp集群") 

## 13 hdfs 配置修改
可以不更改

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb13.png "安装hdp集群") 

## 14HIVE配置源数据

需要上传mysql.jar  执行 图中命令 配置mysql驱动 ， 没有装hive的略过

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb14.png "安装hdp集群") 

## 15显示配置信息

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb15.png "安装hdp集群") 

## 16 开始安装 

![安装hdp集群](http://vis-workworld-image.oss-cn-qingdao.aliyuncs.com/11echarts/11wiki/amb16.png "安装hdp集群") 

# 卸载ambari

卸载ambari 请看下一篇文章 



/*参考资料*/

官方文档:   
http://docs.hortonworks.com/HDPDocuments/Ambari-2.4.1.0/bk_ambari-installation/content/ch_Getting_Ready.html

HDP组件安装:  
http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.0/bk_command-line-installation/content/ch_getting_ready_chapter.html

配置ssh免密码登陆:  
http://www.cnblogs.com/xia520pi/archive/2012/05/16/2503949.html



