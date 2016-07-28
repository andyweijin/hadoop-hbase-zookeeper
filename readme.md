# hadoop-hbase-zookeeper  环境搭建

## 运行环境

> centos6.6-86_64 x 2

## 安装步骤

> 运行环境

|num | ip | hostname |
|---|:---:|:---:|
|1|192.168.111.11|hadoop-master|
|2|192.168.111.12|hadoop-slave|    

> ssh 免密登录，也可以自己登录自己

> 添加用用户

```shell
# groupadd hadoop
# useradd -g hadoop hadoop
```

 - 修改hosts

```shell
# vim /etc/hosts

```


> > 避免闭环，slave连不上master上面的namenod

> > 注释掉 127.0.0.1 hadoop-master 或 hadoop-slave

> > 注释掉 ::1 hadoop-master 或 hadoop-slave

```

 - install openjdk （devel 版本带jps命令，两台机器都需要执行）

```
# yum install java-1.7.0-openjdk-devel.x86_64
```


> java 环境变量

```
# vim /etc/profile
```

> > export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk.x86_64

> > export JRE_HOME=/usr/lib/jvm/jre-1.7.0-openjdk.x86_64

> > export HADOOP_HOME=/opt/hadoop-1.2.1

> > export HADOOP_HOME_WARN_SUPPRESS=1

> > export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib

> > export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_HOME/bin:$PATH


 - hadoop 安装

```
# chown -R hadoop:hadoop hadoop-1.2.1
# cd hadoop-1.2.1/conf
```

```
# vim hadoop-env.sh
```

> > export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk.x86_64

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

> > hadoop-master

```shell
# vim slaves
```

> > hadoop-master
> > hadoop-slave

> 将master上面的hadoop配置同步到slave上面 (注意用户为hadoop)

```shell
scp -r /opt/hadoop-1.2.1 hadoop-slave:/opt/
```

> 启动测试 （启动和停止的操作只能在master上面执行）

```shell
# hadoop namenode -format
# cd /opt/hadoop-1.2.1/bin
#./start-all.sh
```

## 测试

> master上面执行：jps

```
SecondaryNameNode
JobTracker
TaskTracker
NameNode
DataNode
Jps
```

>slave 上面执行：jps

```
TaskTracker
DataNode
Jps
```














	
~                 