---
layout: post
comments: true
title: "Web开发利器-Fiddler简介"
categories: ["fiddler", "代理", "http", "工具"]
---

## 1 什么是Fiddler?

![什么是Fiddler](/assets/images/2013/fiddler/image1.png)

Fiddler是一个http调试代理，以代理服务器的方式，监听系统的Http网络数据流动，
Fiddler可以也可以让你检查所有的http通讯，设置断点，以及Fiddle所有的“进出”的数据。

Fiddler还包含一个简单却功能强大的基于JScript .NET 事件脚本子系统，它可以支持众多的http调试任务。
你是不是曾经疑惑过你的web程序和IE是如何交互的？你是不是遇到过一些奇怪的而你又无法解决的性能瓶颈？
你是不是对那些发送给服务器端的cookie和那些你下载下来的被标记为可缓存的内容感到好奇？
无论你是从事什么开发，哪种语言，只要你想更深入的了解HTTP，这个工具就值得你去了解，它对前端开发工作是很有价值的。

Fiddler官方网站及下载可以在[http://www.fiddler2.com/fiddler2/]找到，安装过程很简单，这里就不介绍了。
同样，Fiddler支持插件扩展，常见插件可以在[http://fiddler2.com/add-on]找到。下面，简单介绍Fiddler的功能和常见应用场景。

## 2 Fiddler界面及功能介绍

![Fiddler界面及功能介绍](/assets/images/2013/fiddler/image2.png)

Fiddler整个界面布局如上所示，下面再简单介绍一些特殊的概念：

代理模式，支持缓存模式和流模式:

* 缓冲模式(Buffering Mode Fiddler直到HTTP响应完成时才将数据返回给应用程序。
可以控制响应，修改响应数据。但是时序图有时候会出现异常。
* 流模式(Streaming Mode): Fiddler 会即时将HTTP响应的数据返回给应用程序。
更接近真实浏览器的性能。时序图更准确。但是不能控制响应。

断点类型，支持请求断点和响应断点:

* 请求断点: 在向目标服务器发送请求前截获。
* 响应断点: 在将结果返回到应用程序前截获。此时如果使用的是流模式，则此断点类型将失效。

## 3 常见案例介绍

### 快速定位问题页面

有些复杂页面，具有多层嵌套页面或者大量Ajax交互，甚至于模态窗口等，都可能让你无法定位请求路径、问题所在页面等。
借助于Fiddler的请求会话查询功能，可以让你一揽全局，无需逐个页面进行查找。

第一步:使用Ctrl+X清空会话列表，再刷新页面

![清空会话列表](/assets/images/2013/fiddler/image3.png)

第二步:使用Ctrl+F弹出搜索框,输入关键字进行查询

![弹出搜索框](/assets/images/2013/fiddler/image4.png)

第三步:参看具体会话缩小定位范围

![缩小定位范围](/assets/images/2013/fiddler/image5.png)

第四步: 定位到具体请求，进行下一步处理

接下来可以考虑使用AutoResponse进行快速修复验证，或者根据请求路径反查后台逻辑。

### 使用AutoResponse快速修复验证

在日常开发工作中，有时侯会发现测试环境中某个html/css/javascript文件有问题。
我们利用Fiddler可以修改HTTP数据的特性，非常方便的定位问题并进行验证。

第一步:使用Fiddler查看页面的数据流列表，找到js文件保存到本地

![保存到本地](/assets/images/2013/fiddler/image6.png)

第二步:创建重定向规则,使用本地文件

![使用本地文件](/assets/images/2013/fiddler/image7.png)

第三步:刷新页面,如果看到灰色背景的请求会话，就表示生效了

![刷新页面](/assets/images/2013/fiddler/image8.png)

第四步:修改本地文件，进行测试

修改本地文件之后，重新刷新页面,就可以看到修改后的效果了。
这种调试方式不需要发布到线上再验证，避免了修改不成功、对用户造成影响的风险，
而且不需要搭建复杂的开发服务器等开发环境，非常适合快速web调试。

### 使用Composer模拟报文发送

有时候在做一些长流程页面、Web服务接口调试时，为了避免修改后重新发起调用的时间过长，可以通过Composer构造请求报文进行快速测试。
等接口调通之后再进行集成测试，可以有效的提高开发和测试的效率。

第一步:拖拽相应的请求会话到Composer页面，会自动生成请求报文

![自动生成请求报文](/assets/images/2013/fiddler/image9.png)

第二步:修改请求报文，然后按Execute进行发送

例如，我去掉Cookie中的JESSIONID，重新发送可以看到多了两次请求会话信息。

![修改请求报文](/assets/images/2013/fiddler/image10.png)

第三步:检查响应报文，验证结果

例如: 我去掉Cookie中的JESSIONID，应该会被跳转到登录页面

![验证结果](/assets/images/2013/fiddler/image11.png)

除了用来做功能快速调试之外，还可以用来做一些安全方面的测试工作，
例如构造一些xss注入、SQL注入报文，看看应用能否妥善处理。

### 观察页面性能

大多数情况下，一个页面会有好几个请求，除了html页面，还有js/css/图片等。
但是IE等浏览器不能很方便的观察到页面加载的情况，例如每个请求消耗的时间等。
如果使用Fiddler的话，可以使用Statistic视图和Timeline视图，观察页面加载情况，便于定位页面性能瓶颈。

第一步:选择一个或多个请求，根据Statistic视图查看统计时间

![查看统计时间](/assets/images/2013/fiddler/image12.png)

第二步: 选择一个或多个请求，根据Timeline视图查看加载流程

![查看加载流程](/assets/images/2013/fiddler/image13.png)

### 模拟特殊场景

例如在广东XX项目中，各地市使用的域名是不一样的，有些逻辑根据域名来进行特殊处理。为了模拟这种场景，可以考虑修改本地hosts文件。
不过修改hosts需要重启浏览器，比较麻烦。使用Fiddler的hosts设置功能，就能很方便的模拟。

第一步:选择Tools->HOSTS功能,设置相应的域名映射

![设置相应的域名映射](/assets/images/2013/fiddler/image14.png)

第二步:直接使用域名访问,验证功能

![验证功能](/assets/images/2013/fiddler/image15.png)

Fiddler提供大量的规则允许你模拟各类场景，你甚至可以自定义规则，值得大家深入探讨，多实践多思考。例如:

* 通过GZIP压缩，测试性能
* 模拟Agent测试，查看服务端是否对不同客户端定制响应
* 模拟慢速网络，测试页面的容错性
* 禁用缓存，方便调试一些静态文件或测试服务端响应情况
* 根据一些场景自定义规则

![模拟各类场景](/assets/images/2013/fiddler/image16.png)

## 4 总结

本文简单介绍了Fiddler的常见应用场景。目前，Fiddler在我们项目的团队中广泛使用。



