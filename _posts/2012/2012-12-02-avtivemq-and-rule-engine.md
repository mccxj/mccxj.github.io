---
layout: post
comments: true
title: "avtivemq和规则引擎资料(旧)"
description: "以前收集的一些资料"
categories: ["资料", "activemq", "drools"]
---

10年时接触activemq和drools时收集的一些资料

## activeMQ资料
[activeMQ官方网站][1]  
[JMS消息类型模型][2]  
[实战activeMQ][3]  
[ActiveMQ4.1 +Spring2.0的POJO JMS方案(上)整理版][4]  
[ActiveMQ4.1 +Spring2.0的POJO JMS方案(下)整理版][5]  
[ActiveMQ5.0实战一: 安装配置ActiveMQ5.0][6]  
[ActiveMQ5.0实战二: 基本配置][7]  
[ActiveMQ5.0实战三:使用Spring发送,消费topic和queue消息][8]  
[ActiveMQ测试报告][9],这是一个很棒的报告  
[ActiveMQ 高级特性][10]  
[Articles on ActiveMQ, Messaging and JMS][11] 来自官网的文章集锦  

#### 客户端
关于客户端都可以在相应的官网上找到，这里要说的一个客户端是openamq-jms，简单来说，这是针对java JMS的客户端，
可用于OpenAMQ和其他AMQP服务器。还有一个是StompConnect，当然这是走stomp协议的，特色就是把JMS弄成一个Stomp Broker，
具体是怎样的，还没实践过。不过，要跨语言消息交互，更好的选择是使用已经提供的本地Stomp支持。另外，关于stomp的erlang客户端可以选择gigix推荐的stomperl

#### 书籍
其实官方文档已经非常丰富，书籍的意义并没有那么大~~另一方面，关于ActiveMQ的书籍暂时还没有上市，只有本期书(activeMQ in action)，
现在可以在网上找到这本书的手稿版~确实找不到的话，可以找我要，呵

#### activemq和rabbitmq
activemq和rabbitmq都是非常有名的开源消息队列。activemq是java写的，而rabbitmq是erlang写的。
activemq支持的是stomp和openwire，xmpp等协议，而rabbitmq支持的是amqp(Advanced Message Queuing Protocol)。
stomp是一个简单的容易实现的基于文本的协议，openwire是一种快速的二进制协议，activemq计划在amqp协议1.0版本完成的时候提供amqp协议的支持。
通过这些协议，activemq可以支持多种客户端，如C, C++, C#, Ruby, Python, Perl, PHP等等。而amqp同时定义了消息中间件的语意层面和协议层面，
并且是语言中立的，它可以让MQ成为一个可编程的消息中间件，可以想象这样的场景：
你使用java写的消息通过Erlang编写的中间件来和一个C编写的遗留系统进行消息交互。所以amqp前途是光明的～～

