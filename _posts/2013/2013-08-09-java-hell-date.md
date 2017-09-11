---
layout: post
comments: true
title: "java中的日期工具"
categories: ["java hell", "date", "java"]
---

### 蹩脚的日期API

众所周知，jdk自带的日期API用起来非常蹩脚，属于JDK中设计较差的典型。

首先我们来看看常见类的分工：

* Calendar实现的是日期和时间之间的转换,其他就没什么用了
* DateFormat用来格式化和解析日期字符串
* Date用来表示日期和时间信息

下面我们再看看其中很混淆的一些设计:

* 有两个Date类，分别是java.sql.Date和java.util.Date，前一个继承后一个，不要搞混了，通常我们是使用后一个。*
* java.sql是用于数据库类型的，这个Date是一个单纯的日期类型，意思是没有时间的概念，如果要时间，应该选择java.sql.Timestamp。这是个很悲催的设计，把原来的Date功能阉割了。
* 注意Date的构造方法参数是很神奇的，年份是减去1900的数，月份是从0到11来表示的。
* Date的getTime返回的时间是相对于“1970-01-01 00:00:00”的毫秒数差值
* 日期的调整、差距计算非常麻烦
* 这些类都不是线程安全的，特别是格式化功能，经常有人掉坑，是重灾区来的

### API的替代品

日期API实在是太烂了，幸好有其他开源选择，例如著名的Joda-Time，有兴趣的可以去试试。

另外，还有最新的关于日期API的规范:JSR-310。官方的描述叫做“This JSR will provide a new and improved date and time API for Java.”，JSR-310将解决许多现有Java日期API的设计问题。比如Date和Calendar目前是可变对象，你可以随意改变对象的日期或者时间，而Joda就将DateTime对象设计成String对象一样地不可变，能够带来线程安全等等的好处，因此这一点也将被JSR-310采纳。

### 时间的度量

有两个特殊的API是和计时有关的，就是System.currentTimeMillis和System.nanoTime。两个的区别和用法如下:

currentTimeMillis的值是相对于“1970-01-01 00:00:00”的毫秒数差值，跟new Date().getTime是一样的，因为new Date()就调用currentTimeMillis作为参数。大家应该对这个是比较熟悉的。

nanaTime是jdk5才增加的API，能够提供更为精确的时间度量(纳秒呀，精度太高了，应该是系统时钟的近似值)。不过，它返回的是一个相对时间，而不像currentTimeMillis是一直增长的。所以要让时间有意义，必须用上时间差，就是用两个nanaTime来相减。

简单来说，currentTimeMillis表示的是精确时间点的概念，nanaTime表示的是相对时间差的概念。用来表示一段时间差的话，nanoTime可以提供更好的精度。

顺便提一下，JDK没有度量微妙的API，需要的话只能自己模拟了。

### 更多时间工具

如果是多线程编程的话，就不能不忽视java.util.concurrent工具包。它也提供了一些时间方面的工具，如TimeUnit类，简单来说，它是一个单位转换的辅助类，可以用更加直观的时间单位来操作。例如休眠3s,可以用下面的代码:

```java
TimeUnit.SECONDS.sleep(3d);
```

这个类也用于这个包里边许多带超时机制的API中，例如使用lock的API是这样使用的的:

```java
Lock lock = ...;
if (lock.tryLock(50L, TimeUnit.MILLISECONDS)) {
    try {
        // manipulate protected state
    } finally {
        lock.unlock();
    }
} else {
    // perform alternative actions
}
```

关于这个并发工具包的内容，以后有机会再介绍。