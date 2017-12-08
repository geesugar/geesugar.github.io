---
title: boost
date: 2016-06-29 10:14:16
tags:
---

## 编译
g++ http_server.cpp  -levent



## muduo
### 代码
```
#include <muduo/base/Logging.h>
#include <muduo/net/EventLoop.h>
#include <muduo/net/TcpServer.h>
#include <string>

using namespace std;

void onConnection(const muduo::net::TcpConnectionPtr& conn){
    if(conn->connected()){
           LOG_INFO << conn->name(); 
    }    
}

void onMessage(const muduo::net::TcpConnectionPtr& conn, muduo::net::Buffer* buf, muduo::Timestamp time){
    LOG_INFO << "MESSAGE: " << buf->retrieveAllAsString();
}

int main(){
    muduo::net::EventLoop loop;
    muduo::net::InetAddress listenAddr(5432);
    muduo::net::TcpServer server(&loop, listenAddr, "JACK");
    server.setConnectionCallback(onConnection);
    server.setMessageCallback(onMessage);
    server.setThreadNum(3);
    server.start();
    loop.loop();
}
```
### 编译
g++ test.cpp -I/home/jack/work/c++/muduo/build/release-install/include -L/home/jack/work/c++/muduo/build/release-install/lib  -lmuduo_net -lmuduo_base -lpthread

### 测试
echo 1111 | netcat -n 0.0.0.0 5432

## bind
```
#include <boost/bind.hpp>
#include <boost/shared_ptr.hpp>

struct X 
{ 
    bool f(int a) 
    { 
                printf("f %d\n", a);
        return static_cast<bool>(a); 
    }
};

int main(){
    X x; 
    boost::shared_ptr<X> p(new X);
    int i = 5; 
    boost::bind(&X::f, &x, _1)(i);    // (&x)->f(i); 
    boost::bind(&X::f, x, _1)(i);   //(copy x).f(i); 
    boost::bind(&X::f, p, _1)(i);   //(copy p)->f(i); 
    return 0; 
}
```

## share_ptr 与 weak_ptr
### shared_ptr 可实现copy_on_write（写时复制）
参考：	  http://blog.csdn.net/kimg_bo/article/details/50644608
源码参考： muduo项目中examples\asio\chat\server_threaded_efficient.cc
原理：
- read在都之前引用计数+1，读完引用技术-1
- write在写之前检查引用计数是否为1，如果为1直接修改，如果大于1,reset复制一个副本，在副本上修改，然后用副本替换以前的数据。
```
    MutexLockGuard lock(mutex_);
    if (!connections_.unique())
    {
      connections_.reset(new ConnectionList(*connections_));  //创建副本，在副本上修改数据
    }
    assert(connections_.unique());
```

### shared_ptr在循环引用时可能会引起内存泄漏
```
#include <stdio.h>
#include <string.h>
#include <boost/shared_ptr.hpp>
#include <boost/weak_ptr.hpp>

class parent;
class child;

typedef boost::shared_ptr<parent> parent_ptr;
typedef boost::shared_ptr<child> child_ptr;

class parent{
public:
    ~parent(){
        printf("parent decon\n");    
    }
    child_ptr child;
};

class child{
public:
    ~child(){
        printf("child decon\n");    
    }
    parent_ptr parent;
};


int main(){
    parent_ptr p(new parent());
    child_ptr c(new child());
    p->child = c;
    c->parent = p;
    printf("use %d\n", p.use_count());
    parent_ptr a = p;
    printf("use %d\n", p.use_count());
    p.reset();
    printf("use %d\n", p.use_count());
}
```
### weak_ptr解决shared_ptr存在问题
weak_ptr对象引用资源时不会增加引用计数，但是它通过lock()方法判断它所管理的资源是否被释放。
PS：不能通过weak_ptr直接访问资源
```
#include <stdio.h>
#include <string.h>
#include <boost/weak_ptr.hpp>

class parent;
class child;

typedef boost::weak_ptr<parent> parent_ptr;
typedef boost::weak_ptr<child> child_ptr;

class parent{
public:
    ~parent(){
        printf("parent decon\n");    
    }
    child_ptr child;
};

class child{
public:
    ~child(){
        printf("child decon\n");    
    }
    parent_ptr parent;
};


int main(){
    boost::shared_ptr<parent> ps(new parent());
    boost::shared_ptr<child> cs(new child());
    parent_ptr p(ps);
    child_ptr c(cs);
    ps->child = c;
    cs->parent = p;
    printf("use %d\n", p.use_count());
    parent_ptr a = p;
    printf("use %d\n", p.use_count());
    p.reset();
    printf("use %d\n", p.use_count());
    return 0;    
}
```