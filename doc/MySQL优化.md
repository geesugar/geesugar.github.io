---
title: MySQL优化
date: 2016-06-13 21:48:56
tags:
---

## 项目
### mysql
```
create table user ( id varchar(20) not null primary key, phone varchar(20), name varchar(20), age int, image varchar(1024));
```

## 优化流程
![](http://www.geesugar.com/BlogImg/databaseyouhualiucheng.png)

## innodb
InnoDB线程：4个IO线程（read buffer， write buffer， log thread， insert buffer thread），1个master线程，1个锁监控线程，1个错误监控线程

## 观察周期性变化
- 是否是由于缓存过期引起的周期性变化
```
#!/bin/bash
while true
do
	mysqladmin -uroot -p31415926 ext | awk '/Queries/{q=$4}/Threads_connected/{c=$4}/Threads_running/{r=$4}END{printf("%d %d %d\n", q, c, r)}' >> status.txt
	sleep 1
done

//ab测试 apache
ab -c 50 -n 200000 http://192.168.1.201/index.php

//awk处理status文本，获取当前sql连接数
awk '{q=$q-last; last=$1}{printf("%d %d %d"), q, $2, $3}'
```

## sql语句分析 show processlist（排序，创建临时表）
```
//监控脚本
#!/bin/bash
while true
do
	mysql -uroot -p31415926 -e'show processlist\G' | grep state |sort -rn|  uniq -c | sort -rn 
	usleep 100000  //每秒钟10次
done

//测试脚本
sysbenc --test=oltp
```
### 需要注意的状态：
- stattistics  分析语法
- **copying to tmp table** 
- sending data
- writing to net
- **sorting result**

show profiles;
show profile for query 2;

### 什么时候产生临时表

## 表的优化与列类型选择
- 尽量不要用null （索引大，查询不方便）
explain select * from t4 where name='a'\G;
- enum类型，内部使用整形存储。
- 字段类型优先级 整形> enum > char > blob

### 既然hash查找如此高效，为什么不用hash索引
- hash函数计算后的结果是随机的
- 无法对范围查询进行优化
- 无法利用前缀索引
- 排序也无法优化

## 多列索引的左前缀规则