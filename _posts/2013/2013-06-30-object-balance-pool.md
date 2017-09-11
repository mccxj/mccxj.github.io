---
layout: post
comments: true
title: "带平衡的对象池设计"
categories: ["java", "pool"]
---

最近有个需求，是关于一个Socket连接池的功能改造，要求实现下面的需求:

1. 可配置多个服务器地址
2. 服务器地址可以配置权重(比如3:2:1)
3. 连接池可以设置最小连接和最大连接
4. 某服务器从崩溃中恢复后，连接池中的连接数可以自动恢复到服务器之间的权重比。
5. 连接可设置最大空闲释放时间

一开始有同事自定义了一个链表来做取数和归还的操作，在线程安全方面操作感觉还是有点麻烦。
后来我采用PriorityBlockingQueue来实现，其实就是内部变成了堆的结构。见下面的图:

![数据结构](/assets/images/2013/pool/1.png)

![类图](/assets/images/2013/pool/2.png)

![状态图](/assets/images/2013/pool/3.png)