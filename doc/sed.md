---
title: sed
date: 2016-06-16 14:13:02
tags:
---

## sed是什么
sed是一个“非交互”面向字符流的编辑器，是Linux操作系统中最重要的自动化运维工具

## sed作用
- 在一个或多个文件上自动实现编辑操作
- 编写转换程序
- 自动化运维

## 学习难点
- 理解工作方式
- 正则表达式
- 与shell配合工作
- 编写技巧

## sed流程与原理
### 模式空间与缓存空间
- 模式空间： 处理文件中一行内容的临时缓冲区，处理完后会将该模式空间中的内容打印到标准输出并自动清空该空间中的内容。
- 缓存空间： 另一个缓冲区，不会自动清空也不会主动打印，主要用于sed高级命令处理，该空间是sed的辅助空间。

## 正则表达式

### 元字符：
.        除换行符以外的任意字符
\w		 匹配字母、数字、下划线或汉字
\s		 匹配任意空白符
\d		 匹配数字
\b       匹配单词的开始或结束
^		 字符串的开始
$		 字符串的结束
\        转义字符
### 限定符：
*        重复0次或多次
+        重复1次或更多
？        重复0次或1次
{n}		 重复n次
{n,}	 重复n次或更多
{n,m}	 重复n次到m次
[1-9]    指定范围内的字符

### 反义：
\W		匹配任意不是字母、数字、下划线、汉字的字符
\S		
\D
\B

### 捕获
(exp)   匹配exp，并捕获文本到自动命名的组里
(?<name>ex) 匹配ex，并捕获文本到名称为name的组里
(?:exp)  匹配exp，不捕获匹配文本，也不分配组号
### 贪婪与懒惰（sed不支持）
*？     重复任意次，但尽可能少重复

### sed注意
- sed默认为贪婪模式、不支持懒惰模式
- sed使用正则表达式，需要注意shell特殊字符冲突问题，比如\(exp\)
- sed不支持\d和反义中的\W等表达式
```
a.c    abc a1c a#c a_c a?c a c
a\wc   abc a1c a_c 
a*c    c(匹配) 
a+c    ac aac aaac
a{9}c  aaaaaaaaac
a[^b]c  中间不是b的字符
```

## sed基础命令

### 命令格式
sed [-options] [commands] filename
command格式：
[address-range] [pattern-to-match] [sed-command]
```
sed -n '5,8p' passwd
```
例子：
打印passwd文件中bash结尾的行
```
sed -n '/bash$/p' /etc/passwd
```
删除passwd文件中以test开头的行
```
sed '/^test/d' /etc/passwd
```
打印bash结尾的用户名
```
sed -r -n '/bash$/s/(\w+):.*/\1/p' passwd
sed -r -n '/bash$/s/(\w+):\w:[0-9]+:[0-9]+:.*:(.*):.*/\1 \2/p' passwd
//-r 表示不需要转义, 括号是需要截取
sed -i 's/nologin/jikexuyuan/g' passwd

//第5行插入
sed -i '5i\hello world' passwd
//第5行追加
sed -i '5a\hello world' passwd
//修改第5行
sed -i '5c\hello world' passwd
//转换
echo "abcdefg" | sed 'y/abcdefg/ABCDEFg/'
//打印到第5行
sed '5q' passwd
//获取ip地址
ifconfig eth0  | grep "inet addr" | sed  -r  -n 's/\s+inet addr:([0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}).*/\1/p'
```

## sed自动化实战应用
### 一键安装脚本
要求：一键安装pptpd服务并配置用户为guest密码为123456的账户可以成功连接VPN并访问外网。
需要修改文件：/etc/pptpd.conf
			/etc/pptpd.options
			/etc/ppp/chap-secreats

```
sudo sed -i 's/logwtmp/# logwtmp/' /etc/pptpd.conf
localip=`ifconfig eth0  | grep "inet addr" | sed  -r  -n 's/\s+inet addr:([0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}).*/\1/p' 192.168.1.105`

//双引号是因为要用刀变量localip
sudo sed -i "s/xxxxxx/$localip" /etc/pptpd.conf
```

## 高级应用

### 高级命令
- 空间操作
	h 将模式空间的内容复制到缓冲空间
	H 将模式空间的内容追加到缓冲空间
	g 将缓冲空间复制到模式空间
	G 将缓存空间追加到模式空间
	x 交换缓冲空间和模式空间的内容
- 多行模式
	n 读取下一行到模式空间，覆盖模式空间中的内容
	N 追加下一行到模式空间，改变当前行号
	:a 定义标签a
	ba 返回标签a
	ta  如果s///命令执行成功就返回标签a
	Ta  如果s///命令执行不成功就返回标签a
	
### 实战应用
- 奇偶输出
	seq 10 | sed -n 'p;n'  //打印奇数行
	seq 10 | sed -n 'N;P'   //小写p输出模式空间所有内容，P输出模式空间第一行内容
	seq 10 | sed 'n;d'
	
	//偶数行
	seq 10 | sed -n 'n;p'   //大小写p效果一样，因为n是覆盖
	seq 10 | sed -rn 'N;s/.*\n(.*)/\1/p'  //将下一行内容读出，然后匹配第二行，再打印
- 反向输出
	seq 10 | tac
	seq 10 | sed '1!G;h;$!d'
- 收尾匹配
	合并行
	sed -rn '/nologin$/{:a;N;$!ba};{s/(.*nologin).*/\1/p}' /etc/passwd
### 常用范列
- 编号
	给passwd添加行号
	sed '=' /etc/passwd | sed 'N;s/\n/\t/'
	统计行数
	sed  -n '$=' /etc/passwd
- 操作行
	打印某行内容
	sed -n '5p' /etc/passwd
	打印文件前5行
	sed '5q' /etc/passwd
	输出最后5行
	sed ':a;$q;N;6,$D;ba' /etc/passwd
- 效率
	sed -n '100,200p' filename   //效率慢，操作大文件会处理所有行
	sed -n '201q;100,200p' filename  //效率高