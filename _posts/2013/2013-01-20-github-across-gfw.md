---
layout: post
comments: true
title: "抢票插件搞得上github page都要轻功"
description: ""
categories: ["博客", "github", "轻功", "ssh"]
img: "/images/2013/github-across-gfw.png"
---

![抢票插件搞得上github page都要轻功][5]

春运临近，浏览器抢票软件也变得流行，没回来那几天就看到新闻说:[12306 抢票版插件拖垮 Github 服务器][1]
，没想到回来之后就发现github page不能用了，正确的说应该是github的子域名都不能用了，看来是贴倒部和宫刑部的新春贺礼来的。

## 抢票插件和github什么关系
抢票插件引用了github上的一个js文件，但github有个安全检测，当访问比较频繁的时候就会直接返回403 forbidden。
然后作者没多想就在插件里加了个重试机制。如果返回的是403就每5秒重试一次，并且是永久重试，结果github认为你访问的更频繁了于是一直返回403。
可想而知这就成了死循环，使用插件的用户一多，对github而言就产生DDOS了。
换句话说，这是github的一种安全机制而已，抢票插件和github基本没什么关系，有关部门的做法更是弱智得不行呀。:)

## 轻功之ssh
以前用过free gate这种东西，不过不大稳定，而且感觉风险比较高。呵呵，你懂的。  
[ssh使用帮助][2],跟人感觉用ssh命令行配合chrome插件是最方便的。  
[免费ssh代理][3],速度还不错，不过不是特别稳定。  
[Nutz福利之轻功][4],应该不错,不过我还没用上。

 [1]: http://www.oschina.net/news/36770/12306_ticket_helper
 [2]: http://www.ssh110.com/help.html
 [3]: http://blog.onlybird.com/%E5%85%8D%E8%B4%B9ssh%E4%BB%A3%E7%90%86
 [4]: http://wendal.net/2013/0108.html
 [5]: /assets/images/2013/acrossgfw.jpg
