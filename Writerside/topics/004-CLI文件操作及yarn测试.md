# 004-CLI文件操作及yarn测试

## 一、CLI操作HDFS

```shell
# 创建目录
hdfs dfs -mkdir -p /opt/data
# 上传文件
hdfs dfs -put /opt/data/wc.input /opt/data/
```



## 二、测试yarn任务管理

```shell
cd /opt/modules/hadoop-3.1.3

hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /opt/data/wc.input /opt/data/wc.output

```

### 1.错误一

成功报错以下内容

```tex
Error: Could not find or load main class org.apache.hadoop.mapreduce.v2.app.MRAppMaster

Please check whether your etc/hadoop/mapred-site.xml contains the below configuration:
```

解决方法：

在etc/hadoop/mapred-site.xml中添加以下配置

```xml
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
    </property>
```

### 2. 错误二

报错信息

```tex
2024-09-19 17:52:34,156 INFO ipc.Client: Retrying connect to server: 0.0.0.0/0.0.0.0:8032. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
```

查看日志

```shell
jps
# 发现resourcemanager直接挂掉了，于是检查resourcemanager的日志
tail -n100 /opt/modules/hadoop-3.1.3/logs/hadoop-hadoop-resourcemanager-hadoop03.log
# 日志报错信息如下
2024-09-19 17:52:21,634 ERROR org.apache.hadoop.yarn.server.resourcemanager.rmapp.RMAppImpl: App: application_1726739460497_0001 can't handle this event at current state
```

问题原因

```tex
内存不足
```

解决方法

```tex
更改规格，改为4G内存
```

### 3. 错误三

成功报错以下内容

```
Container exited with a non-zero exit code 1. Error file: prelaunch.err.
```

解决方式

```shell
hadoop classpath
# 拷贝输出内容添加到yarn-site.xml
/opt/modules/hadoop-3.1.3/etc/hadoop:/opt/modules/hadoop-3.1.3/share/hadoop/common/lib/*:/opt/modules/hadoop-3.1.3/share/hadoop/common/*:/opt/modules/hadoop-3.1.3/share/hadoop/hdfs:/opt/modules/hadoop-3.1.3/share/hadoop/hdfs/lib/*:/opt/modules/hadoop-3.1.3/share/hadoop/hdfs/*:/opt/modules/hadoop-3.1.3/share/hadoop/mapreduce/lib/*:/opt/modules/hadoop-3.1.3/share/hadoop/mapreduce/*:/opt/modules/hadoop-3.1.3/share/hadoop/yarn:/opt/modules/hadoop-3.1.3/share/hadoop/yarn/lib/*:/opt/modules/hadoop-3.1.3/share/hadoop/yarn/*
```

```xml
    <property>
        <name>yarn.application.classpath</name>
        <value></value>
    </property>
```

### 4. 错误四

运行发现卡住不动

```shell
# 卡在这里不动了一直是0%
INFO mapreduce.Job:  map 0% reduce 0%

free -m # 发现内存用完了
```

解决方法

```
更改规格，改为4G内存
```

### 5. 错误五

运行报错

```shell
2024-09-19 20:11:05,727 INFO mapreduce.JobSubmitter: Cleaning up the staging area file:/tmp/hadoop/mapred/staging/hadoop824734288/.staging/job_local824734288_0001
ENOENT: No such file or directory
```

原因

```tex
/tmp目录为root用户所有，hadoop用户无法对其进行操作，将/tmp目录所有者修改为hadoop
```

解决方法

```tex
chown -R hadoop:hadoop /tmp/hadoop
```



## 三、未报错测试成功

在yarn-web和hdfs-web页面可查看运行的任务和计算结果。

在hdfs-web页面中查看/opt/data下是否有wc.output文件夹，并进入查看是否有_SUCCESS标识文件。有则表示成功。

在yarn-web页面中点击Applications，下方查看执行的任务，查看FinalState一栏中是否为SUCCEEDED，是则表示成功，FAILED表示失败。

```shell
# 查看计算结果
hdfs dfs -cat /opt/data/wc.output/part-r-00000
# 输出计算结果
a       2
b       1
c       1
d       2

# 下载结果文件
hdfs dfs -get /opt/data/wc.output ~
# 查看下载文件
ls ~
```

上述均成功后完成伪分布式所有内容（hadoop、hdfs、yarn）的配置。
