title: 乡村销客-大数据-ambari卸载
date: 2016-12-17 20:32:50
tags:
- 乡村销客
- 大数据
- ambari
categories: ambari
---

# 1.乡村销客
__ 乡村销客官网 __: http://www.vilsale.com 
乡村销客是面向化肥行业的企业互联网营销工具。
<!-- more -->
乡村销客基于软件即服务的互联网理念，创建国内第一个专注__ 肥料行业的SAAS平台 __，为肥料企业打造性价比最高的企业互联网营销工具。
 
-----------这是广告结束的分割线--------------------------

-----

#  2. 转载脚本

```bash
#!/bin/bash
# Program:
#    uninstall ambari automatic
# History:
#    2014/01/13      First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

#取得集群的所有主机名，这里需要注意：/etc/hosts配置的IP和主机名只能用一个空格分割
hostList=$(cat /etc/hosts | tail -n +3 | cut -d ' ' -f 2)
yumReposDir=/etc/yum.repos.d/
alterNativesDir=/etc/alternatives/
pingCount=5
logPre=TDP

read -p "Please input your master hostname: " master
master=${master:-"master"}
ssh $master "ambari-server stop"
#重置ambari数据库
ssh $master "ambari-server reset"

for host in $hostList
do
    #echo $host
    #检测主机的连通性
    unPing=$(ping $host -c $pingCount | grep 'Unreachable' | wc -l)
    if [ "$unPing" == "$pingCount" ]; then
        echo -e "$logPre======>$host is Unreachable,please check '/etc/hosts' file"
        continue
    fi

    echo "$logPre======>$host deleting... \n"
    #1.)删除hdp.repo、HDP.repo、HDP-UTILS.repo和ambari.repo
    ssh $host "cd $yumReposDir"
    ssh $host "rm -rf $yumReposDir/hdp.repo"
    ssh $host "rm -rf $yumReposDir/HDP*"
    ssh $host "rm -rf $yumReposDir/ambari.repo"
    
    #删除HDP相关的安装包
    ssh $host "yum remove -y  sqoop.noarch"
    ssh $host "yum remove -y  lzo-devel.x86_64"
    ssh $host "yum remove -y  hadoop-libhdfs.x86_64"
    ssh $host "yum remove -y  rrdtool.x86_64"
    ssh $host "yum remove -y  hbase.noarch"
    ssh $host "yum remove -y  pig.noarch"
    ssh $host "yum remove -y  lzo.x86_64" 
    ssh $host "yum remove -y  ambari-log4j.noarch"
    ssh $host "yum remove -y  oozie.noarch"
    ssh $host "yum remove -y  oozie-client.noarch"
    ssh $host "yum remove -y  gweb.noarch"
    ssh $host "yum remove -y  snappy-devel.x86_64"
    ssh $host "yum remove -y  hcatalog.noarch"
    ssh $host "yum remove -y  python-rrdtool.x86_64"
    ssh $host "yum remove -y  nagios.x86_64"
    ssh $host "yum remove -y  webhcat-tar-pig.noarch"
    ssh $host "yum remove -y  snappy.x86_64"
    ssh $host "yum remove -y  libconfuse.x86_64"
    ssh $host "yum remove -y  webhcat-tar-hive.noarch"
    ssh $host "yum remove -y  ganglia-gmetad.x86_64"
    ssh $host "yum remove -y  extjs.noarch"
    ssh $host "yum remove -y  hive.noarch"
    ssh $host "yum remove -y  hadoop-lzo.x86_64"
    ssh $host "yum remove -y  hadoop-lzo-native.x86_64"
    ssh $host "yum remove -y  hadoop-native.x86_64"
    ssh $host "yum remove -y  hadoop-pipes.x86_64"
    ssh $host "yum remove -y  nagios-plugins.x86_64"
    ssh $host "yum remove -y  hadoop.x86_64"
    ssh $host "yum remove -y  zookeeper.noarch"   
    ssh $host "yum remove -y  hadoop-sbin.x86_64"
    ssh $host "yum remove -y  ganglia-gmond.x86_64"
    ssh $host "yum remove -y  libganglia.x86_64"
    ssh $host "yum remove -y  perl-rrdtool.x86_64"
    ssh $host "yum remove -y  epel-release.noarch"
    ssh $host "yum remove -y  compat-readline5*"
    ssh $host "yum remove -y  fping.x86_64"
    ssh $host "yum remove -y  perl-Crypt-DES.x86_64"
    ssh $host "yum remove -y  exim.x86_64"
    ssh $host "yum remove -y ganglia-web.noarch"
    ssh $host "yum remove -y perl-Digest-HMAC.noarch"
    ssh $host "yum remove -y perl-Digest-SHA1.x86_64"
    ssh $host "yum remove -y bigtop-jsvc.x86_64"
    
    #删除快捷方式
    ssh $host "cd $alterNativesDir"
    ssh $host "rm -rf hadoop-etc" 
    ssh $host "rm -rf zookeeper-conf"
    ssh $host "rm -rf hbase-conf" 
    ssh $host "rm -rf hadoop-log" 
    ssh $host "rm -rf hadoop-lib"
    ssh $host "rm -rf hadoop-default" 
    ssh $host "rm -rf oozie-conf"
    ssh $host "rm -rf hcatalog-conf" 
    ssh $host "rm -rf hive-conf"
    ssh $host "rm -rf hadoop-man"
    ssh $host "rm -rf sqoop-conf"
    ssh $host "rm -rf hadoop-confone"

    #删除用户
    ssh $host "userdel -rf nagios"
    ssh $host "userdel -rf hive"
    ssh $host "userdel -rf ambari-qa"
    ssh $host "userdel -rf hbase"
    ssh $host "userdel -rf oozie"
    ssh $host "userdel -rf hcat"
    ssh $host "userdel -rf mapred"
    ssh $host "userdel -rf hdfs"
    ssh $host "userdel -rf rrdcached"
    ssh $host "userdel -rf zookeeper"
    ssh $host "userdel -rf sqoop"
    ssh $host "userdel -rf puppet"
    ssh $host "userdel -rf flume"
    ssh $host "userdel -rf tez"
    ssh $host "userdel -rf yarn"

    #删除文件夹
    ssh $host "rm -rf /hadoop"
    ssh $host "rm -rf /etc/hadoop" 
    ssh $host "rm -rf /etc/hbase"
    ssh $host "rm -rf /etc/hcatalog" 
    ssh $host "rm -rf /etc/hive"
    ssh $host "rm -rf /etc/ganglia" 
    ssh $host "rm -rf /etc/nagios"
    ssh $host "rm -rf /etc/oozie"
    ssh $host "rm -rf /etc/sqoop"
    ssh $host "rm -rf /etc/zookeeper" 
    ssh $host "rm -rf /var/run/hadoop" 
    ssh $host "rm -rf /var/run/hbase"
    ssh $host "rm -rf /var/run/hive"
    ssh $host "rm -rf /var/run/ganglia" 
    ssh $host "rm -rf /var/run/nagios"
    ssh $host "rm -rf /var/run/oozie"
    ssh $host "rm -rf /var/run/zookeeper"
    ssh $host "rm -rf /var/log/hadoop"
    ssh $host "rm -rf /var/log/hbase"
    ssh $host "rm -rf /var/log/hive"
    ssh $host "rm -rf /var/log/nagios" 
    ssh $host "rm -rf /var/log/oozie"
    ssh $host "rm -rf /var/log/zookeeper" 
    ssh $host "rm -rf /usr/lib/hadoop"
    ssh $host "rm -rf /usr/lib/hbase"
    ssh $host "rm -rf /usr/lib/hcatalog" 
    ssh $host "rm -rf /usr/lib/hive"
    ssh $host "rm -rf /usr/lib/oozie"
    ssh $host "rm -rf /usr/lib/sqoop"
    ssh $host "rm -rf /usr/lib/zookeeper" 
    ssh $host "rm -rf /var/lib/hive"
    ssh $host "rm -rf /var/lib/ganglia" 
    ssh $host "rm -rf /var/lib/oozie"
    ssh $host "rm -rf /var/lib/zookeeper"
    ssh $host "rm -rf /var/tmp/oozie"
    ssh $host "rm -rf /tmp/hive"
    ssh $host "rm -rf /tmp/nagios" 
    ssh $host "rm -rf /tmp/ambari-qa" 
    ssh $host "rm -rf /tmp/sqoop-ambari-qa"
    ssh $host "rm -rf /var/nagios"
    ssh $host "rm -rf /hadoop/oozie"
    ssh $host "rm -rf /hadoop/zookeeper"
    ssh $host "rm -rf /hadoop/mapred"
    ssh $host "rm -rf /hadoop/hdfs"
    ssh $host "rm -rf /tmp/hadoop-hive" 
    ssh $host "rm -rf /tmp/hadoop-nagios" 
    ssh $host "rm -rf /tmp/hadoop-hcat"
    ssh $host "rm -rf /tmp/hadoop-ambari-qa" 
    ssh $host "rm -rf /tmp/hsperfdata_hbase"
    ssh $host "rm -rf /tmp/hsperfdata_hive"
    ssh $host "rm -rf /tmp/hsperfdata_nagios"
    ssh $host "rm -rf /tmp/hsperfdata_oozie"
    ssh $host "rm -rf /tmp/hsperfdata_zookeeper"
    ssh $host "rm -rf /tmp/hsperfdata_mapred"
    ssh $host "rm -rf /tmp/hsperfdata_hdfs"
    ssh $host "rm -rf /tmp/hsperfdata_hcat"
    ssh $host "rm -rf /tmp/hsperfdata_ambari-qa"

    #删除ambari相关包
    ssh $host "yum remove -y ambari-*"
    ssh $host "yum remove -y postgresql"
    ssh $host "rm -rf /var/lib/ambari*"
    ssh $host "rm -rf /var/log/ambari*"
    ssh $host "rm -rf /etc/ambari*"

    echo "$logPre======>$host is done! \n"
done
```

