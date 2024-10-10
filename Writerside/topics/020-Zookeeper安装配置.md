# 020-Zookeeper安装配置

‌**ZooKeeper是一个开源的分布式协调服务，设计用于解决分布式系统中经常遇到的一些问题，如数据一致性、系统配置管理、集群管理、分布式同步等。**‌

ZooKeeper的主要功能和特性包括：

- ‌**分布式协调**‌：ZooKeeper提供了一套简单的原语集，用于解决分布式系统中的协调问题，如分布式锁、分布式队列等。它通过维护一个全局的、一致的数据模型，确保所有客户端看到的数据模型是一致的，从而实现了分布式系统中的协调功能‌12。
- ‌**数据一致性**‌：ZooKeeper确保所有的更新操作全局有序，每个更新都有一个唯一的时间戳（zxid），保证了数据的一致性和顺序性。所有的事务请求处理结果在整个集群中所有机器上的应用情况是一致的，即整个集群要么都成功应用了某个事务，要么都没有应用‌13。
- ‌**高可用性**‌：ZooKeeper的设计目标是提供高可用性的服务。它通过复制的方式维护数据，确保即使部分节点故障，系统仍然能够提供服务‌2。
- ‌**可靠性**‌：一旦服务端成功地应用了一个事务，并完成对客户端的响应，那么该事务所引起的状态变更将会一直被保留，除非有另一个事务对其进行了变更‌3。
- ‌**实时性**‌：ZooKeeper保证在一定的时间段内，客户端最终一定能够从服务端上读取到最新的数据状态‌3。
- ‌**客户端支持**‌：ZooKeeper提供了Java和C的接口，方便开发者在不同的编程语言环境中使用‌1。

ZooKeeper最初是Google Chubby的一个开源实现，被广泛用于Hadoop和HBase等分布式系统中，提供了诸如配置维护、域名服务、分布式同步、组服务等关键服务‌1。它是Apache的顶级项目，由Apache软件基金会维护，并作为一个高可用的文件系统使用‌。ZooKeeper的应用场景包括但不限于发布/订阅、负载均衡、命令服务、分布式协调/通知、集群管理等‌。

## 环境准备

- 华为云加购云服务器2台
- 虚拟私有云手动分配ip分别为10.0.0.12和10.0.0.13
- 服务器名称分别为hadoop02和hadoop03

登录服务器后先修改云服务配置，关闭重启后自动添加127.0.0.1本地ip服务，具体方法见笔记008

## 一、下载

下载地址：

https://dlcdn.apache.org/zookeeper/zookeeper-3.8.4/apache-zookeeper-3.8.4-bin.tar.gz

将安装包下载到`/opt/packages`目录下

## 二、安装

```shell
cd /opt/packages
# 解压
tar -zxvf apache-zookeeper-3.8.4-bin.tar.gz -C /opt/modules
# 改名
mv /opt/modules/apache-zookeeper-3.8.4-bin/ /opt/modules/zookeeper-3.8.4
```

## 三、配置

### 配置主机名映射

`vim /etc/hosts`

```tex
#  在打开的文件末尾换行追加以下内容
10.0.0.11    hadoop01
10.0.0.12    hadoop02
10.0.0.13    hadoop03
```

### 配置root用户和hadoop用户在三台机器之间免密登录

在另外两台机器上分别创建hadoop用户，密码1234，不会创建的同学自行百度

```shell
ssh-keygen  # 生成密钥
for i in {04,05,06};do ssh-copy-id hadoop$i;done; # for循环遍历配置免密登录
```

> **注意：**`hadoop`用户和`root`用户都要配置

### 创建日志、数据目录

```shell
cd zookeeper-3.8.4/

mkdir logs
mkdir zkData
```

### 修改配置文件

修改zookeeper的配置文件，`vim conf/zoo.cfg`，打开是个空文件不要紧张，写入下面内容

```python
tickTime=2000
dataDir=/opt/modules/zookeeper-3.8.4/zkData
log4j.configuration=/opt/modules/zookeeper-3.8.4/conf/log4j.properties
clientPort=2181
initLimit=5
syncLimit=2
server.1=hadoop01:2888:3888
server.2=hadoop02:2888:3888
server.3=hadoop03:2888:3888
```

