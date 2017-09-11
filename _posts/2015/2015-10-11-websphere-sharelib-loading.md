---
layout: post
comments: true
title: "Websphere共享库加载顺序问题"
description: "Websphere共享库加载顺序问题"
categories: ["was", "共享库", "类加载"]
---

## 问题描述

昨天收到有个童鞋发来的一个问题咨询，如下图所示。

![问题截图](/assets/images/2015/was-sharelib1.png)

提到几个疑问:

1. 配置如图所示,然后共享库和项目自身lib下都有一个“xxxx-common.jar”，如果项目用到jar包里面的一个类，将会是共享库的还是自身lib的呢？发现是用的lib里边的。
2. 现网情况也是xxlib和lib下都有那jar包，但是根据日志来看，是用共享库的。

## 我的疑惑

现网的配置如何，暂时没查明。不过就开发环境的配置来看，我一直认为Parent First应该会走共享库的，目前的现象和我掌握的知识不匹配。

于是，我上网搜索了一下Websphere关于共享库的资料，主要的链接如下：

* http://www.ibm.com/developerworks/cn/websphere/library/techarticles/haoaili/0512/
* http://www-01.ibm.com/support/knowledgecenter/SSAW57_8.5.5/com.ibm.websphere.nd.doc/ae/tcws_sharedlib_nativelib.html?lang=zh
* https://10.132.10.69:9043/ibm/help/index.jsp?topic=/com.ibm.ws.console.environment/ucws_rsharedlib_inst.html

通过这些链接资料的描述(不得不说，这些中文翻译很隐晦)，但的确是可以解释目前的情况的。

## 关于Websphere共享库的理解

首先，Websphere的共享库和tomcat的共享库差别很大，而我却一直以为是差不多的。  
tomcat的共享库是一个独立的类加载器，并且在多个Web应用中共享。好处是明显的，共享加载的类，优化内存使用。

其次，Webshpere的共享库非常灵(fu)活(za)，有多种配置组合可以影响结果。具体如下:

#### 共享库是可以选择和服务器关联或者和应用关联的

* 和服务器关联，参考http://www-01.ibm.com/support/knowledgecenter/SSAW57_8.5.5/com.ibm.websphere.nd.doc/ae/tcws_sharedlib_server.html?lang=zh
* 和应用关联，参考http://www-01.ibm.com/support/knowledgecenter/SSAW57_8.5.5/com.ibm.websphere.nd.doc/ae/tcws_sharedlib_app.html?lang=zh

#### 共享库是可以选择是否使用隔离的类装入器(就是独立的类加载器)

设置参考下图所示:

![请对此共享库使用隔离的类装入器](/assets/images/2015/was-sharelib2.png)

#### 和共享库相关的类加载策略如下:

* 如果选择和服务器关联，那么将忽略"请对此共享库使用隔离的类装入器"的选项，此时共享库路径将会添加到应用程序服务器(application server)类装入器加载路径上。
* 如果选择和应用关联，并且没有设置"请对此共享库使用隔离的类装入器",那么共享库路径将会添加到应用的类加载器加载路径上。此时共享库只有优化管理类库的作用，并不能减少重复加载类造成的内存占用。
* 如果选择和应用关联，并且设置"请对此共享库使用隔离的类装入器",那么共享库将作为独立的类加载器，并且各个应用之间共享这个共享库。此时共享库和tomcat的共享库类似，可以减少重复加载类造成的内存占用。

对于第三种情况，它的类加载顺序如下：

如果应用的类载入顺序选择“父类装入器装入的类最先”,即Parent First，那么顺序如下:

* 检查相关联的库类装入器是否可以装入类。(共享库)
* 检查它的父代类装入器是否可以装入类。(应用服务器及更高)
* 检查应用程序或 WAR 模块类装入器是否可以装入类。(应用)

如果应用的类载入顺序选择“本地类装入器装入的类最先”,即Parent Last，那么顺序如下:

* 检查应用程序或 WAR 模块类装入器是否可以装入类。(应用)
* 检查相关联的库类装入器是否可以装入类。(共享库)
* 检查它的父代类装入器是否可以装入类。(应用服务器及更高)

## 现象解释

* 开发环境中，共享库和应用关联，并且没有设置"请对此共享库使用隔离的类装入器"，所以共享库路径将会添加到应用的类加载器加载路径上，相当于在一个类加载路径上存在同样的类，所以使用到lib中的是可能的。
* 生产环境中配置尚未查明，如果共享库和应用关联，并且设置"请对此共享库使用隔离的类装入器"，按同样的载入顺序设置，即Parent First，那么是会加载到共享库的。
* 如果同样是没有设置"请对此共享库使用隔离的类装入器"，那么情况如开发环境情况，使用到共享库中的也是可能的。
* 对于同一个类加载路径上存在同样的类，具体会加载哪个是不确定的，所以上述情况都是合理的。所以应该把应用中重复的jar包移除。



