---
title: awk
date: 2016-06-13 21:49:34
tags:
---
## awk简介
完整的独立语法结构的编程语言。
### 特点
- 功能强大
- 完整语法（函数，流程控制）
- 进程控制
- 简短高效

### 用处
- 自动化运维
- 文本处理

## 基本语法
- 书写格式
	- 命令行格式
		```
		awk -F: '{print $1}' /etc/passwd
		```
	- 文件格式
		```
		#!/usr/bin/awk
		BEGIN{ FS=":"}
		{print $1}
		```
- 变量
	- 常用内置变量
		```
		$0 所有所有字段
		$1~$n 当前第n个字段
		FS 字符分隔符
		RS 输入记录的分隔符（默认回车换行符）
		OFS ORS
		NF 字段个数   awk -F: '{print NF}' /etc/password
		NR 行号
		```
	- 自定义及外部变量
		```
		awk -v host=$HOSTNAME 'BEGIN{print host}'
		```
- 操作符
		```
		~  匹配
		!~ 不匹配
		awk -F: '{$7 ~/^\/bin/{print $0}}' /etc/password
		```
- 输出
		- print  直接打印
		- printf 格式化打印（不换行）

## awk流程控制
### 条件
	if语句：
	```
	if(experssion) action 1; [else action2]
	//例子
	seq 10 | awk '{if($0%2==0){print "OK"} else {print "NO"}}'
	awk -F: '{if($NF=="/bin/bash"){print $0}}' /etc/password
	```
### 循环
	while语句：
	```
	while(expression) action
	//例子
	awk -F: '{i=i;while(i<=NF){printf(" %d:%s")}}'
	```
	for语句：
	for(i=0; i<=10; i++)
	for(obj in array)
### 数组
	与python的list非常类似

## awk函数
### 算术函数
- int(x) 返回x的整数部分
- sqrt（x） x的平方根
- rand()   伪随机数 0~1
- srand(x) 重新建立伪随机种子。
### 字符串函数
- sub, gsub  替换函数
```
	echo "hello world" | awk '{sub("world", "jack"); print $0'}
	echo "hello world world" | awk '{gsub("world", "jack"); print $0'}   //全局替换
```
- index  返回位置
- length(s) 返回字符串或数组长度
- match(s, r) 正则表达式
- split()
```
	echo "00-11-22-33-44" | awk '{split($0, a, "-"); for(i in a){print i":"a[i]}}'
```
- tolower()
- toupper()
### 自定义函数
```
	awk 'function sum(n, m){total=n+m; return total}BEGIN{print sum(5,8)}'
```

## awk实战应用
### 截取IP地址
```
ifconfig eth0 | awk -F':| +' '/inet addr:/{print $4}'
//+作用表示多个连续空格为一个分隔符
```
### 统计网络连接数
```
netstat -an | awk '/^tcp/{state[$NF]++} END {for(key in state){print key"\t"state[key]}}' | column -t
```