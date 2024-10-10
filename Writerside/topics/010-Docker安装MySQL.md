# 010-Docker安装MySQL5.7

## 一、拉取MySQL5.7镜像

```bash
$ docker pull mysql:5.7
# 等待镜像拉取完成
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
mysql        5.7       5107333e08a8   5 months ago   501MB
```

## 二、快速启动一个MySQL容器 {id="mysql_1"}

```bash
$ docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7

$ docker ps

CONTAINER ID   IMAGE       COMMAND                  CREATED        STATUS         PORTS                                                  NAMES
812f56f8f239   mysql:5.7   "docker-entrypoint.s…"   4 seconds ago   Up 4 seconds   33060/tcp, 0.0.0.0:3306->3306/tcp, :::3306->3306/tcp   mysql
```

> 参数解析：
>
> - -i：以交互模式运行容器，并保持终端激活。
> - -t（--tty）：为容器分配一个伪终端。
> - -d（--detach）：以后台模式运行容器，并返回容器ID
> - -p（--publish）：发布容器端口到主机端口。常用：端口映射到宿主机
> - --name：为容器指定一个名称。

## 三、挂载数据卷方式启动MySQL容器

### 1.创建数据挂载目录

```bash
$ mkdir -p /opt/software/mysql/mysql-01/conf
$ touch /opt/software/mysql/mysql-01/conf/my.cnf
$ mkdir -p /opt/software/mysql/mysql-01/data
$ mkdir -p /opt/software/mysql/mysql-01/logs
```

### 2.编写my.cnf配置文件

> [mysqld]
>
> #skip-grant-tables
>
> skip-host-cache
> skip-name-resolve
> datadir=/var/lib/mysql
> socket=/var/run/mysqld/mysqld.sock
> secure-file-priv=/var/lib/mysql-files
> user=mysql
>
> symbolic-links=0
>
> pid-file=/var/run/mysqld/mysqld.pid
> [client]
> socket=/var/run/mysqld/mysqld.sock
>
> !includedir /etc/mysql/conf.d/
> !includedir /etc/mysql/mysql.conf.d/

### 3.挂载启动容器

```bash
$ docker run -p 3306:3306 --name mysql --restart=always --privileged=true \
-v /opt/software/mysql/mysql-01/logs:/var/log/mysql \
-v /opt/software/mysql/mysql-01/data:/var/lib/mysql \
-v /opt/software/mysql/mysql-01/conf/my.cnf:/etc/my.cnf \
-v /etc/localtime:/etc/localtime:ro \
-e MYSQL_ROOT_PASSWORD=123456 -itd mysql:5.7
```

> 参数解析：
>
> - --restart=always：总是跟随docker启动
> - --privileged=true：获取宿主机root权限
> - -e MYSQL_ROOT_PASSWORD=123456：指定mysql环境变量，root用户的密码为123456
> - -v /usr/local/mysql/data:/var/lib/mysql：映射数据目录，宿主机:容器


### 4.查看容器状态，测试链接

```bash
$ docker ps
```

> CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS         PORTS                                                  NAMES
> 9416e7e20cde   mysql:5.7   "docker-entrypoint.s…"   13 minutes ago   Up 3 minutes   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql-01

```bash
$ docker logs mysql-01
```

> 2024-05-23T09:02:28.555882Z 0 [Note] Event Scheduler: Loaded 0 events
> 2024-05-23T09:02:28.556181Z 0 [Note] mysqld: ready for connections.
> Version: '5.7.44'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)

```bash
$ docker exec -it mysql-01 mysql -uroot -p123456
```

> Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
>
> mysql> exit
>
> Bye





## 5.MySQL无法登录问题

```
"Host '106.6.72.227' is not allowed to connect to this MySQL server.

ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```

将`my.cnf`配置文件中的`#skip-grant-tables`取消注释后重启容器

```shell
docker restart mysql
```

重启后登录mysql执行以下操作

```sql
use mysql;
update user set host = '%' where user = 'root';
flush privileges;
exit;
```

再将`my.cnf`配置文件中的`skip-grant-tables`注释后重启容器









