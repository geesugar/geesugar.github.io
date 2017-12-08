---
title: I/O
date: 2016-06-21 18:24:20
tags:
---
## muduo libevent nginx

## select()  poll()
### I/O复用场景
- 客户端需要同时处理多个socket
- 客户端呢需要同时处理用户输入和网络连接
- 服务器需要同时处理监听socket和连接socket
- 服务器需要同时处理TCP请求和UDP请求
- 服务器需要同时监听多个端口

### select()

### poll()


## epoll()
linux特有I/O复用函数，是一组函数来实现的。内核维护事件表，因此不需要像select每次都赋值描述符，用户态到内核态的拷贝很耗时间

int epoll_create(int size)  //返回描述符，size多少个事件
int epoll_ctl(int epfd, int op,  int fd, struct epoll_event* event)
//op操作类型三种， 注册事件，修改事件，删除事件
struct epoll_event{
	int32 events; //epoll时间表
	epoll_data data; 
}
int  epoll_wait(int epfd, struct epoll_event* events, int maxevents, int timeout); //返回数目

```
int ret = poll(fd, MAX_SOCKET_NUMBERS, -1);
for(int i=0; i<MAX_SOCKET_NUMBERS; i++){
	if(fd[i].revents | POLLIN){
		int sockfd = fd[i].fs;
	}
}

int ret = epoll_wait(epollfd, events, MAX_SOCKET_NUMBERS, -1);
for(int i=0; i<ret; i++){
	int sockfd = events[i].data.fd;
}
```

![](http://www.geesugar.com/BlogImg/epoll.png)

## I/O复用模型的使用 