-----

或者


```bash
#1.删除hdp.repo和hdp-util.repo
cd /etc/yum.repos.d/
rm -rf hdp*
rm -rf HDP*
#rm -rf ambari*
#2.删除安装包
#用yum list installed | grep HDP来检查安装的ambari的包
yum remove -y  sqoop.noarch  
yum remove -y  lzo-devel.x86_64  
yum remove -y  hadoop-libhdfs.x86_64  
yum remove -y  rrdtool.x86_64  
yum remove -y  hbase.noarch  
yum remove -y  pig.noarch  
yum remove -y  lzo.x86_64  
yum remove -y  ambari-log4j.noarch  
yum remove -y  oozie.noarch  
yum remove -y  oozie-client.noarch  
yum remove -y  gweb.noarch  
yum remove -y  snappy-devel.x86_64  
yum remove -y  hcatalog.noarch  
yum remove -y  python-rrdtool.x86_64  
yum remove -y  nagios.x86_64  
yum remove -y  webhcat-tar-pig.noarch  
yum remove -y  snappy.x86_64  
yum remove -y  libconfuse.x86_64    
yum remove -y  webhcat-tar-hive.noarch  
yum remove -y  ganglia-gmetad.x86_64  
yum remove -y  extjs.noarch  
yum remove -y  hive.noarch  
yum remove -y  hadoop-lzo.x86_64  
yum remove -y  hadoop-lzo-native.x86_64  
yum remove -y  hadoop-native.x86_64  
yum remove -y  hadoop-pipes.x86_64  
yum remove -y  nagios-plugins.x86_64  
yum remove -y  hadoop.x86_64  
yum remove -y  zookeeper.noarch      
yum remove -y  hadoop-sbin.x86_64  
yum remove -y  ganglia-gmond.x86_64  
yum remove -y  libganglia.x86_64  
yum remove -y  perl-rrdtool.x86_64
yum remove -y  epel-release.noarch
yum remove -y  compat-readline5*
yum remove -y  fping.x86_64
yum remove -y  perl-Crypt-DES.x86_64
yum remove -y  exim.x86_64
yum remove -y ganglia-web.noarch
yum remove -y perl-Digest-HMAC.noarch
yum remove -y perl-Digest-SHA1.x86_64
#3.删除快捷方式
cd /etc/alternatives
rm -rf hadoop-etc 
rm -rf zookeeper-conf 
rm -rf hbase-conf 
rm -rf hadoop-log 
rm -rf hadoop-lib 
rm -rf hadoop-default 
rm -rf oozie-conf 
rm -rf hcatalog-conf 
rm -rf hive-conf 
rm -rf hadoop-man 
rm -rf sqoop-conf 
rm -rf hadoop-conf
#4.删除用户
userdel nagios 
userdel hive 
userdel ambari-qa 
userdel hbase 
userdel oozie 
userdel hcat 
userdel mapred 
userdel hdfs 
userdel rrdcached 
userdel zookeeper 
#userdel mysql 
userdel sqoop
userdel puppet
#5.删除文件夹
rm -rf /hadoop
rm -rf /etc/hadoop 
rm -rf /etc/hbase 
rm -rf /etc/hcatalog 
rm -rf /etc/hive 
rm -rf /etc/ganglia 
rm -rf /etc/nagios 
rm -rf /etc/oozie 
rm -rf /etc/sqoop 
rm -rf /etc/zookeeper 
rm -rf /var/run/hadoop 
rm -rf /var/run/hbase 
rm -rf /var/run/hive 
rm -rf /var/run/ganglia 
rm -rf /var/run/nagios 
rm -rf /var/run/oozie
rm -rf /var/run/zookeeper
rm -rf /var/log/hadoop 
rm -rf /var/log/hbase 
rm -rf /var/log/hive 
rm -rf /var/log/nagios 
rm -rf /var/log/oozie 
rm -rf /var/log/zookeeper 
rm -rf /usr/lib/hadoop
rm -rf /usr/lib/hbase 
rm -rf /usr/lib/hcatalog 
rm -rf /usr/lib/hive 
rm -rf /usr/lib/oozie 
rm -rf /usr/lib/sqoop 
rm -rf /usr/lib/zookeeper 
rm -rf /var/lib/hive 
rm -rf /var/lib/ganglia 
rm -rf /var/lib/oozie 
rm -rf /var/lib/zookeeper 
rm -rf /var/tmp/oozie 
rm -rf /tmp/hive 
rm -rf /tmp/nagios 
rm -rf /tmp/ambari-qa 
rm -rf /tmp/sqoop-ambari-qa 
rm -rf /var/nagios 
rm -rf /hadoop/oozie 
rm -rf /hadoop/zookeeper 
rm -rf /hadoop/mapred 
rm -rf /hadoop/hdfs 
rm -rf /tmp/hadoop-hive 
rm -rf /tmp/hadoop-nagios 
rm -rf /tmp/hadoop-hcat 
rm -rf /tmp/hadoop-ambari-qa 
rm -rf /tmp/hsperfdata_hbase 
rm -rf /tmp/hsperfdata_hive 
rm -rf /tmp/hsperfdata_nagios 
rm -rf /tmp/hsperfdata_oozie 
rm -rf /tmp/hsperfdata_zookeeper 
rm -rf /tmp/hsperfdata_mapred 
rm -rf /tmp/hsperfdata_hdfs 
rm -rf /tmp/hsperfdata_hcat 
rm -rf /tmp/hsperfdata_ambari-qa
#5.重置数据库，删除ambari包
#采用这句命令来检查yum list installed | grep ambari
ambari-server stop
ambari-agent stop
ambari-server reset
yum remove -y ambari-*
yum remove -y postgresql
rm -rf /etc/yum.repos.d/ambari*
rm -rf /var/lib/ambari*
rm -rf /var/log/ambari*
rm -rf /etc/ambari*
```

# 参考
http://www.cnblogs.com/ivan0626/p/4221964.html

https://community.hortonworks.com/questions/1110/how-to-completely-remove-uninstall-ambari-and-hdp.html

http://www.cnblogs.com/cenyuhai/p/3287855.html

