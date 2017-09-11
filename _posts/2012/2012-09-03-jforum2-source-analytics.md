---
layout: post
comments: true
title: "小毛的jforum2源码分析(旧)"
description: "很久的一个源码分析文章"
categories: ["jforum2", "总结", "源码"]
---

这个文章大约是08年的时候写的了，jforum是java中还算比较有名的开源论坛系统，我上一家公司拿它来做二次开发。jforum2是内部自己实现的mvc，到了jforum3的时候全部重写了，都使用开源框架来做，现在不知发展到什么情况了。所以，这里特指jforum2.

## 正文
怎么才算好的源码分析呢？当然我这个肯定不算。我想大概分为几个层面吧，写写注释那算最基本的了，
写写要点思路和难点，算是还不错拉，再难的就是跳出源码举一反三，形成自己的一套思路吧。好好努力吧。

这次针对的是jforum2.1.8,大概jforum团队已经没心情理这个版本了，都冲着jforum3去了。选择这个版本，
主要是因为jforum在java论坛类应用中算是佼佼者了，很多人都拿这个来做二次开发，而jforum3使用的是另外一套架构了，
而且还没完全release，所以斟酌一下，还是选择这个经典的版本。

关于jforum的介绍网上已经很多了，这里也简单抄录一段：JForum 是一个功能强大 ，易于管理的论坛。
它的设计完全遵从MVC设计模式，能够在任何Servlet容器与EJB服务器上运行。而且可以轻松的定制与扩展JForum论坛。 
上面这段简述还是中肯的。另外，jforum是模仿phpbb写的，使用的是classic-blue风格，但不能自己选择风格，要的话只能自己修改了。
再说几句，说jforum比较优秀是因为java开源的论坛系列精品少，而且jforum的bug也真的不少，不信试试就知道了。
不过作为一个成型的组件，功能强大并且适合二次开发，还是应该列入考虑范围的。

不管怎样，jforum是个不错的学习范本，至少让你觉得写个山寨框架不是什么难事，
而事实也的确是这样的。重要的一点是，不要轻易拿出来害人就是了：)这里先列举出可能一些分析点：

●	web.xml  
●	初始化流程  
●	处理请求流程(mvc)  
●	文件监控  
●	缓存实现  
●	数据库访问实现  
●	权限控制  

首先了解一个web应用，首要的就是知道处理流程。首先来看看入口web.xml，里边的内容还是挺清晰的，
可以看到里边有个监听器ForumSessionListener，\*.page的过滤器ClickstreamFilter，还有2个\*.page的处理器，
其中InstallServlet是安装相关的，JForum则是前端处理器。

基本上**整个流程**就是

    client request -> ForumSessionListener -> ClickstreamFilter -> JForum -> server response.

ForumSessionListener实现了HttpSessionListener接口，但是只是对session destory做了处理，
在这个过程中，保存session的历史记录到DB，并清除用户信息和相关的security信息。
ClickstreamFilter实现了Filter接口，主要的任务就交给BotChecker了，是用来检测client是不是一个robot来的。
主要的工作还是在JForum上面，不过先来看看jforum是怎么检测robot的？

BotChecker只有一个静态工具方法isBot，首先是检测是否请求robot.txt(这是标准的robot协议文件),
接下去判断User－Agent头部，最后是判断remotehost。而已知的robot都是写在文件clickstream-jforum.xml
里边的(包括agent和host)，并通过ConfigLoader加载进来的(SAX方式)。

