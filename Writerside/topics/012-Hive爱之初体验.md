# 012-Hive爱之初体验

- 启动hadoop
- 启动hive元数据管理服务
- 进入hive shell

```hive
create database if not exists test;

use test;

create table if not exists test
(
    user_id  int,
    username string not null,
    password string not null
);

insert into test (user_id, username, password) values (1, '孙悟空', '111111');
insert into test values (2, '赵子龙', '999999'), (3, '鲁智深', '888888'), (4, '贾宝玉', '666666');

select * as count from test;
select count(*) as count from test group by username;
```

数据插入和分组查询会转成mapreduce程序运行，可以打开yarn管理界面(hadoop01:8088)查看。

![image-20241001231525891](./assets/image-20241001231525891.png)

同时可以打开hdfs的管理界面(hadoop:9870)，可以看到这hdfs多出了一个/user目录。

