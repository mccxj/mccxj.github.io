---
layout: post
comments: true
title: "commons-logging源码学习"
description: "commons-logging源码学习"
categories: ["commons-logging", "源码"]
---

commons-logging是一个流行的logging统一接口，代码非常简单，具体的logging可以使用不同的实现，如log4j，jdk log等，即使没有这些，它还是能在控制台上输出，它可以帮你选择一种合适的logging实现。

commons-logging有LogFactory和Log两个主要接口，LogFactory实现了如何找到合适的Log，而Log是一个标准的logging接口。

LogFactory是一个抽象类，它的具体实现通过以下顺序确定:

1. 系统属性org.apache.commons.logging.LogFactory
2. META-INF/services/org.apache.commons.logging.LogFactory
3. commons-logging.properties的org.apache.commons.logging.LogFactory
4. org.apache.commons.logging.impl.LogFactoryImpl

基本上，都会是LogFactoryImpl这个实现。

另外，commons-logging可以采用SPI指定LogFactory实现，不过commons-logging并没有使用标准的ServiceLoader来处理，可能是由于commons-logging要兼容老的java版本吧。

还有，commons-logging.properties是支持重新加载的，按道理classpath加载资源是有缓存的，
见LogFactory中的getProperties，它不是直接使用getResourceAsStream，而是采用getResources/getResource,对得到的URL进行openConnection得到URLConnection对象，对它设置setUseCache为false，这的确是个小技巧。

对于Log实现，如果没有通过commons-logging.properties的的org.apache.commons.logging.Log这个key指定的话，是按照下面的方式尝试获取的:

1. org.apache.commons.logging.impl.Log4JLogger
2. org.apache.commons.logging.impl.Jdk14Logger
3. org.apache.commons.logging.impl.Jdk13LumberjackLogger
4. org.apache.commons.logging.impl.SimpleLog

很明显，只要有log4j的jar包就会选择log4j，否则就会选择jdk logging或者System.err。

所以正常情况下，不需要设置这个commons-logging.properties里边的org.apache.commons.logging.Log对应的属性值(其实这个配置文件都是不需要的)。

曾经就有项目，依赖一个自己写的jar包，就把commons-logging.properties打包进去，里边配置成SimpleLog，导致很多框架的日志不能输出到log4j指定的文件中去。
