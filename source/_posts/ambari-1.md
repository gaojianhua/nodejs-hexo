title: 乡村销客离线计算--ambari2.4.1和hdp2.5安装部署
date: 2016-10-29 22:12:19
tags:
- 开始
- 我
- 日记
categories: ambari
---

# 1.乡村销客
__ 乡村销客官网 __: http://www.vilsale.com 
乡村销客是面向化肥行业的企业互联网营销工具。通过“移动应用+云计算+应用市场”的互联网领先技术，帮助化肥生产销售企业快速实现  
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
HOSTNAME=SY-001.hadoop
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
## 


## 



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

hdp.repo


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

## 安装mysql jdbc 驱动

```bash
yum install mysql-connector-java
```


----

# 5.安装配置ambari  

## 

## 

## 


----

# 6.安装hdp集群 
  
## 

## 

## 

----

# 7.注意事项  

## 

## 

## 


/*参考资料*/

官方文档:   
http://docs.hortonworks.com/HDPDocuments/Ambari-2.4.1.0/bk_ambari-installation/content/ch_Getting_Ready.html

HDP组件安装:  
http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.0/bk_command-line-installation/content/ch_getting_ready_chapter.html

配置ssh免密码登陆:  
http://www.cnblogs.com/xia520pi/archive/2012/05/16/2503949.html