修改日志配置文件`log4j.properties`， `vim conf/log4j.properties`，也是一个空文件，也不要紧张，写入下面内容
```python
# 定义日志输出路径
log4j.appender.file.File=/opt/modules/zookeeper-3.8.4/logs/zookeeper.log
# 定义日志轮循策略，按天轮循
log4j.appender.file.DatePattern='.'yyyy-MM-dd
log4j.appender.file.append=true
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{ISO8601} [%t] %-5p %c{1}:%L - %m%n
```

### 配置myid

在zkData目录下新建myid文件，写入机器id， `vim zkData/myid`

```tex
1
```

没错，文件里就写一个1就可以。

### 配置JAVA_HOME

在`zkServer.sh`中的第29行，配置JAVA_HOME

```shell
JAVA_HOME=/opt/modules/jdk8
```

### 配置环境变量

`vim /etc/profile`， 末尾追加ZK_HOME

```shell
export ZK_HOME=/opt/modules/zookeeper-3.8.4
export PATH=$ZK_HOME/bin:$PATH
```

修改完环境变量要记得`source /etc/profile`一下

### 编写启动脚本

编写一个一键启动脚本`zkctl.sh`，这样就不需要在每台机器上一个一个启动zk，`vim bin/zkctl.sh`

```shell
#!/bin/bash
 
# 在调用shell脚本时，需要传入一个参数，用于标识执行开启或者关闭zk集群的开启关闭和查询状态
 
#判断调用shell脚本时 是否正常的传入参数
 
#参数小于1
if [ $# -lt 1 ]
then
  echo "调用该脚本时需要传入一个参数"
  exit ;
fi
 
#传入的第一个参数 有三种情况
case $1 in 
"start")
	echo "----------启动zk集群----------"
	
	for hostname in hadoop01 hadoop02 hadoop03
do
	echo "-------------$hostname------------"
	ssh $hostname "/opt/modules/zookeeper-3.8.4/bin/zkServer.sh start"
 
done
	
;;
"stop")
	echo "----------关闭hadoop集群----------"
		for hostname in hadoop01 hadoop02 hadoop03
do
	echo "-------------$hostname------------"
	ssh $hostname "/opt/modules/zookeeper-3.8.4/bin/zkServer.sh stop"
 
done
	
;;	
"status")
	echo "----------查询zk集群状态-------------"
		for hostname in hadoop01 hadoop02 hadoop03
do
	echo "-------------$hostname------------"
	ssh $hostname "/opt/modules/zookeeper-3.8.4/bin/zkServer.sh status"
 
done
;;	
*)
	echo "输入的参数不符合脚本运行的规则，请输入start或者stop，status"
;;
esac	
```

### 集群分发

将zookeeper和jdk8分发到每一台机器上，在此之前先在其余两台机器上创建好`/opt/modules`目录

```shell
# 分发zookeeper
scp -r /opt/modules/zookeeper-3.8.4/ hadoop02:/opt/modules/zookeeper-3.8.4/
scp -r /opt/modules/zookeeper-3.8.4/ hadoop03:/opt/modules/zookeeper-3.8.4/
# 分发jdk8
scp -r /opt/modules/jdk8/ hadoop02:/opt/modules/jdk8/
scp -r /opt/modules/jdk8/ hadoop03:/opt/modules/jdk8/
# 分发hosts文件
scp -r /etc/hosts hadoop02:/etc/hosts
scp -r /etc/hosts hadoop03:/etc/hosts
# 分发环境变量
scp -r /etc/profile hadoop02:/etc/profile
scp -r /etc/profile hadoop03:/etc/profile
# 分发sudoers授权文件
scp /etc/sudoers hadoop02:/etc/sudoers
scp /etc/sudoers hadoop03:/etc/sudoers
```

### 集群配置

1-其余两台机器在分发完环境变量后，需要`source /etc/profile`一下。

2-其余两台机器在分发完环境变量后，需要分别将zookeeper中的myid文件中的数字id修改为2和3

## 四、启动

```shell
zkctl.sh start  # 启动
zkctl.sh status  # 查看集群状态