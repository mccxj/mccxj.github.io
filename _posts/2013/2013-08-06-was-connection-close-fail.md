---
layout: post
comments: true
title: "WebSphere数据源中的连接被意外关闭案例"
categories: ["WebSphere", "datasource", "connection", "close"]
---

### 频繁创建连接的现象

前阵子维护反馈说，oracle数据库每秒创建连接数过高，而主要来源来自于WebSphere集群所在的主机。
按理说，使用连接池的应用，连接是不会很频繁的。追溯一下所在主机的程序，发现可疑对象是一个使用jdbc轮子的应用。
就是自己写了一套代码封装了jdbc操作，虽然连接是从数据源中获取的。

但是，单纯看代码，的确没发现有什么问题，该关闭的地方也关闭，不存在泄露的情况。再说，这种现象也不是泄露的表现。
另外，由于生产上的数据库连接串，开发人员是没有的，所以生成都是通过jndi的方式来处理的，不存在盗链的代码。

不过，我们还是在日志里边发现一些问题。在SystemOut的日志里边，频繁出现类似下面的异常信息:

``` java
[12-2-27 9:01:29:262 GMT+08:00] 00000038 MCWrapper     E   J2CA0081E: 
尝试对资源 ecareDB 的 ManagedConnection WSRdbManagedConnectionImpl@23f823f8 执行方法 cleanup 时，
方法 cleanup 失败。捕获的异常：com.ibm.ws.exception.WsException: DSRA0080E: 
An exception was received by the Data Store Adapter. See original exception message: 
 Cannot call 'cleanup' on a ManagedConnection while it is still in a transaction..
```

异常的来源，居然是连接的close方法。

### 为什么close会失败

在WebSphere的资料上没有找到相关的描述，但是我在ibatis的文档上找到了相关的描述(ibatis developer guide的12-13页)：

The <transactionManager> element also allows an optional attribute commitRequired that can be true or 
false.  Normally iBATIS will not commit transactions unless an insert, update, or delete operation has been 
performed.  This is true even if you explicitly call the commitTransaction() method.  This behavior 
creates problems in some cases.  If you want iBATIS to always commit transactions, even if no insert, 
update, or delete operation has been performed, then set the value of the commitRequired attribute to true. 
Examples of where this attribute is useful include:

1. If you call a stored procedures that updates data as well as returning rows.  In that case you would 
call the procedure with the queryForList() operation – so iBATIS would not normally commit the 
transaction.  But then the updates would be rolled back.

2. In a WebSphere environment when you are using connection pooling and you use the JNDI 
<dataSource> and the JDBC or JTA transaction manager.  WebSphere requires all transactions on 
pooled connections to be committed or the connection will not be returned to the pool.

Note that the commitRequired attribute has no effect when using the EXTERNAL transaction manager.

### 结论及对策

从上面的描述上看到，WebSphere环境的连接是需要提交事务的，否则会被意外关闭。
并且，如果选择ibatis的事务管理机制，就应该设置commitRequired属性，要么就应该使用spring的事务解决方案。

我们尝试给查询语句增加commit操作，果然异常信息不再出现，并且数据库连接数也大幅度减少了。
看来，在生产系统中还是应该优先考虑成熟的解决方案，不要随便造轮子。