---
layout: post
comments: true
title: "学习和理解servlet(旧)"
description: "以前学习servlet的笔记"
categories: ["总结", "servlet", "学习"]
---

08年写的笔记，看上去很不是太烂:)

## 简单应用服务器
经常会听到应用服务器的说法，主要作用就是能够根据http请求来分配资源。在java里边，就是跟java.net.\*相关的socket编程。
更详细的内容需要查看相关内容。这里只是不想上当受骗，以为是某某魔术来着。

## 什么是规范
为了方便开发，应用服务器开发商只要实现相关的规范，使用者在使用这些api的时候根本不用考虑对应的服务器是怎样的。
简单来说，就是定义好的接口。而servlet和jsp就是这样一类规范。tomcat6.0.18是支持servlet2.5和jsp2.1的规范的。在源码里边实现了这套规范，相关的包是javax.servlet.\*.

## 学习和理解servlet
servlet是一套比较简单的规范，单从tomcat的实现代码就可以知道。从可以得到的资料上看，
也没有多少复杂的内容。窃以为从书上就可以获取到基本的内容了，有个感性印象的基础读一遍
api doc就基本ok了。再多的东西，具体的应用实现，查阅cookbook之类的guice。
这部分内容相当重要，是java web最重要的基础

接下去的部分，基本上是api doc和tomcat实现的内容的梳理。即使再怎样，我都要经常翻阅资料，这是免不了的。
可以注意到servlet一共有两个包，javax.servlet和javax.servlet.http,明显就是一个是基础接口，一个是http协议的实现，
从里边的类名也可以看出，http包的类前面基本都加上Http前缀的。先对他们进行更详细的分类，
大概就是各个基础组件(context,request,response,servlet,filter,stream,listener)，还有一系列对应的listener,几个http特殊的东西(session,cookie).下面分别来说，就怕有些乱，请见谅：

#### ServletConfig
servlet初始化的时候，web容器传递的一个servlet configuration object。

#### ServletContext
我们知道，每个jvm里边一个web应用（通常是一个war包，具有特定的部署结构）就对应一个context，
这个接口就是定义了context和servlet容器交互的一些方法。context是web应用全局可见的，因此ServletConfig就有个方法getServletContext来获取对应的context。

#### Servlet
所有的Servlet必须实现Servlet接口。一个servlet是一个跑在web服务器的java程序，用来接受请求，并做出响应，
对应的请求和响应就是ServletRequest和ServletResponse了。这个生命周期主要有三部分，对于一个jvm来说，
servlet初始化的时候会调用init方法，容器负责把ServletConfig作为参数传递给这个方法，这个步骤对于一个生命周期来说只会做一次。
初始化之后，任何请求调用都会调用service方法，而ServletRequest和ServletResponse就作为两个参数传递进去了。
在容器关闭的时候，servlet将会给销毁，这个时候就会调用destroy方法，这个步骤跟init一样，也是调用一次的。
为了方便使用，已经GenericServlet和HttpServlet做了部分的实现，特别是HttpServlet，根据Http请求方法，分发给doGet，doPost等方法，
这就是为什么继承HttpServlet做servlet实现的时候，一般只是覆盖doGet或doPost方法就可以的原因，

#### ServletRequest,ServletResponse
在调用Servlet的service的方法时，会涉及ServletRequest和ServletResponse，这两个object是
由容器负责生成并传递进来的。简单的解释，ServletRequest是用来提供客户端的请求信息的，例如请求方法，参数，ip等等，
而ServletResponse就是用来返回给客户端的响应信息，一般来说，我们会通过getWriter(字符数据)或者getOutputStream(二进制)来输出信息。
当然，character encoding和content type也是放在ServletResponse的，你可以进行改变。一般来说，我们处理浏览器请求，几乎都是http协议，
所以几乎都是HttpServletRequest和HttpServletResponse。

