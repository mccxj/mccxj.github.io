---
layout: post
comments: true
title: "连接池泄露定位案例"
categories: ["连接池", "泄露"]
---

前阵子，其他项目组里边有个某项目频繁出现获取不到连接的问题，基本上每一天都出现一次。

打出来的javacore里面，有大量的线程等待获取连接。并且现场反映，从数据库那边监控到大量的空闲连接没有释放。

事实上，这个系统会用到两个数据源，其中有个是正常的。有意思的是，这几个月都没有新版本上载。

走读了java里边的连接获取释放逻辑，虽然获取和关闭的调用到处都是(没封装好)，但最后还是调用一些静态方法来完成的，并且都在finally块中进行了处理，感觉不会存在泄露的问题。

尝试看看能不能重现，运气还不错，在开发环境模拟生产的业务场景(业务功能不多，但逻辑处理流程比较长)进行了压力测试，也的确出现了这种情况。

既然能够重现问题，定位就方便许多了。我知道有Btrace这类神器可以帮助定位，但我没有使用过(下次再尝试一下)。我是通过修改代码来定位的，基本原理就是在获取连接时记录堆栈信息，在关闭时清除，这样出问题的时候就可以找到哪些地方有问题。其实用工具也是类似的做法。

大体上是这样的:

```java
// 用来存储堆栈信息
private static final Map<Connection, Exception> conns = new ConcurrentHashMap<Connection, Exception>();

// 当获取链接的时候
public Connection getConnection(){
	Connection conn = dataSource.getConnection();
	conns.put(conn, new Exception());
	return conn;
}

// 当释放链接的时候
public void releaseConnection(Connection conn){
	conns.remove(conn);
	dataSource.close(conn);
}
```

修改后重新压力测试后，打开记录的堆栈一看，居然是在jsp里边获取的，真是顿时无语。

后来，虽然听说这个问题是其他原因引起的，导致非正常情况下走到这段逻辑，但这个地雷还是以前埋进去的，也怨不了别人。

其实，关于资源释放的逻辑封装，可以参考spring的jdbc封装(回调的方式)，又或许用ThreadLocal、拦截器等方式在整个应用上进行处理。

 