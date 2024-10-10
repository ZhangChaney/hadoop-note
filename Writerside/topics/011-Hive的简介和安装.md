# 011-Hive的简介和安装

## 一、Hive简介

‌**Hive是一个基于Hadoop的数据仓库工具，主要用于进行数据提取、转化、加载（ETL）操作，并能够存储、查询和分析存储在Hadoop中的大规模数据。**‌ Hive通过将结构化的数据文件映射为数据库表，并提供类SQL查询功能，使用户可以通过熟悉的SQL语句来查询数据。其核心是将SQL语句转换为MapReduce任务进行运算，底层由HDFS提供数据存储‌。

Hive的特点和用途包括：

- ‌**数据映射**‌：Hive能够将结构化的数据文件映射为数据库表，使用户可以使用SQL语句进行查询。
- ‌**类SQL查询**‌：Hive提供了类SQL的查询语言HQL，使得熟悉SQL的用户可以方便地查询数据。
- ‌**MapReduce执行**‌：Hive将SQL语句转换为MapReduce任务进行运算，适合处理大规模数据集。
- ‌**离线数据处理**‌：Hive主要用于离线数据处理，处理批量、延迟较高但数据量巨大的任务‌。

![image-20241001192350468](./assets/image-20241001192350468.png)

## 二、相关工具下载地址

**Hive下载地址**

https://archive.apache.org/dist/hive/  课程使用3.1.3版本，找到3.1.3中的apache-hive-3.1.3-bin.tar.gz  

**MySQL驱动包**

https://repo1.maven.org/maven2/mysql/mysql-connector-java/ 课程使用5.1.47版本，在5.1.47中找到mysql-connector-java-5.1.47.jar  

**下载方式同学们自行选择**

- 可以本地下载后上传到服务器/opt/packages目录下
- 可以使用wget命令下载到服务器/opt/packages目录下



## 三、配置Hadoop

在`core-site.xml`文件中配置hadoop代理用户

```xml
    <property>
        <name>hadoop.proxyuser.hadoop.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.hadoop.groups</name>
        <value>*</value>
    </property>
```



## 四、Hive安装

解压hive

```sh
cd /opt/packages  # 进入压缩包存放目录
tar -zxvf apache-hive-3.1.3-bin.tar.gz -C /opt/modules/  # 解压到modules
mv /opt/modules/apache-hive-3.1.3-bin/ /opt/modules/hive-3.1.3  # 原来名字太长了改个名字
```

将mysql驱动的jar包拷贝到hive的lib目录下

```sh
cp mysql-connector-java-5.1.47.jar /opt/modules/hive-3.1.3/lib/
```



## 五、Hive配置

在`profile`文件中追加配置hive的环境变量

```sh
export HIVE_HOME=/opt/modules/hive-3.1.3
```

保存后记得source一下。

在Hive的conf目录内，新建`hive-env.sh`文件，填入以下环境变量内容:

```sh
export HADOOP_HOME=$HADOOP_HOME
export HIVE_CONF_DIR=$HIVE_HOME/conf
export HIVE_AUX_JARS_PATH=$HIVE_HOME/lib
```

在Hive的conf目录内，新建`hive-site.xml`文件，填入以下内容:

```xml
<configuration>
    <!-- jdbc连接信息 -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://10.0.0.11:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
    <!-- server2绑定的主机 -->
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>10.0.0.11</value>
    </property>
    <!--  元数据服务绑定 -->
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://10.0.0.11:9083</value>
    </property>
    <!--  api授权认证 -->
    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
</configuration>
```



## 六、格式化

在MySQL中新建名为hive的数据库

```sql
create database hive charset utf8;
```

执行hive格式化命令

```sh
cd /opt/modules/hive-3.1.3

bin/schematool -initSchema -dbType mysql -verbos
```

出现以下内容表示完成

```tex
Initialization script completed
schemaTool completed
```

完成后查看数据库是否生产hive相关的数据表。

**此时可能出现以下错误：**

```sh
Exception in thread "main" java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V
        at org.apache.hadoop.conf.Configuration.set(Configuration.java:1357)
        at org.apache.hadoop.conf.Configuration.set(Configuration.java:1338)
        at org.apache.hadoop.mapred.JobConf.setJar(JobConf.java:518)
        at org.apache.hadoop.mapred.JobConf.setJarByClass(JobConf.java:536)
        at org.apache.hadoop.mapred.JobConf.<init>(JobConf.java:430)
```

**错误原因：**

> 在hadoop目录下的/opt/modules/hadoop-3.1.3/share/hadoop/common/lib文件夹中，存在着guava-27.0-jre.jar这个包，而在/opt/modules/hive-3.1.3/lib文件夹中，存在着guava-19.0.jar这个包，此时两个版本的包发生了冲突，产生错误，因此这里需要把guava-19.0.jar的包删除，统一使用27.0版本

**解决方法：**

```sh
mv $HIVE_HOME/lib/guava-19.0.jar $HIVE_HOME/lib/guava-19.0.jar.bak
cp $HADOOP_HOME/share/hadoop/common/lib/guava-27.0-jre.jar $HIVE_HOME/lib
```

创建日志文件夹

```sh
mkdir /opt/modules/hive-3.1.3/logs
```

最后将hive目录修改为hadoop用户所有

```sh
chown -R hadoop:hadoop /opt/modules/hive-3.1.3
```



## 七、启动

```sh
su hadoop
# 先启动hadoop， hive依赖于hadoop
start-all.sh
```

启动hive元数据管理服务

```sh
# 前台启动（一般不用）
# bin/hive --service metastore
# 后台启动
nohup bin/hive --service metastore >> logs/metastore.log 2>&1 &
# 此时使用jobs命令可以看到后台运行的hive进程
jobs
# [1]+  Running                 nohup bin/hive --service metastore >> logs/metastore.log 2>&1 &
```

启动客户端

```sh
# 两种启动方式
# Hive Shell方式，可以直接写SQL，先选择这种方式
bin/hive
# hiveserver2模式， 不可以直接写SQL，需要外部调用,先不使用
# nohup bin/hive --service hiveserver2 >> logs/hiveserver2.log 2>&1 &
```

在CLI窗口执行sql语句`show databases;`，出现一个default的数据库则表示启动成功。

C:\Users\HUAWEI\Documents\TencentMeeting\2024-10-10 11.34.57 chaney的个人会议室 9903385961
