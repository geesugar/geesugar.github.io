---
title: MQ
date: 2016-07-05 11:46:20
tags:
---

## ACID CAP BASE理论 两次提交
CAP，一致性，可用性，分区容错性。 对分布式系统分区容错性是最基本需求，所以必然放弃一致性。
NoSQL支持AP，传统数据库支持CA
BASE： basically available（基本可用），soft-state（软状态/柔性事物），Eventual Consistency（最终一致）
BASE模型是传统ACID的反面，BASE强调牺牲高一致性，从而获得可用性，数据允许一段时间内的不一致，只保证最终一致。

## 分布式事物 
### 两次提交
```
案例： 老师短信小明，小强爬山。 如果小明、小强回到都没问题，大家如约而至。如果小明回答明天没空，老师会立即通知小明小强爬上取消。
存在问题：如果小强没看手机，老师会一直等待回复，小明也一直在家等老师消息，那小明明天是否去爬山？
```

### 三次提交
![](http://www.geesugar.com/BlogImg/DTP.png)
XA协议：是指TM（事物管理器）和RM（资源管理器）之间的接口。目前主流的关系型数据库产品都实现了XA接口。
![](http://www.geesugar.com/BlogImg/DTP1.png)

![](http://www.geesugar.com/BlogImg/DTP2.png)

总结： linux上的mysql是不支持分布式事物的。 这种方式实现难度不算太高，比较适合传统的单体应用，在一个方法中存在跨库操作的情况。 但是分布式事物对性能影响比较大，不适合高并发和高性能场景。



## 简单脚本
### 生产者
```
#!/usr/bin/env python  
import pika  
  
connection = pika.BlockingConnection(pika.ConnectionParameters(  
        host='localhost'))  
channel = connection.channel()  
  
channel.queue_declare(queue='hello')  
  
channel.basic_publish(exchange='',  
                      routing_key='hello',  
                      body='Hello World!')  
print " [x] Sent 'Hello World!'"  
connection.close()  
```
### 消费者
```
#!/usr/bin/env python  
import pika  
  
connection = pika.BlockingConnection(pika.ConnectionParameters(  
        host='localhost'))  
channel = connection.channel()  
  
channel.queue_declare(queue='hello')  
  
print ' [*] Waiting for messages. To exit press CTRL+C'  
  
def callback(ch, method, properties, body):  
    print " [x] Received %r" % (body,)  
  
channel.basic_consume(callback,  
                      queue='hello',  
                      no_ack=True)  
  
channel.start_consuming()  
```

## 持久化脚本
### 生产者
```
#!/usr/bin/env python  
import pika  
import sys  
  
connection = pika.BlockingConnection(pika.ConnectionParameters(  
        host='localhost'))  
channel = connection.channel()  
  
channel.queue_declare(queue='task_queue', durable=True)  
  
message = ' '.join(sys.argv[1:]) or "Hello World!"  
channel.basic_publish(exchange='',  
                      routing_key='task_queue',  
                      body=message,  
                      properties=pika.BasicProperties(  
                         delivery_mode = 2, # make message persistent  
                      ))  
print " [x] Sent %r" % (message,)  
connection.close()  
```


### 消费者
```
#!/usr/bin/env python  
import pika  
import time  
  
connection = pika.BlockingConnection(pika.ConnectionParameters(  
        host='localhost'))  
channel = connection.channel()  
  
channel.queue_declare(queue='task_queue', durable=True)  
print ' [*] Waiting for messages. To exit press CTRL+C'  
  
def callback(ch, method, properties, body):  
    print " [x] Received %r" % (body,)  
    time.sleep( body.count('.') )  
    print " [x] Done"  
    ch.basic_ack(delivery_tag = method.delivery_tag)  
  
channel.basic_qos(prefetch_count=1)  
channel.basic_consume(callback,  
                      queue='task_queue')  
  
channel.start_consuming()  
```