可以看到JForum和InstallServlet都继承了JForumBaseServlet这个HttpServlet，而JForumBaseServlet包括2个重要的方法init和startApplication。众所周知，init是servlet初始化时调用的方法，JForumBaseServlet里边的init方法的流程是：

    调用父类的init(正常情况这是必须调用的)  -> 配置log4j -> startSystemglobals(加载全局参数配置SystemGlobals.properties -> 加载数据库配置database.driver.config(如mysql就是WEB-INF/config/database/mysql/mysql.properties)  -> 加载自定义配置(默认的是jforum-custom.conf)) -> 配置缓存引擎 -> 配置freemarker模板引擎 -> 加载模块配置modulesMapping.properties -> 加载url映射配置urlPattern.properties -> 加载I18n配置(languages/*) -> 加载页面映射配置(templatesMapping.properties) -> 加载BBcode配置bb_config.xml -> 结束

jforum实现了自己的mvc，整个mvc的脉络就是

    client request -> 解析url(urlPattern.properties),获取module/action/param -> 通过module获取相应的module class，并通过action识别并调用相应的方法(modulesMapping.properties) -> 使用dao完成业务逻辑 -> 调用template进行渲染(templatesMapping.properties)

其实整个mvc和struts没什么两样的，具体的流程以后再提。

JForumBaseServlet里边的startApplication方法的流程是：

    加载通用sql文件sql.queries.driver(就是/database/generic/generic_queries.sql) -> 加载特定sql文件(如mysql就是/database/mysql/mysql.sql) -> 加载Quartz定时任务配置 -> 加载登录验证器(验证方式) -> 加载Dao实现方式 -> 加载文件修改监听器 -> 加载查询索引管理器 -> 加载定时统计任务

jforum实现了自己的orm，当然不是hibernate那种，是类似ibatis的那种sql mapping，并提供了多套的sql文件来实现数据库无关的特性，
整个流程也是比较清晰的:

    加载数据库配置 -> 加载sql mapping file -> 设置DAO实现 -> 通过named sql找到对应的sql(在*.sql里边对应着) -> 运行出数据

继续重点。JForum的init流程如下：

    JForumBaseServlet.init -> JForumBaseServlet.startApplication -> 启动数据库 -> 预加载一些数据到缓存中(ForumRepository[Categories,Forums,同时在线最大人数，最后登录用户，注册用户数等等],用户等级,表情数据，屏蔽列表) -> 结束

上面简单提到了**Jforum处理请求的过程**，现在在来看看这个过程，就是service方法，这次采用代码概要的方式展示：
{% highlight java %}
 // 初始化JForumExecutionContext
JForumExecutionContext ex = JForumExecutionContext.get();
// 包装request和response
request = new WebRequestContext(req);
response = new WebResponseContext(res);
// 检查数据库状态
this.checkDatabaseStatus();
// 创建JForumContext并设置到JForumExecutionContext中去
.......
JForumExecutionContext.set(ex);
// 刷新session    	   
utils.refreshSession();
// 加载用户权限    	  
SecurityRepository.load(SessionFacade.getUserSession().getUserId());
// 预加载模板需要的上下文
utils.prepareTemplateContext(context, forumContext);
// 从request中解析module name
String module = request.getModule();
// module name  -> module class
String moduleClass = module != null ? ModulesRepository.getModuleClass(module) : null;
// 判断是否在ban list里边
......
boolean shouldBan = this.shouldBan(request.getRemoteAddr());
// 主角出场
out = this.processCommand(out, request, response, encoding, context, moduleClass);
// 扫尾工作,例如db的rollback
this.handleFinally(out, forumContext, response);
{% endhighlight %}

processCommand会调用Command的process方法：
{% highlight java %}
// 获取一个module实例(继承了Command)
Command c = this.retrieveCommand(moduleClass);
// 进入process
Template template = c.process(request, response, context);
// 这里开始是process方法
//获取action
String action = this.request.getAction();
//如果不是ignore的，就调用这个action
if (!this.ignoreAction) {this.getClass().getMethod(action, NO_ARGS_CLASS).invoke(this, NO_ARGS_OBJECT);}
//如果是转发的，就把TemplateName清空
if (JForumExecutionContext.getRedirectTo() != null) {this.setTemplateName(TemplateKeys.EMPTY);}
//不是转发且attribute里边存在template，则设置为templateName
else if (request.getAttribute("template") != null) {this.setTemplateName((String)request.getAttribute("template"));}
//是否coustomContent？例如下载，验证码子类的不需要页面的操作
if (JForumExecutionContext.isCustomContent()) {return null;}
//返回一个template
return JForumExecutionContext.templateConfig().getTemplate(
                new StringBuffer(SystemGlobals.getValue(ConfigKeys.TEMPLATE_DIR)).
                append('/').append(this.templateName).toString());
        }
// 从process出来，回到processCommand
// 设置content type
response.setContentType(contentType);
//生成页面并flush
if (!JForumExecutionContext.isCustomContent()) {
				out = new BufferedWriter(new OutputStreamWriter(response.getOutputStream(), encoding));
				template.process(JForumExecutionContext.getTemplateContext(), out);
				out.flush();
			}
		}
{% endhighlight %}
这是一般的流程，就像上面提到的customContent，就是要自己处理了，可以参考CaptchaAction.generate().

这样的话，如果我们要增加一些action进行**二次开发**的话，大体的流程就是，增加一个继承了Command的类，
例如叫ExampleAction,定义一个方法，例如叫test()，在urlPattern.properties中定义一个映射,
例如为example.test.1 = forum_id，再在modulesMapping.properties中定义module class的映射，
如example = ExampleAction，最后我们在templatesMapping.properties定义个模板的映射，
如：example.test = example_test.htm。现在假设我们的请求url是/example/test/1,再来看看test里边的一些方法：
{% highlight java %}
this.request.getIntParameter("forum_id"))  //获取参数，得到1
this.context.put("obj", obj); //把结果写入context，这样可以在template中获取到
this.setTemplateName("example.test");//设置template的名字
{% endhighlight %}
这样的简单流程应该还比较好理解吧?

另外，还可以看出，jforum使用了自己的一套映射机制，这是通过urlPattern.properties来定义的
(参考上面JForumBaseServlet的init流程)，这是在JForumBaseServlet的loadConfigStuff方法的第一行实现的，
并加载到UrlPatternCollection中去，如下所示：
{% highlight java %}
Properties p = new Properties();
fis = new FileInputStream(SystemGlobals.getValue(ConfigKeys.CONFIG_DIR) + "/urlPattern.properties");
p.load(fis);

for (Iterator iter = p.entrySet().iterator(); iter.hasNext(); ) {
   Map.Entry entry = (Map.Entry) iter.next();
   UrlPatternCollection.addPattern((String)entry.getKey(), (String)entry.getValue());
}
{% endhighlight %}

可以知道这里的key和value都是String来的
{% highlight java %}
UrlPatternCollection.patternsMap.put(name, new UrlPattern(name, value));
{% endhighlight %}
但在addPattern方法里边其实是生成一个UrlPattern作为value，如何**构造一个UrlPattern**可以看看代码，
举例来说把，对于example.hello.2=a,b,这样会生成一个UrlPattern,里边的内容是name为example.hello.2,value为a,b.
而size和vars是用a,b解析出来的，用来表示一共有多少个参数，参数名组成的数组。
所以UrlPattern存储的就是一个url格式的定义，而放在UrlPatternCollection里边的一系列的url映射格式是在请求的url解析的时候用到的。

现在再分析一下jforum怎么使用这个UrlPatternCollection的?按照我们不严格的思路，应该是service中处理url，
获取.page前面的一部分,如/example/hello/2/1，用/做一下split，获取module name，action name,
把最后的作为参数，用module,action,参数个数组成一个key(example.hello.2),通过UrlPatternCollection找到对应的UrlPattern，
通过里边的格式对应(vars里边的参数名和url的参数值)就可以把参数添加到request的parameters里边去。
实际的情况也差不多就这个样。在说到jforum中的service方法的时候，简单提到过request和response是经过包装的:
{% highlight java %}
request = new WebRequestContext(req);
response = new WebResponseContext(res);
{% endhighlight %}
WebResponseContext只是简单的delegate给HttpServletResponse(这样做的好处是全部方法都限制在ResponseContext中)，
而WebRequestContext是继承了HttpServletRequestWrapper并实现了RequestContext接口。
所以WebRequestContext是一个HttpRequest，但是通过RequestContext接口实现了一些特定的方法就是了，
例如getModule/getAction，而这个解析url的过程是在构建WebRequestContext对象的过程中实现的。
可以看看WebResponseContext的构造方法，这里就不详细说了。注意的是，所有的parameters最后都保存到query(一个私有的map)里边去的。
还有就是上面说到的jforum的特定url映射机制，这是通过WebRequestContext的parseFriendlyURL方法实现的，
原理就和上面提到的那样，也不详说了。

到这里，基本上整个处理流程就差不多了。现在来说说jforum里边的文件修改监听器(JForumBaseServer的startApplication流程)，
如果你在使用jforum的过程中，修改了某些文件如\*.sql，jforum就会重新加载修改后的配置。
我原来以为是用quartz框架来实现的，后来才知道是用jdk的TimerTask类来实现的。
请看ConfigLoader的listenForChanges方法：
{% highlight java %}
FileMonitor.getInstance().addFileChangeListener(new QueriesFileListener(),
				SystemGlobals.getValue(ConfigKeys.SQL_QUERIES_GENERIC), fileChangesDelay);
{% endhighlight %}

这里给各个部分分一下责任，FileMonitor是大管家，负责管理所有的文件监听器；FileChangeListener是一个监听器接口，
只有一个方法，就是fileChanged(String filename)，意思就是对某个filename的修改作出怎样的反应。
使用的方法也很简单，就是实现一个FileChangeListener，并和监控的文件名，检查间隔作为参数传入就可以生效了。
FileMonitor里边的实现原理就是，通过一个map(timerEntries)来保存(文件名/timertask),
每次加入一个监听器的时候，会根据文件名先移出原来的文件监听器(缺点是只能能对一个文件添加一个监听器)，
然后构建一个TimerTask并加入到timerEntries中去。关于TimerTask的具体用法，可以参考api。

作为一个论坛，应用层缓存这样的东西似乎必不可少，jforum也提供了缓存配置(上面也提到一些)。
jforum提供了数种**缓存实现**(JForumBaseServlet的init流程)，分别是DefaultCacheEngine(简单的内存实现),
JBossCacheEngine，EhCacheEngine。，请看ConfigLoader的startCacheEngine方法，
流程大概就是得到cacheEngine的实现配置(SystemGlobals.properties中配置cache.engine.implementation)，
然后产生CacheEngine的实例，调用它的init方法进行初始化，然后找到所有的可缓存类(实现了Cacheable接口，并在SystemGlobals.properties中配置cacheable.objects)，最后把cacheEngine注入进去获得cache的能力。
虽然jforum自己实现了许多这样的注入(除了cacheEngine，还有db，dao等等)，
虽然达到了一定的的目的，可是怎么说还是到处充满了Singleton的实现(参考spring2.5文档3.9. 粘合代码和可怕的singleton)，
为了寻求更好的组织方式(例如使用ioc来管理对象，使用成熟的orm来隔离数据库)和获得更多的用户群(选择更广泛使用的框架帮助)，
大概才会萌发jforum3的想法吧。

顺便提一下jforum的**Dao实现方式**(参考JForumBaseServlet的startApplication流程)，
参考ConfigLoader的loadDaoImplementation方法，原理就是通过配置dao.driver(在特定的数据库配置里边如mysql.properties):

    获取到DataAccessDriver的实现 -> 初始化DataAccessDriver -> 获取到所有的Dao实现。

可以这么理解，实现一个DataAccessDriver就获得一整套Dao的实现方式，对于dao里边的实现方法，给个范例：
{% highlight java %}
//例行公事
PreparedStatement p = null;
ResultSet rs = null;
//获得connect，并执行named sql
p = JForumExecutionContext.getConnection().prepareStatement(SystemGlobals.getSql("GroupModel.selectById"));
p.setInt(1, groupId);
rs = p.executeQuery();
Group g = new Group();
//循环resultset进行处理
if (rs.next()) {g = this.getGroup(rs);}
{% endhighlight %}

整个实现很直白，就是一个jdbc实现方式来的。对于如何获取connection，查看JForumExecutionContext的getConnection(),可以注意到这么一句：
{% highlight java %}
c = DBConnection.getImplementation().getConnection();
{% endhighlight %}
也是比较清晰的，另外可以知道的是，在每次请求的过程中，connection只会获取一次，
并在第一次获取到以后放到ThreadLocal里边去,这样在每个线程中保留一份数据(正确理解TheradLocal )，
在请求请求结束以后才释放connection(service流程中的handleFinally方法)。

JForumExecutionContext，如字面意，就是请求执行的上下文，例如上面提到的数据库连接，
还有ForumContext(放着和request，response相关的信息)，context(freemarker的上下文变量)，
redirectTo(转发地址)，contentType(响应内容格式)，isCustomContent(不使用默认渲染，上面有提到)，
enableRollback(db是否会滚)。

jforum是可以配置权限的，可控制的权限类型放在SecurityConstants里边，
对应的配置界面是根据permissions.xml生成的(参考GroupAction的permissions)。而每个用户的权限(PermissionControl)是通过SecurityRepository来管理的

    最用形成的权限系统是role(权限)－group(用户组,可以多级)－用户这样的结构图。

如何判断权限?
对于一个用户来说，为了获取用户的权限(PermissionControl)，流程是这样的(详细看SecurityRepository的load方法)：

    获取用户信息 -> 获取用户的所有groupid并组成一个用逗号隔开的字符串groupids  -> 根据groupids获取所有的name/role_value -> 组装成RoleValueCollection －> 生成RoleCollection -> 最后生成PermissionControl

判断权限是使用SecurityRepository的canAccess(int userId, String roleName, String value)方法：

    根据userid获取PermissionControl-> 如果value参数为空的话，就判断是否拥有该roleName(通过内部的RoleCollection对象的keys),就是是否含有该权限 -> 如果value参数不为空的话，除了需要含有该权限，还要拥有相应的rolevalue(通过内部的RoleCollection对象的values)。参数中的value指数可以为论坛分类id，论坛id之类，随业务而定。

总体上jforum还算清晰，大部分的业务代码没有细看(那些Command类)，有兴趣可以对照着写，
大体分为三个包(admin是管理，jforum是公共页面，install是安装页面)。

既然说到验证，就顺便要说说jforum的**sso验证机制**
官方文档:

    http://www.jforum.net/doc/SSO
    http://www.jforum.net/doc/ImplementSSO
    http://www.jforum.net/doc/SSOcookies
    http://www.jforum.net/doc/SSOremote

有上面这些文档基本可以自己实现一个，主要就是实现net.jforum.sso接口就是了。

在Jforum的service方法里边有段(service流程中的刷新session)：
{% highlight java %}
ControllerUtils utils = new ControllerUtils()
utils.refreshSession();//重点
{% endhighlight %}
里边提到，在没有usersession的情况下，如果配置的验证类型是sso(authentication.type)：

    调用checkSSO(UserSession userSession)的方法 -> 生成SSO实例(使用sso.implementation来配置) -> 调用authenticateUser(RequestContext request)返回username -> 假如取不到的username，就设为匿名 -> 否则，如果不存在该用户(utils.userExists(username)则注册一个(utils.register(password, email)) -> 假如已经存在,则让用户登录(configureUserSession(userSession, utils.getUser()))

当已经存在usersession的时候，并且验证方式是sso的时候，就是验证是否有效(sso.isSessionValid(userSession, request))。
所以，整个过程和官方文档提到的流程是一样的，如果要实现自己的sso，这是实现SSO接口，
使用authenticateUser来验证不存在usersession的情况，并返回username or null，
而使用isSessionValid来判断一个已经存在的usersession是否有效。
参考上面几个连接文档，实现和已有系统的sso集成，还是比较清晰明了的。
