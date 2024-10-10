# 019-Hive常用函数

## 技能要求

Hive函数在ETL工程、数仓构建中起非常关键的作用。

官方文档地址：https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-Built-inFunctions

掌握基本的七类Hive内置函数的使用

- 数值函数
- 日期函数
- 集合函数
- 类型转换函数
- 字符串函数
- 条件函数
- 脱敏函数



## 数值函数

```sql
// round() 取整函数，可以指定精度，四舍五入
select round(5.21548963);
select round(5.21548963, 2);
// rand() 取随机数， 可以设置种子
select rand();
select rand(1);
// abs() 取绝对值
select abs(-1);
// log() 取对数，e为底数 
select log(8);
```



## 日期函数

```sql
// current_timestamp()  当前时间戳
select current_timestamp();

// current_date()  当前的日期
select current_date();

// to_date() 时间戳转日期
select to_date(current_timestamp());

// year(), month(), day()
select year('2024-10-01');

// datediff() 日期间隔计算
select datediff('2024-10-07', '2024-10-01');

// data_add(), data_sub()  加减
select data_add('2024-09-30', 1);
select data_sub('2024-09-30', 1);
```



## 集合函数

> 查看负责数据类型部分笔记。



## 类型转换函数

```sql
// binary()  字符串转二进制
select binary('hadoop');

// cast() 自由类型装换
select cast('1' as int);
```



## 字符串函数

```sql
// 拼接
select concat('123', '456');
select concat_ws('--', '123', '456'); // 可以指定分隔符

// length

// lower, upper

// trim

// split
```



## 条件函数

```sql
// isnull()  isnotnull()
select isnull('');
select isnotnull('');

// if判断
select if(1, 'true', 'false');
select if(0, 'true', 'false');

// nvl(arg1, arg2) ，false返回第一个参数，true返回第二个参数，

// case .. when .. then .. end 语法等同mysql
```



## 脱敏函数

```sql
// hash加密
select mask_hash('hadoop');

// md5加密
select md5('hadoop');
```