#### FilterConfig,Filter，FilterChain
过滤器，在web.xml经常看到元素，它就是实现Filter的一些类而已。。可以注意到Filter接口跟Servlet有些像，
在初始化的时候，容器传递一个FilterConfig对象作为init方法的参数，而在销毁的时候会调用destroy方法。当有请求过来的时候，容器会拦截符合要求的请求（根据请求路径），
并调用doFilter方法，可以注意到除了平时看到的request和response参数，还有个FilterChain参数。这个对象是由容器负责维护，可以看做是这次请求需要经过的filter链。一般来说
{% highlight bash %}
before_filter :dosoming with request and response
chain.doFilter(request,response);//进入其他Filter
after_filter :dosoming with request and response
{% endhighlight %}
上面前后两部分不一定都会存在，大家有机会看看例子就知道了。filter的作用范围很大，一般用来做验证权限，记录日志，响应内容压缩，内容加密，特殊参数过滤，转换编码格式等等

#### Cookie,HttpSession
Http协议是一种无状态协议，所以有必要做点其他东西来保留状态。cookie带有简单信息的东东（key/value对，看起来就像个Map），
用来保存在浏览器的，然后每次请求的时候，又把cookie发送到服务器。可以通过ServletResponse的addCookie方法加入cookie。
需要注意的是，cookie具有一些特殊性，可以设置保留时间，是否加密等等，鉴于安全性考虑，一般只用来对付那些不是很重要的数据。
HttpSession说的是session，他的数据其实是保存在服务器的，服务器根据每个sessionid，保留一份对应的数据，客户端就是根据这个sessionid来作为钥匙的。
这个钥匙也是要像cookie那样传来传去的，这个时候的sessionid一般有两种交互方式，基于cookie（作为cookie的key/value对）
和基于url rewrite（在url后边带上个参数而已），常用的方法：setAttribute,getAttribute（跟ServletRequest的方法差不多）,invalidate等等

#### Listener,Event
经常会在web.xml里边看到元素，对应的类是实现ServletContextListener接口的。其实这类Listener有不少，在servlet2.5规范里边有8个，而且有对应的Event类

**ServletContextListener,ServletContextEvent**
这是用来监听servlet context的，是在初始化和销毁的时候会触发相应的事件，并会有个ServletContextEvent对象
参数，通过这个参数的getServletContext方法就可以获取相应的ServletContext对象了。

**ServletContextAttributeListener，ServletContextAttributeEvent**
这个也是用来监听servlet context的，关注点是它里边的attribute的改变。一个类如果实现了
这个接口，那么在attribut发生变化的时候，就会触发相应的方法。

**ServletRequestListener,ServletRequestEvent,ServletRequestAttributeListener,ServletRequestAttributeEvent**
这些都是类似的，只不过这里是ServletRequest

**HttpSessionListener,HttpSessionEvent,HttpSessionAttributeListener,HttpSessionEvent**
类似，就是这两个listener用的是同一个Event

**HttpSessionBindingListener,HttpSessionBindingEvent**
一个类如果实现了这个接口，那么在他和session绑定或者解绑的时候就会触发相应的方法。

**HttpSessionActivationListener,HttpSessionEvent**
注意这里使用的也是那个Event。此接口处理从一个服务器移动到另一个服务器的会话，貌似是用在分布式环境的。

监听器还是很多类型的，注意区分，貌似需要用到的时候也不是很多，感觉没filter用得多

#### RequestDispatcher,SingleThreadModel
貌似这个dispatcher经常用来做请求转发(forward)或者资源内嵌(include)的工作。可以通过ServletRequest来获得一个转发对象。
使用forward可以确保attribute里边的信息不丢失，实现信息传递。
SingleThreadModel，因为Servlet并非线程安全，所以才有这么个东东来保证一个jvm只有一个线程可以同时访问。具体的看资料，这个东西是不推荐使用的，
理由很简单，性能不好罗。既然非线程安全，那么就要保留servlet的无状态性。。。。

**要点：区分各个对象的生命周期和作用**

#### 其他讨论
* 能否在一个ServletRequest对象中获取ServletConfig对象，为什么?
* 讨论何时使用context,cookie，session，request传递信息。他们的存活期是怎样的？
* SingleThreadModel的弊端和解决办法？
* 上传文件和提交普通表单的处理方式？如何避免重复sumbit form？
* 重定向和转发的区别，在servlet的里边的使用方法？


