[TOC]



bin 在线 网络带宽
rpm 离线部署 相应下载依赖的rpm包  
    不是真正的离线部署(需要访问外网或者私服)
tar 真正离线部署



CDH真正离线部署:
1.MySQL离线部署  元数据
mysql-5.7.11-linux-glibc2.5-x86_64.tar.gz

2.CM离线部署 tar 闭源 cloudera核心产品 主从架构 server agent
cloudera-manager-centos7-cm5.16.1_x86_64.tar.gz

3.Parcel文件离线源部署 hdfs yarn hive hbase zk
CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel
CDH-5.16.1-1.cdh5.16.1.p0.3-el7.parcel.sha1
manifest.json



## 1.配置

阿里云3台服务器 CentOS7.2
按量付费 2核8G



## 2.软件包
jdk-8u45-linux-x64.gz Oracle 
mysql-connector-java-5.1.47.jar   maven



##  3.节点初始化
### 3.1 hosts文件
172.16.153.168 hadoop001
172.16.153.169 hadoop002
172.16.153.167 hadoop003



### 3.2 防火墙
云: 关闭 + web防火墙
内部服务器 : 
	防火墙关闭  部署  ，等部署好之后，通过CDHweb界面提供的端口，我们将防火墙开启，设置这些端口通过 (文档)

```
[root@hadoop001 ~]# systemctl stop firewalld
[root@hadoop001 ~]# systemctl disable firewalld
[root@hadoop001 ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
[root@hadoop001 ~]# iptables -F
[root@hadoop001 ~]# 
```





### 3.3 selinux
```
[root@hadoop001 ~]# vi /etc/selinux/config 

# This file controls the state of SELinux on the system.

# SELINUX= can take one of these three values:

# enforcing - SELinux security policy is enforced.

# permissive - SELinux prints warnings instead of enforcing.

# disabled - No SELinux policy is loaded.

SELINUX=disabled

# SELINUXTYPE= can take one of three two values:

# targeted - Targeted processes are protected,

# minimum - Modification of targeted policy. Only selected processes are protected.

# mls - Multi Level Security protection.

SELINUXTYPE=targeted
```



重启生效



### 3.4 时区和时钟同步 
timedatectl set-timezone Asia/Shanghai

yum install -y ntp
hadoop001 时间同步的主节点
hadoop002--003 时间同步的从节点

云主机:  时区和时钟同步 不需要做
非云主机:  
从节点 /usr/sbin/ntpdate hadoop001  同步



### 3.5 jdk
/usr/java
解压之后的用户及用户组的修正



## 4.离线部署mysql

```
hadoop001:mysqladmin:/usr/local/mysql/data:>cat hostname.err |grep password
2019-05-11T04:04:37.181848Z 1 [Note] A temporary password is generated for root@localhost: a*D(Jys*_9(o
hadoop001:mysqladmin:/usr/local/mysql/data:>

mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
mysql> alter user root@localhost identified by 'ruozedata123';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'ruozedata123' ;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql>  flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> 
```



 ## 5.事先创建db 用户
```
create database cmf default character set utf8;
GRANT ALL PRIVILEGES ON cmf.* TO 'cmf'@'%' IDENTIFIED BY 'ruozedata' ;

create database amon default character set utf8;
GRANT ALL PRIVILEGES ON amon.* TO 'amon'@'%' IDENTIFIED BY 'ruozedata' ;
flush privileges;
```



## 6.部署 mysql jdbc jar
cmf
amon 
都要部署在hadoop001节点上  那么此节点就要部署jar

```
[root@hadoop001 cdh5.16.1]# mkdir -p /usr/share/java
[root@hadoop001 cdh5.16.1]# cp mysql-connector-java-5.1.47.jar /usr/share/java/mysql-connector-java.jar
```



## 7.离线部署 cm
解压
agent配置 hadoop001-hadoop003
server配置 hadoop001
创建cloudera-scm
文件夹用户用户组修改



## 8.parcel文件离线源 hadoop001



## 9.所有节点创建大数据软件的安装目录 用户和用户组

```
[root@hadoop001 parcel-repo]# mkdir -p /opt/cloudera/parcels
[root@hadoop001 parcel-repo]# chown -R cloudera-scm:cloudera-scm /opt/cloudera
[root@hadoop001 parcel-repo]# 
```



## 10.启动server及agent

```
hadoop001:
/opt/cloudera-manager/cm-5.16.1/etc/init.d/cloudera-scm-server start

等待1min 看日志出现7180
/opt/cloudera-manager/cm-5.16.1/log/cloudera-scm-server/cloudera-scm-server.log
2019-05-11 12:47:50,705 INFO WebServerImpl:org.mortbay.log: Started SelectChannelConnector@0.0.0.0:7180
2019-05-11 12:47:50,705 INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server.


hadoop001-003:
/opt/cloudera-manager/cm-5.16.1/etc/init.d/cloudera-scm-agent start

```



 web界面 防火墙开启7180端口
http://47.111.246.61:7180 admin/admin



## 11.web界面部署

```
echo never > /sys/kernel/mm/transparent_hugepage/defrag 
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```



数据库连接不上可能出现的3个问题:

1.%
2.jdbc  jar
3.flush 