---
layout: post
comments: true
title: "一次INTERNAL_SERVER_ERROR的问题分析"
description: "晚上版本上线后，发现工号进入首页后页面空白，显示INTERNAL_SERVER_ERROR"
categories: ["was", "ihs"]
---

## 问题现象

晚上版本上线后，发现工号进入首页后页面空白，显示INTERNAL_SERVER_ERROR

## 过程回顾

1. 通过fiddler抓包，发现某个请求出现500错误
2. 检查应用，was，ihs日志,没有发现有效日志
3. 发现只有部分工号有问题，开始怀疑存在数据问题，准备导数据回测试环境验证
4. 同时，新建一套ihs和was环境进行验证
5. 新环境验证，发现原来失败的工号可以正常，怀疑环境配置有问题。同时发现请求超过4s就会返回500错误
6. 尝试直接访问原was,发现是正常的
7. 对比两个环境配置，was和ihs配置是一样的，但plugins-in参数ServerIOTimeout有差异，旧的是-1,新的为0
8. 修改新的为-1,访问问题重现
9. 准备修改为0，问题修复

## 技术分析

关于ServerIOTimeout参数的描述，可以参考以下链接。

* [参数描述](http://www-01.ibm.com/support/docview.wss?uid=swg21219808)
* [官方推荐](http://www-01.ibm.com/support/docview.wss?uid=swg21318463)

取值为负数的情况，描述如下:
> ServerIOTimeout value can be either positive or negative. If positive, when the ServerIOTimeout pops, the plug-in will not mark that server down. If negative, when the ServerIOTimeout pops, it will mark that server down. If your application uses HttpSession object, then there will be session affinity in play, so it would be best to choose a negative ServerIOTimeout value, to ensure that the retry will not be sent back to the same server that just timed-out. Since that server will be marked down, the retry will go to a different appserver in the cluster.

简单来说，关于参数的取值都是需要权衡的。ServerIOTimeout表示ihs和was之间连接的读写超时时间，取值有三种值:

* 0 表示一直等待was响应，这样可能会造成长时间不响应，连接也不释放
* 正数 表示读写超时时间，超时之后会重试，这样可能会造成业务重复受理
* 负数 和正数一样，不过超时之后会认为当前was为下线状态，然后重试其他服务器，对于无状态的请求同样可能有重复受理的问题。在上面的问题中，ihs后面有4个was，所以刚好看上去就是超过4s会超时

## 小总结

* 优化应用日志很重要，特别是那些没什么用的异常日志，需要定期优化现网日志
* 最小化的可重现问题的环境
* 熟悉常见调试工具，如fiddler,telnet,wget,curl,tcpdump等
* 修改参数应该慎重，知道参数的含义，不要想当然。这次就是以为-1和0是没有区别
