---
title:  postgreSQL 日期函数整理
categories: ["Coding"]
tags: ["sql"]
date: 2018-03-19 21:18:41
---


最近在项目中维护某一张表的固定数据的时候，经常会用到postgreSQL的各种日期函数，所以简单整理一下。<!-- more -->
## 获取系统时间的函数
### 获取当前完整时间
* 通过now()获取的时间是最完整的时间，包括时区，秒也保留到了6位小数。
``` 
select now();
-- 得到如下结果
'2018-03-19 21:28:31.545145+08'
```
* current_timestamp 效果是和now()一样的。
``` 
select current_timestamp效果是和now;
-- 得到如下结果
'2018-03-19 21:29:31.545145+08'
```
### 获取当前时间  
* current_time 只显示当前的时间，不包括日期
``` 
select current_time;
  --    得到的结果如下
       21:29:31.545145+08'
```
### 获取当前日期 
* current_date 只显示当前的日期，不包括小时等信息
 ``` 
select current_date;
  --     得到的结果如下
       '2018-03-19'
 ```

## 日期计算函数
###  日期简单的加减
 1. 一年后
```
select now() + interval '1 years';
select now() + interval '1 year'; 
select now() + interval '1 y';
select now() + interval '2 Y';
select now() + interval '2Y'; 
```
这几种写法都OK的。
 2. 一个月后
 ```
select now() + interval '1 month';
 ```
 3. 一周后
```
select now() + interval '1 week';
```
 4. 一天后
```
select now() + '1 day'
```
 5. 一分钟后
```
select now() + '1 min';   
```

interval 可以不写，其值可以是：

| Abbreviation | Meaning                    |
| ------------ | -------------------------- |
| Y            | Years                      |
| M            | Months (in the date part)  |
| W            | Weeks                      |
| D            | Days                       |
| H            | Hours                      |
| M            | Minutes (in the time part) |
| S            |               Seconds             |


### 计算时间差
使用 age(timestamp, timestamp)计算时间差
函数描述：计算两个日期之间相隔多少天，单个参数时表示与当前日期(current_date)相比
参数：age(timestamp,timestamp),age(timestamp)
返回值：interval，两个日期之间的相隔天数

示例：
```
Select age(timestamp '2001-04-10', timestamp '1957-06-13')
Result: 43 years 9 mons 27 days
```
## 时间字段的截取

在开发的过程中，有时候经常会用到日期的年，月，日，小时等值，PostgreSQL给我们 提供了一个非常便利的EXTRACT函数。
```
EXTRACT(field FROM source)
```
field 表示取的时间对象，source 表示取的日期来源，类型为 timestamp、time 或 interval。
* 取年份
```
select extract(year from now());
Result: 2018
```
* 取月份
```
select extract(month from now());
Result: 3
select extract(day from timestamp '1994-08-20');
Result: 8
```
* 查看今天是一年中的第几天
```
select extract(doy from now());
```
* 查看现在距1970-01-01 00:00:00 UTC 的秒数
```
select extract(epoch from now());
```
* 把epoch 值转换回时间戳
```
SELECT TIMESTAMP WITH TIME ZONE 'epoch' + 1369755555 * INTERVAL '1 second'; 
```

## 结语
以上是基本的PG时间/日期函数使用，有些函数因为下班了，家里电脑没环境，没跑结果出来，明天补上。`（明日复明日。。明日何其多。。）`

详细用法请参考：[PostgreSQL官方说明][1]


[1]: http://www.postgresql.org/docs/9.2/static/functions-datetime.html