#### 2009-01-15附加
activemq的failover功能配置很简单~例如简单修改brokerURL为"failover:(tcp://caixiaojian.hnjk.com:61616?wireFormat.maxInactivityDuration=0)"，正常情况是会出现下面的信息：
{% highlight properties %}
ActiveMQ Task 15/01 16:52:43,672 DEBUG [failover.FailoverTransport]-802 Waiting 5120 ms before attempting connection.
ActiveMQ Task 15/01 16:52:45,569 DEBUG [failover.FailoverTransport]-593 urlList connectionList:[tcp://caixiaojian.hnjk.com:61616?wireFormat.maxInactivityDuration=0]
ActiveMQ Task 15/01 16:52:45,569 DEBUG [failover.FailoverTransport]-712 Attempting connect to: tcp://caixiaojian.hnjk.com:61616?wireFormat.maxInactivityDuration=0
ActiveMQ Task 15/01 16:52:45,570 DEBUG [failover.FailoverTransport]-753 Connect fail to: tcp://caixiaojian.hnjk.com:61616?wireFormat.maxInactivityDuration=0, reason: java.net.ConnectException: Connection refused
ActiveMQ Task 15/01 16:52:45,570 DEBUG [ tcp.TcpTransport]-458 Stopping transport tcp://null:0
{% endhighlight %}

#### 2009-01-16附加
结合spring和activemq的时候，发现启动挺慢的。因为老是会去网上寻找xsd文件，把xml里边的schemaLocation换成下面这个样子就可以了：
http://activemq.apache.org/schema/core http://activemq.apache.org/schema/core/activemq-core.xsd
ActiveMQ的Virtual Destinations和Mirrored Queues非常有价值，5.3新增的Message Groups也是很有用的特性来的

## 规则引擎资料
[Java规则引擎与其API(JSR-94)][12] 介绍jsr94标准  
[使用 Drools 规则引擎实现业务逻辑][13] 一个简单的例子  
[在你的企业级java应用中使用Drools][14] 提供web层和dao层的简单继承，不过rule用的xml格式～  
[drools和spring的集成][15] 集成spring好像是必备功能似的  
[规则引擎实现探讨][16] 一个小讨论，还看到一段Erlang代码  
[应用规则引擎组织业务规则的注意要点][17] 里边提到一些建议和规范  
看惯中文的，感觉[这个blog][18]还行,但是内容是基于drools3的  
[一个小学题目的解: 采用规则引擎Drools实现][19] 一个喝汽水的智力题,一个比较不错的例子  
[springside里边用的jboss rules][20]  
另外还有drools官方文档，关于最重要的rule language和example部分是在Drools Expert上

#### 其他想法
我对规则引擎理解不深，从drools的例子可以看到一些AI,workflow,专家系统的影子，看似用途很广很强大，可是规则引擎不是这么个用法的把  
按照我的想法，规则引擎应该用在局部复杂的流程控制上(局部复杂的if else...),应该是基于业务走向的判断。  
如上面[yimlin][22]提到的，我觉得规则引擎是来简化工作的，所以要避免任何具有实际意义的业务操作，很容易让人混淆。  
怎么说这些规则也不大可能是客户来维护的了，因为可能涉及事实库的修改，dsl顶多也是对程序员友好的方式。  
**哪些业务规则适合抽象出来由规则引擎处理，规则引擎和业务系统的交互方式是主要问题。** 

引入规则引擎真的会带来很多好处么？对很多系统来说，这种[java应用中嵌入groovy][21]的方案还算满经济的。


 [1]: http://activemq.apache.org/
 [2]: http://andyao.javaeye.com/blog/153173
 [3]: http://www.javaeye.com/topic/275045
 [4]: http://wiki.springside.org.cn/display/springside/ActiveMQ
 [5]: http://wiki.springside.org.cn/display/springside/ActiveMQ-part2
 [6]: http://www.javaeye.com/topic/153171
 [7]: http://www.javaeye.com/topic/154092
 [8]: http://www.javaeye.com/topic/234101
 [9]: http://sdh5724.javaeye.com/blog/318376
 [10]: http://www.javaeye.com/topic/397902
 [11]: http://activemq.apache.org/articles.html
 [12]: http://www.ibm.com/developerworks/cn/java/j-java-rules/
 [13]: http://www.ibm.com/developerworks/cn/java/j-drools/
 [14]: http://javahy.javaeye.com/blog/384538
 [15]: http://blog.csdn.net/nimeimei/archive/2005/12/23/560379.aspx
 [16]: http://www.javaeye.com/topic/25215
 [17]: http://www.javaeye.com/topic/18198
 [18]: http://www.blogjava.net/guangnian0412/
 [19]: http://www.javaeye.com/topic/257039
 [20]: http://wiki.springside.org.cn/display/springside/JBossRules
 [21]: http://ivan.javaeye.com/blog/373237
 [22]: http://yimlin.javaeye.com/
