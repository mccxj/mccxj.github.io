---
layout: post
comments: true
title: "源码阅读应该得到什么?"
description: "源码阅读应该得到什么?"
categories: ["opensource", "reading"]
---

我们平时会接触到不少开源项目, 很多人都期望能够通过阅读源码理解更多东西，把从中学到的东西用于平时的框架使用，设计和编码实现过程。

不过，浩瀚的代码量，应该关注什么? 作为一个屌丝程序员，分享一下屌丝经验:

1. 首先，你应该有一定的使用经验。从使用中，去找相关特性是如何实现的。
2. 最好阅读一下官方文档，了解使用的场景，高层设计、特性介绍。做到心中有数。
3. 关于设计模式。提前学习一下还是有用的，学习源码的时候就不用揪着设计模式不放，会崩溃的。
4. 同上，很多源码书会画很庞大的类图，时序图等。这些看上去很高大上，然并卯。
5. 还是要带的问题去跟踪，一次跟踪一小部分，就是一次要有一个关注点，集中分析。别走马观花。
6. 建议使用maven等高大上工具关联源码，应该多调试调试，但是单步调试帮助不大。
7. 同上，应该提高代码"猜想"能力,大步调试过去，验证自己的猜测。
8. linus说: 烂程序员关心的是代码。好程序员关心的是数据结构和它们之间的关系。
9. 多补充理论知识。虽然很花时间。
10. 凑数的，找个感兴趣的开始吧。

上面废话很多，下面来个事例。

commons httpclinet 3.x 这是一个很超好用的库，实际上也是比较容易理解的。

我阅读的时候，列了一下自己的关注点或者问题列表。

* 如何打开HttpClient库的详细日志
* httpclient默认参数有哪些?如User Agent、Charset怎么进行定制?
* httpclient的连接是怎样管理的? 内部是连接池还是什么? HttpClient能用于多线程环境么?如何应用? 
* 关于MultiThreadedHttpConnectionManager的数据结构是怎样的?
* uri和content中的字符编码是怎么处理的? 例如像处理带中文的路径。
* HttpClient#executeMethod实现细节，为什么hostconfig要克隆一份?为什么不能直接设置?
```java
        if (hostconfig == defaulthostconfig || uri.isAbsoluteURI()) {
            // make a deep copy of the host defaults
            hostconfig = (HostConfiguration) hostconfig.clone();
            if (uri.isAbsoluteURI()) {
                hostconfig.setHost(uri);
            }
        }
```
* HttpClient、HttpMethod(GetMethod、PostMethod)、HostConfiguration、HttpConnectionManager、HttpClientParams、HttpConnection、HttpState、AuthChallengeProcessor主要负责什么?
* 命令行参数、HttpClient、HttpMethod、HostConfiguration都可以设置params，有区别? 参数在框架中是怎样存储的?
```java
        HttpClient httpClient = new HttpClient();
        httpClient.getParams().setConnectionManagerTimeout(30000L);
        httpClient.getHostConfiguration().getParams().setLongParameter(
                HttpClientParams.CONNECTION_MANAGER_TIMEOUT, 30000L);
        GetMethod post = new GetMethod("http://10.132.10.59:4567");
        post.getParams().setHttpElementCharset("UTF-8");
```
* HttpClient会自动处理请求跳转? 会不会出现死循环? 怎样的响应才会自动跳转? 如何关闭这个特性
* HttpClient的自动重试功能是怎样? 怎么进行定制?
* HttpClient是使用java自带的HttpUrlConnection实现的么? 报文是怎么组装和解析的?
* 连接超时、读取超时、读写缓冲区、还有禁用Nagle等常见属性如何设置?
* 多次调用之间的cookie怎么管理的?
* chunked模式是怎么实现的? 多大刷一次? 

大家可以参考这个，在阅读的时候，列出问题列表，然后从源码中找找答案。

**细节是魔鬼:)**

**========================华丽的分割线========================**

关于第8点，以MultiThreadedHttpConnectionManager为例，如何分析数据结构? (昨晚分析的，时间有限，不保证正确性)

从本质来说，整个链接管理的结构是:

```
维护一个connectionPool, 内部维护一个mapHosts，它的结构是Map<HostConfiguration, HostConnectionPool>
HostConnectionPool的内部结构是两个链表，freeConnections维护空闲连接，waitingThreads维护正在等待连接的线程

还有一个静态的ReferenceQueue和ReferenceQueueThread，用来实现防止某些连接丢失的情况。至于是怎么做到丢失的连接能够回到ReferenceQueue，就是通过对HttpConnection进行加强得到的。

实际上通过管理器得到的HttpConnection是经过层层代理的。层次是这样的: HttpConnectionAdapter - HttpConnectionWithReference - HttpConnection
HttpConnectionWithReference就是通过一个WeakReference关联到ReferenceQueue，这样根据弱引用特性，一定可被回收就会进入ReferenceQueue从而被线程扫描到。

为了实现闲时连接关闭的功能，使用了IdleConnectionHandler，它的内部结构是有个Map记录空闲连接(在releaseConnection的时候放入)和当时的时间。
```

代码就是围绕这些核心数据结构进行操作的。通过上面的基本结构，就可以实现以下特性:

1. 总连接数和每主机总连接数限制。连接数默认50，每host默认2连接(经常容易忽略，通常需要定制)
2. 能够防止连接丢失。如上面所说，拿出去的HttpConnection是加强过的，会关联到一个静态的ReferenceQueue,并使用一个线程ReferenceQueueThread不停监听并回收。
3. 实现了获取连接的超时限制。就是通过waitingThreads实现的。当找不到空闲连接的时候就添加进去，然后wait。如果有releaseConnection的时候，就会interupt等待的第一个线程。我个人认为线程有可能被interupt之后仍然占用不到连接，从而排到最后去，真是悲剧。
4. 能够支持连接闲时关闭。这个默认是需要主动触发，通过IdleConnectionHandler比较时间来实现的。

最后，这个类是支持多线程的，主要是使用了同步机制，锁定ConnectionPool对象实现的。  
另外，HttpConnection对象实际上并不是一个正式的连接，要open之后才会真的建立连接。  
实际上也是一个lazy代理对象，可以避免整个连接管理操作不被阻塞，只有到实际操作时open的时候才连接。

**========================华丽的分割线========================**
 
关于第9点，以commons-httpclient为例，这当中涉及到哪些理论基础呢? 简单列举一下:

1. java弱引用、克隆特性
2. 线程协调、中断机制
3. 网络编码、http协议
4. tcp相关参数的含义
5. 常见对象池化技术