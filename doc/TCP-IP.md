---
title: TCP/IP
date: 2016-06-21 07:41:18
tags:
---
## TCP/IP
### 网络编程
#### 接收连接过程
tcp需要处理的问题（丢包、重复、延迟）
- accept建立连接过程，三次握手
1. 内核处理流程
![](http://www.geesugar.com/BlogImg/sanciwoshou.png)
syn和accept不是无限长度，和调用listen函数参数backlog有关，第一步、第二步不可控，第三步由应用程序控制。
若syn队列满，则会直接丢弃请求。
若accept队列满，则不会导致放弃连接，也不会把连接从syn队列中移出。
2. 阻塞模式接收过程
![](http://www.geesugar.com/BlogImg/zusaimoshishi.png)
3. 非阻塞模式接收过程
![](http://www.geesugar.com/BlogImg/feizusaimoshi.png)
4. tcp消息发送过程
![](http://www.geesugar.com/BlogImg/tcpxiaoxiguocheng.png)
问题：
- 发送成功返回时,意味着消息到达目标主机？
	否定,必须等待发送数据对应序列号的应答才算目标主机成功接收消息，而read和write不处理该信息，只是内核保证将把消息发送给目标主机
- 发送成功返回时，意味着消息发送到网络上？
- 阻塞和非阻塞对发送方法的影响是什么？
- MSS与MTU是什么关系？ 
	MSS(最大报文长度，避免IP层分片，在3次握手时互相告知对方，mtu长度1500 - 头 = 1460字节)
- 为何会有TCP层分片和IP层分片现象
- MSS的值什么时候确定？
- MSS的值会发送改变么？
- TCP是如何保证可靠传输的?

滑动窗口，最大不超过3次握手定义的大小，tcp push在发送数据时，会和滑动打交道
慢启动，拥塞窗口：
nagle算法：在应用进程发送方法时，小的tcp报文会增加网络拥塞的可能 

- tcp消息接收过程
问题：阻塞和非阻塞对接收消息的影响是什么？
阻塞模式下，函数标志位参数对接收有影响没？
网卡接收到的消息如何分发到对应TCP连接上？
消息到达在消息之前，内核如何处理？
并发访问时，内核如何处理？

场景1：
1、网卡先收到消息，用户后调用接收函数
2、套接字为阻塞模式
3、接收低水位为默认值1
4、没有使用prequeue队列
![](http://www.geesugar.com/BlogImg/tcpjieshouxiaoxi.png)

场景2：
1、用户先调用接收函数，网卡后接收消息
2、套接字为阻塞模式
3、接收低水位为默认值1
4、涉及了prequeue队列
![](http://www.geesugar.com/BlogImg/tcpjieshouxiaoxi1.png)

场景3：
1、网卡先接收到消息，用户后调用接收函数
2、套接字为阻塞模式
3、设置接收低水位为n
4、没有使用prequeue队列
![](http://www.geesugar.com/BlogImg/tcpjieshouxiaoxi2.png)


- tcp连接关闭过程
1、多线程共享的连接如何关闭？
2、关闭连接时，如何处理消息？
3、so_linger选项的功能是什么？

close函数
缺点：
1、只是将套接字引用技术减一
2、close会终止TCP的双工链路

shutdown函数
1、 不管引用计数就激发TCP的正常连接终止序列
2、 利用shutdown函数可以关闭全双工的任意一端
3、 so_linger选项功能：（谨慎运用）
    可以确保可靠性传输
	可以定时关闭连接
	控制close的阻塞时长


### TCP超时重传
- TCP模块为每个TCP报文段维护一个重传定时器
- 如果超时，则进行重传，重新设置定时器

### TCP拥塞控制过程
提高网络利用率、降低丢包率、拥塞控制
在发送方可以通过滑动窗口进行流量控制
拥塞控制：发送窗口SWND，窗口小（延迟），窗口大（拥塞）
		拥塞窗口CWND，
拥塞控制分为四部分：
- TCP慢启动
	只要网络没有出现拥塞，就增大拥塞窗口，加倍，当主机开始发送数据时，如果将大量字节注入网络，可能引起网络拥塞，可以从小到大来试探网络情况，逐步增大发送端的拥塞窗口。阈值，小于阈值采用拥塞控制算法，大于阈值采用拥塞避免算法。
- 拥塞避免
	为了让拥塞窗口缓慢加大，每次加一。

拥塞发生后的处理过程：
	收到三个重复的确认处理过程，拥塞窗口减半，快速重传，快速恢复
	收到1个重复的确认处理过程，执行拥塞避免算法
	收到新数据确认时的处理过程。

- 快速重传
- 快速恢复

## 流量控制：
所谓流量控制,让发送方发送速率不要太快，要让接收方来得及接收。流量控制可以有效防止网络中瞬间数据带来的冲击。
发送窗口不能超过接收窗口的大小。

## 拥塞控制
网络拥塞现象是发送端发送给接收端的网络分组过多，使该部分网络来不及处理。以致引起部分至整个网络性能下降的现象，严重时甚至会导致网络通信业务陷入停顿，即出现死锁现象。

## 网络套接字
### SOL_SOCKET 协议簇选项
getsockopt(SOL_SOCKET)
setsockopt(SOL_SOCKET)
level： SOL_SOCKET   IPPROTO_IP  IPPROTO_TCP(最大重传事件等)
SO_KEEPALIVE： 保持连接选项，2小时没有数据交互，发送探测报文，有三种回应：
1.回应一个ACK报文
2、回应一个RST报文
3、没有任何回应（在75秒内发送8个探测报文，然后关闭socket）、

SO_LINGER: 设置TCP关闭连接后的行为
struct linger{
	int l_onoff;		//开启，关闭
	int l_linger;       //滞留时间
}
l_onoff=0； 此时l_linger不起作用
l_onoff不为0， l_linger为0；  close立即返回

SO_RCVBUF SO_SNDBUF缓冲区大小
1、TCP和UDP连接中的含义有所不同，TCP缓冲区大小就是滑动窗口大小
2、connect()系统调用之前设置

SO_RCVTIMEO 接收超时
SO_SNDTIMEO 发送超时

SO_RCVLOWAT SO_SNDLOWAT 低水位标记





