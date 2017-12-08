---
title: server
date: 2016-06-18 20:39:41
tags:
---

## 例子
接入层，（维护长连接，业务逻辑少，初步攻防：连接频率, 过滤，压缩与解压缩，黑白名单）
逻辑处理层（接口有限：登录，登出，keepalive，）
存储（内存存储，数据库存储）

拔掉网线，可能客户端和业务端都无法捕捉到epollin，epollout, epollerror事件，每30s发送心跳包到客户断，可以纠正tcp状态

可扩展性，一致性，可靠性
![](http://www.geesugar.com/BlogImg/58tongcheng.png)

## c++11
std::move()
![](http://www.geesugar.com/BlogImg/zhuanyigouzhaohanshu.png)
### 转移构造函数
![](http://www.geesugar.com/BlogImg/zhuanyigouzhaohanshu1.png)
### 左值引用 & 右值引用
右值引用可以作为临时对象的引用
vector<int> input_value();

vector<int>& lref = input_value() //编译错误
const vector<int>& lref = input_value() //旧标准

vector<int>&& lref =  input_value()； //右值引用，不会复制，是用转移构造函数

转移构造函数的优点
1、转移比复制快
2、有些对象只能转移，不能复制
3、。。。

新的模版类型std::unique_ptr替代std::auto_ptr成为标准库中的自动内存管理类型，支持转移构造，且不支持复制构造
vector<auto_ptr<isteam> > input_streams; //不能这样实现，因为istream不支持复制构造，模版类型参数必须支持复制构造，所以auto_ptr不能这样引用
vector<unique_ptr替代std<isteam> > input_streams; //可以实现，vecotor默认使用转移构造，如果转移构造没有再调用复制构造

### 易用性改善 lambda std::function auto关键字 for_each循环
![](http://www.geesugar.com/BlogImg/stlsuanfa.png)
lambda就是函数对象
![](http://www.geesugar.com/BlogImg/lambda.png)
![](http://www.geesugar.com/BlogImg/lambda1.png)
![](http://www.geesugar.com/BlogImg/lambda2.png)
![](http://www.geesugar.com/BlogImg/lambda3.png)

## MQTT


## Reactor模型
主线程只监听文件描述符事件是否发生，IO处理单元，如果有事件发生将其交给工作线程处理（逻辑单元）。
- 主线程注册就绪事件
- 主线程等待连接上的读就绪事件
- 有可读事件，将其放入请求队列
- 工作线程处理请求队列中的事件
- 主线程等待连接可写
- 可写时，主线程将可写时间放入请求队列
- 工作线程往socket上写入服务器处理客户请求

![](http://www.geesugar.com/BlogImg/reactormoxing.png)

## Proactor模型
工作线程仅仅复制业务逻辑，所有IO操作交给内核和主线程（信号），主线epoll_wait(）仅能够监听socket连接请求事件，而不能监听读写事件，这是与reactor的差别
- 主线程注册socket上的读完成事件
- 主线程处理其它逻辑
- 读事件完成，发送信号
- 应用程序调用信号处理函数
- 主线程继续其它逻辑
- 写入数据时，发送信号
- 应用程序使用信号处理函数来善后

![](http://www.geesugar.com/BlogImg/prereactormoxing.png)

同步I/O
![](http://www.geesugar.com/BlogImg/tongbupreactor.png)


共性：
- 在线程或者监听socket上调用epoll_wait
- 工作线程处理连接上的事件
- 主线程和工作线程的沟通通过工作队列

## 高效的并发模型
并发模型特点
- 适用于I/O密集型
- 有多线程和多进程两种
- I/O处理单元和多个逻辑单元协调完成任务

## 提高服务器性能的方法
- 创建池
	内存池
	进程池
	线程池
- 减少数据复制
- 减少上下文切换和锁

## 服务器调试技术
### 调整系统参数的介绍
- 修改最大文件描述符 ulimit -n;  ulimt -SHn max-file-number
和文件系统相关的内核参数
/proc/sys/fs/file-max
/proc

### GDB调试器的介绍
调试线程
thread info
设置线程阻塞还

### 系统检测工具介绍
lsof   列出当前系统文件描述符
-i socket  -u 指定用户指定进程打开的描述符 

netcat  快速构建网络连接，
-i 间隔 -s  -u udp -z 扫描端口
nc -z test-server 2--50
nc -x test2:1080 -X www.baidu.com 80

vmstat 实时输出系统资源使用情况
top  free

ifstat  网络信息