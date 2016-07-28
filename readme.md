# hadoop-hbase-zookeeper  环境搭建

### 运行环境

|num |system| ip | hostname |
|---|:---:|:---:|:---:|
|1|centos6.6-86_64 |192.168.111.11|hadoop-master|
|2|centos6.6-86_64 |192.168.111.12|hadoop-slave|    

### 免密登录，也可以自己登录自己(ssh 登录配置此处略)

### 添加用户

```shell
# groupadd hadoop
# useradd -g hadoop hadoop
```

### 修改hosts

```shell
# vim /etc/hosts

```


> 避免闭环，slave连不上master上面的namenod

>  注释掉 127.0.0.1 hadoop-master 或 hadoop-slave

>  注释掉 ::1 hadoop-master 或 hadoop-slave


### install openjdk （devel 版本带jps命令，两台机器都需要执行）


```shell
# yum install java-1.7.0-openjdk-devel.x86_64
```


### java 环境变量

```shell
# vim /etc/profile
```

> export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk.x86_64

> export JRE_HOME=/usr/lib/jvm/jre-1.7.0-openjdk.x86_64

> export HADOOP_HOME=/opt/hadoop-1.2.1

> export HADOOP_HOME_WARN_SUPPRESS=1

> export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib

> export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_HOME/bin:$PATH

 
### hadoop 安装

```shell
# mv hadoop-1.2.1 /opt/
# mv hbase-0.94.27 /opt/
# mv zookeeper-3.4.6 /opt/ 
# chown -R hadoop:hadoop /opt
# cd /opt/hadoop-1.2.1/conf
```

```shell
# vim hadoop-env.sh
```

> export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk.x86_64

```shell
# vim core-site.xml
```

```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/hadoop</value>
        <description>A base for other temporary directories</description>
    </property>
    <property>
        <name>dfs.name.dir</name>
        <value>/data/hadoop/name</value>
    </property>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://hadoop-master:9000</value>
    </property>
</configuration>
```

```shell
# vim hdfs-site.xml
```

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value> # 计划用两个datanode（master和slave上面都有）
    </property>
    <property>
        <name>dfs.data.dir</name>
        <value>/data/hadoop/data</value>
    </property>
</configuration>
```

```shell
# vim mapred-site.xml
```

```xml
<configuration>
    <property>
        <name>mapred.job.tracker</name>
        <value>hadoop-master:9001</value>
    </property>
    <property>
        <name>mapred.system.dir</name>
        <value>/data/hadoop/mapred_system</value>
    </property>
    <property>
        <name>mapred.local.dir</name>
        <value>/data/hadoop/mapred_local</value>
    </property>
</configuration>
```

```shell
# vim masters
```

> hadoop-master

```shell
# vim slaves
```

> hadoop-master

> hadoop-slave


### zookeeper 搭建(注意 dataDir 的容量)

```shell
# cd /opt/zookeeper-3.4.6/conf
# vim zoo.cfg
```


```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
# dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

dataDir=/home/hadoop/tmp/zookeeper/data
server.1=hadoop-master:7000:7001
server.2=hadoop-slave:7000:7001
```


### 将master上面的hadoop配置同步到slave上面 (注意用户为hadoop)

```shell
scp -r /opt/hadoop-1.2.1 hadoop-slave:/opt/
scp -r /opt/zookeeper-3.4.6 hadoop-slave:/opt/
```

### 启动 （启动和停止的操作只能在master上面执行）

```shell
# hadoop namenode -format
# cd /opt/hadoop-1.2.1/bin
# ./start-all.sh
```

### 测试

- master上面执行：jps

```
SecondaryNameNode
JobTracker
TaskTracker
NameNode
DataNode
Jps
```

- slave 上面执行：jps

```
TaskTracker
DataNode
Jps
```


---
---

# 基于 hadoop 搭建 hbase 


```shell
# cd /opt/hbase-0.94.27/conf
# vim hbase-env.sh
```

> export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk.x86_64


```shell
# vim hbase-site.xml
```

```xml
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://hadoop-master:9000/hbase</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.master</name>
        <value>hadoop-master</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>hadoop-master,hadoop-slave</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/home/hadoop/tmp/zookeeper/data</value>
    </property>
</configuration>
```




```shell
scp -r /opt/hbase-0.94.27 hadoop-slave:/opt/
```  

 - 重新格式化文件系统 (可以看到一个hbase的文件目录)

```shell
# hadoop namenode -format
```

 - 重启 hadoop 集群

```shell
# cd /opt/hadoop-1.2.1
# bin/stop-all.sh
# bin/start-all.sh
```

### 测试

- master上面执行：jps

```
DataNode
HRegionServer
HMaster
TaskTracker
NameNode
Jps
JobTracker
SecondaryNameNode
QuorumPeerMain
```

- slave 上面执行：jps

```
QuorumPeerMain
TaskTracker
DataNode
Jps
```
