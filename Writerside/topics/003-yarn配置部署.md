# 003-yarn配置部署

**yarn是hadoop提供的集群资源管理工具。**

## 一、修改配置文件

- mapred-env.sh 
- yarn-env.sh

在这两个文件的最后一行追加JAVA_HOME

```shell
vim /opt/modules/hadoop-3.1.3/etc/hadoop/mapred-env.sh 
# 最后一行追加以下内容
export JAVA_HOME=/opt/modules/jdk8
```

:wq保存退出

```shell
vim /opt/modules/hadoop-3.1.3/etc/hadoop/yarn-env.sh 
# 最后一行追加以下内容
export JAVA_HOME=/opt/modules/jdk8
```

:wq保存退出

- mapred-site.xml
- yarn-site.xml

```shell
vim /opt/modules/hadoop-3.1.3/etc/hadoop/mapred-site.xml 
```

最下方配置信息如下

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

:wq保存退出

```shell
vim /opt/modules/hadoop-3.1.3/etc/hadoop/yarn-site.xml 
```

最下方配置信息如下

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <!-- Resourcemanager运行的节点 -->
    <property>
        <name>yarn.nodemanager.hostname</name>
        <value>hadoop01</value>
    </property>
</configuration>
```

## 二、启动测试

```shell
su hadoop  # 切换到hadoop用户启动
start-all.sh  # 同时启动hdfs和yarn
jps
# 出现以下六个进程表示启动正常，前面的数字表示进程号不要求一致
#6225 SecondaryNameNode
#6005 DataNode
#6570 NodeManager
#5882 NameNode
#6891 Jps
#6445 ResourceManager

```

正常启动后安全组放行8088端口，浏览器地址输入  服务器公网ip:8088访问yarn-web页面查看























