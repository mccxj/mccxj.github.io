---
layout: post
comments: true
title: "避免对象未知状态"
description: "java object state"
categories: ["java", "异常", "构造方法"]
---

今天在使用一个库的时候就遇到一个很纠结的问题。

通常我们应该避免对象new的时候出错，一种常见的设计方案是采用初始化方法。不过也有new出异常的情况，这种情况应该确保对象占用的资源能够被释放。

不过今天遇到的这个情况，是一个链接短信网关的库，假设是Proxy对象，它在初始化的时候，生成2个线程，一个是心跳线程，一个是接受消息的线程。
在链接不上的情况下，初始化会抛出异常，但是仍然会尝试重连。

```java
public class MyProxy extends SMProxy {
    private final RecvMsgHandler handler;
    public MyProxy(Map props, RecvMsgHandler handler){
        super(props);
	this.handler = handler;
    }

    @override
    public void onRecvMsg(Message msg){
        handler.recvMsg(msg);
    }
}
```

上面是我想实现的构造方法，打算给他注入一个回调方法，让它在特定事件可以通知到我。
由于可能出异常，导致这个对象只是构造了一部分，一旦出现回调的时候，会存在指针异常。

另外一个问题是，本来我想管控一下所有的链接，这样可以随时控制数量。
不过由于出异常的时候，我没法拿到这个对象，所以管控不到这个可能重连的对象，相当于漏掉了这个连接。

这2个问题，特别是第二个，还真的没有什么好的解决方案。

总而言之，这个SMProxy的设计缺陷太明显，看来要绕过这个类的实现才行。