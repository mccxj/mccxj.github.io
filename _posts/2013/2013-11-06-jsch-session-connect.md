---
layout: post
comments: true
title: "小心jsch的sftp连接泄露"
categories: ["sftp", "jsch", "泄露"]
---

今天早上和一个同事处理一个现网问题，从javacore里边可以看到大量的Connect Thread，如下所示：
```java
Connect thread 192.168.1.100 session" prio=6 tid=0x042d3400 nid=0x1458 runnable [0x04e4f000]
```

堆栈信息如下:
```java
...
com.jcraft.jsch.Session.run(Session.java:1193)
java.lang.Thread.run(Thread.java:619)
```

怀疑是资源泄露了，jsch是一个sftp的工具库。检查jsch的使用代码，可以看到代码是有进行关闭的，如下所示：
```java
    JSch jsch = new JSch();
    Session session = jsch.getSession("caixiaojian", "192.168.1.100", 22);
    session.setPassword("******");
    session.setConfig("StrictHostKeyChecking", "no");
    session.connect();
    Channel channel = session.openChannel("sftp");
    channel.connect();
    ChannelSftp c = (ChannelSftp) channel;

    channel.connect();
    //...
    channel.disconnect();
```

不过从官方的[例子](http://www.jcraft.com/jsch/examples/Sftp.java.html)上看到，最需要关闭的是session对象而不是channel对象。
于是写了一个简单的测试Demo，把上面的代码跑5次，看看能不能重现:

```bash
#jstack -l 3621 | grep Connect
Connect thread 192.168.1.100 session" prio=6 tid=0x042d3400 nid=0x1458 runnable [0x04e4f000]
Connect thread 192.168.1.100 session" prio=6 tid=0x042d0400 nid=0x16c8 runnable [0x04def000]
Connect thread 192.168.1.100 session" prio=6 tid=0x041b4000 nid=0xd38 runnable [0x04d8f000]
Connect thread 192.168.1.100 session" prio=6 tid=0x041b2000 nid=0x166c runnable [0x04bcf000]
Connect thread 192.168.1.100 session" prio=6 tid=0x041b1000 nid=0x450 runnable [0x04b2f000]
```

果然出现了，试着在最后调用一下session.disconnect(),重试一下果然不存在了上述线程了。