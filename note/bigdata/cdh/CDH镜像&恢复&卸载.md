[TOC]

## 0.如何通过实例恢复集群

 2.1 内网ip

 实例中变化的是内网ip /etc/hosts

2.2 mysql

2.3 cm server

2.4 agent

2.5 mysql的cmf.hosts表 更改hosts表中的ip

2.6 启动server web界面出来

2.7 启动集群所有节点的agent

2.8 web界面启动  cms服务

2.9 web界面启动  cluster服务



## 1.卸载前的规划

关闭集群 及 MySQL服务     

(现在web界面关闭cluster    关Cloudera Management Service   机器上关a'gen't  再关闭server  切到mysqladmin 关闭 mysql   停止阿里云)

从而在阿里云做镜像(相当云备份)   镜像不收费 快照收费



删除部署文件夹 /opt/cloudera*
删除数据文件夹



## 2.如何完美卸载集群

### 2.1 HDFS YARN ZK存储数据目录
/dfs/nn
/dfs/dn
/dfs/snn

/yarn/nm
/var/lib/zookeeper



### 2.2 关闭集群 及 MySQL服务



### 2.3 杀进程(执行2次)

虽然cm已经关了, 但后台还有一些进程.



kill -9 $(pgrep -f cloudera)
pgrep -f cloudera   (找到对应的pid)



### 2.4 卸载

没有进程 但还是有个挂载的.

umount /opt/cloudera-manager/cm-5.16.2/run/cloudera-scm-agent/process



假如无法卸载，夯住了 就强制kill

```
yum install -y lsof
lsof /opt/cloudera-manager/cm-5.16.2/run/cloudera-scm-agent/process
kill -9 $(lsof /opt/cloudera-manager/cm-5.16.2/run/cloudera-scm-agent/process | awk '{print $2}')
```

一定要再df -h 校验一下



### 2.5 删除cloudera部署文件夹

```
rm -rf /opt/cloudera*

#删除存储目录
rm -rf /dfs 	/yarn 	/var/lib/zookeeper

rm -rf /usr/share/cmf
rm -rf /var/lib/cloudera*
rm -rf /var/log/cloudera* (对日志来说 可删可不删)
rm -rf /run/cloudera-scm-agent
rm -rf /etc/security/limits.d/cloudera-scm.conf 
rm -rf /etc/cloudera*
rm -rf /etc/hadoop* /etc/zookeeper /etc/hive* /etc/hbase* /etc/impala /etc/spark /etc/solr /etc/sqoop*
rm -rf /tmp/scm_*   /tmp/.scm_prepare_node.lock
```



### 2.6 全局搜索 删除

```
find / -name '*cloudera*' | while read line; do rm -rf ${line}; done
```





### 2.7MySQL的数据库

```
cmf db--》cm server
amon db--》amon   (amon角色的部署)

mysql> drop database cmf;
Query OK, 47 rows affected (0.60 sec)

mysql> drop database amon;
Query OK, 64 rows affected (0.23 sec)

mysql> drop user cmf;
Query OK, 0 rows affected (0.00 sec)

mysql> drop user amon;
Query OK, 0 rows affected (0.00 sec)

mysql> use mysql;
Database changed
mysql> select user from user;
+-----------+
| user      |
+-----------+
| root      |
| mysql.sys |
| root      |
+-----------+
3 rows in set (0.00 sec)
```





## 3.有个坑需要踩

在这个命令下会出现坑  alternatives 一个组件多版本 动态管理

[root@ruozedata001 alternatives]# alternatives --config hadoop

There is 1 program that provides 'hadoop'.



Selection    Command

*+ 1           /opt/cloudera/parcels/CDH-5.16.2-1.cdh5.16.2.p0.8/bin/hadoop

+2 	       /opt/cloudera/parcels/CDH-5.16.3-1.cdh5.16.3.p0.8/bin/hadoop

Enter to keep the current selection[+], or type selection number: 1
[root@ruozedata001 alternatives]# 

未来，本套机器需要升级 假如CDH5.16.3,就是hadoop命令找不到
修复这个版本切换：  alternatives --config hadoop
http://blog.itpub.net/30089851/viewspace-2128683/