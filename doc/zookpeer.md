---
title: zookpeer
date: 2016-07-15 07:20:51
tags:
---

高并发，海量存储
集中式 --> 分布式

## Paxos算法
### 三个角色
1. proposer
	提出提案
2. acceptor
	接受提案
3. learner
	学习被批准提案

### 基本语义
1. value(决议)只有被proposer提出后才能被批准
2. 在一次paxos算法中，只批准（chosen）一个value
3. learner只能获取被批准的value




## zookeeper
zookeeper是分布式协调服务，由雅虎创建，是google chubby的开源实现。
zookeeper是高性能的分布式数据一致性解决方案。

## 特点
1、顺序一致性
2、原子性
3、单一视图
4、可靠性
5、实时性
6、高性能。 3台集群可以达到12~13万QPS


## 场景
1、数据发布/订阅
推、拉。发布者将数据发布到zk集群节点上。
2、负载均衡
3、命名服务
数据库表格ID，两种ID：自动增长和UUID
4、分布式协调/通知

## 基本概念
### 集群角色
Leader
Follower
Observer
### 会话
客户端和zookeeper服务器的连接
### 数据节点
1、集群中的一台机器称为一个节点
2、数据模型中的数据单元znode，分为持久节点和临时节点
zookeeper的数据模型是一棵树，树的节点就是znode，znode中可以保存信息。

### 版本
悲观锁，非常严格的锁策略，排他。在事物没有完成之前，下一个事物不能访问相同的资源。
乐观锁，对于数据库我们通常做法是在每个表中增加一个version版本字段，事物修改数据之前先读出数据，版本号也顺势读出，在更新操作时，update ... set ... where id=1 and verison=1,更新失败表示其他事物也修改了数据，系统抛出异常，让客户端自行处理，客户端可选择重试。

版本类型（三种）：
version： 当前数据节点内容的版本号
cversion：当前数据节点子节点的版本号
aversion：当前数据节点ACL变更版本号

### watcher 事件监听器

### ACL权限控制
五种权限类型：
Create：创建子节点权限
READ：
WRITE：
DELETE：
ADMIN：设置节点ACL的权限。

## zookeeper权限搭建
1、 zoo.cfg
server.1=192.168.1.105:2888:3888
server.2=192.168.1.106:2888:3888
2、 myid
3、 ./zkserver.sh start  启动服务器
4、 telnet .. 2181
	stat     //超过一办机器启动可提供服务，可以查看角色
### 伪集群 单机模式

## zookeeper客户端
./skcli.sh -timeout 0 -r -server ip:port //-r只读模式
./skcli.sh -timeout 5000 -server 192.168.1.105：2181
命令：ls /
	 stat /  //节点的状态信息
	 get
	 ls2
	 create   //-s顺序节点 -e临时节点
	 set     //修改节点  set /node_3 999 2  ;版本号
	 quota   //配额，子节点个数或数据长度限制，当超过配额时，会在bin目录下的zookeeper.out文件会警告日志。

### zookeeper客户端API
1、 jar包导入

```
import org.apache.zookeeper.ZooKeeper;

public class CreateSession implements Watcher {
	private static ZooKeeper zookeeper;
	
	public static void main() throws IOException, InterruptedException{
		zookeeper = new ZooKeeper("192.168.1.105:2181", 5000, new CreateSession());
		
		System.out.println(zookeeper.getState()); //异步，这里是正在连接
		Thread.sleep();
	}
	

	public void process(WatchEvent event){
		System.out.println("收到事件" + event);
	}
}
```

### ZkClient Curator



## 实战
### master选举
服务注册
服务发现
创建master， master选举
所有slaver需要关注master删除事件

1、work服务器注册监听，关注节点的删除事件


![](http://www.geesugar.com/BlogImg/kafka1.png)