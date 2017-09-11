---
layout: post
comments: true
title: "书评-Node Web开发"
description: "在oschina写的书评"
categories: ["书评", "node"]
---

![Node Web开发][2]

**2012-05-10@oschina.net**
[书评链接][1]

这本书，其实我是看的英文版的。算是一本不错的入门书，相对于官网上那硬邦邦的api手册，
的确是很不错了，里边有些内容有点旧了(例如npm介绍中关于require.path的说法)。
书里边关于node的模块系统的描述是比较好的，非常清晰，是我觉得写得最好的一章，
另外关于node的异步编程模型的介绍也比较好(例如有个关于Fibonacci数列的例子)，
因为最重要就是理解编程模型，即使很多人熟悉客户端javascript编程，
但是刚刚开始node的javascript编程，就会感到非常纠结。

书里边也引用了一些现有比较出名的开源库，例如express和connect，用来构建web应用。
关于这个，我觉得没什么必要，因为这两个库的在线帮助文档是做得非常好的。
我更感兴趣的是书里边一些step by step的demo，可以比较清晰得看到关键的代码是如何实现的。

总体来说，这书是一本不错的入门书，内容少不是问题，
比较现在还没有关于node的更多最佳实践的资料，而且node关注的就是高访问量，小请求，
业务逻辑较少，快响应的特殊场景，需要关注的领域并不是很多，
写得太多反而会埋没重点。有兴趣的同学，真的值得一看。

 [1]: http://www.oschina.net/question/244461_53105?sort=default&p=2#answers
 [2]: /assets/images/node_web_development.jpg

{% assign series_list = "书评" %}
{% include series_list.html %}
