# 017-Hive数据导入导出

## 使用Hive语句导入导出

```sql
// 数据导入
// 简单创建一个内部表
create table if not exists book
(
    id        int    not null,
    book_name string not null,
    price     double not null
) row format delimited fields terminated by ',';
// 查看表数据
select *
from book;

// 从hdfs导入数据
// 这种方式加载数据源文件会消失，因为本质是将文件移动到表所在目录
load data inpath '/ext_tb/test_ext.txt' into table book;
// 查看表数据
select *
from book;
// 从linux本地文件系统导入数据
// overwrite关键字会覆盖原有数据，小心使用
load data local inpath '/opt/data/test_ext1.txt' overwrite into table book;
// 查看表数据
select *
from book;

// 导出数据到hdfs
// 完成后在hdfs上的/export目录下可以查看到数据文件
insert overwrite directory  '/export'
select * from book;

// 导出数据到linux本地文件系统,指定分隔符
// 完成后在linux上的/opt/data/export目录下可以查看到数据文件
insert overwrite local directory  '/opt/data/export'
row format delimited fields terminated by '\t'
select * from book;

```

## 使用Hive CLI导出数据

```shell
cd $HIVE_HOME
# 导出 export数据到/opt/data/export/exp.txt中
bin/hive -e "select * from book;" > /opt/data/export/exp.txt
cat /opt/data/export/exp.txt

```

通过-f可以执行sql文件导出数据

```sql
bin/hive -f exp.sql > /opt/data/export/exp02.txt
cat /opt/data/export/exp02.txt

```

