---
layout: post
comments: true
title: "关于日志打印行号的性能案例"
description: "上个版本快上线的时候，发现系统整体变慢了。观察页面请求耗时，存在不同程度的性能倒退。观察日志，也没有发现有明显的异常"
categories: ["异常检测", "性能", "日志"]
---

## 问题描述

上个版本快上线的时候，发现系统整体变慢了。观察页面请求耗时，存在不同程度的性能倒退。观察日志，也没有发现有明显的异常。

## 分析过程

页面请求后面还有一堆的接口服务，首先需要定界，还好有服务调用链可以观察。挑选了一个耗时变化比较大的请求，查看调用链信息。发现基本都消耗在接口页面请求的服务上了。

![服务调用链](/assets/images/2020/12_1.jpg)

了解具体应用，定位就相对简单了。定位到是哪个系统后，本地同时发起多次请求，然后打一下堆栈，就可以看到基本都挂在日志输出上面。可以看到堆栈上有获取获取堆栈信息的方法调用。

```
at java/lang/J9VMInternals.getStackTrace(Native Method) 
at java/lang/Throwable.getInternalStackTrace(Throwable.java:268(Compiled Code)) 
at java/lang/Throwable.printStackTrace(Throwable.java:520(Compiled Code)) 
at java/lang/Throwable.printStackTrace(Throwable.java:301(Compiled Code)) 
at …/debug/LocationInfo. (LocationInfo.java:84(Compiled Code)) 
at …/DebugLogImpl.doLogPre(DebugLogImpl.java:492(Compiled Code)) 
```

看到获取堆栈信息，我就有大概知道是什么回事了。一开始我看了一下对应的log配置，并没有发现有添加打印行号的日志格式，回头去看代码才定位到最初的地方。

## 原因梳理

其实这个代码很早就有的了，碰巧是其他几个地方改动把这个问题暴露出来了。有三个点：

- 原本系统在打印日志是warn级别的，这次版本加入调用链改造把日志级别调整成debug，方便后续做动态调整。
- 系统用的是另外一套系统带过来的一个日志实现，代码里边默认就会获取行号(调用堆栈的地方)，而不是通过log格式配置实现的。
- 系统里边有一段对象转换的逻辑，递归调用的并且调用非常频繁，在代码里边有写调试日志，日志级别用的debug。

几个点结合到一起，问题就暴露出来了。单单看一个问题并不复杂，很多时候问题的原因也是非常简单的，但是在复杂的系统调用中，就没那么清晰了，需要借助一些监控分析工具。

## 知识复盘

据我了解，在应用代码中主动new Exception的做法，除了用来控制业务流程之外(不推荐)，还有两个应用场景，主要用在一些框架上：

- 一种是为了记录当前堆栈，主要用来对象池技术上用来调测对象泄露。
- 一种是为了拿到调用行的行号，主要用在日志框架上。
这里碰到的就是第二种应用场景，虽然一些开源日志框架带有这个功能，一般是要谨慎打开的。至于对象池上的应用，例如数据库连接池，一般也是建议有泄露的情况才打开这个特性。

我们知道try-catch并没有多大影响，但new Exception是有比较大消耗的，因为会记录调用堆栈，需要避免频繁调用。

也有一种技巧，是不让异常记录堆栈，这种做法要区分是哪里的问题，可能就得添加一些附加信息，例如自定义错误码。

```
public class EmptyStackTrace extends XXException {
    @Override
    public synchronized Throwable fillInStackTrace() {
        return this;
    }
}
```
