---
layout: post
comments: true
title: "认真看待java web基础"
description: "对java web的其他认识"
categories: ["web"]
---

很多人一开始接触java技术，做的项目都是web相关的，搞过servlet,jsp,struts,springmvc,用过tomcat，终于感觉是web开发没什么问题了，简历可以标上熟悉java web开发。

现在我们从基本的war包开始，重新梳理一下java web基础，权当复习。

曾经我面试问一个小问题，你知道war包是怎样的结构么？有人会说有src,还有webroot,webapp之类的目录，有人说有平级的classes和WEB-INF目录.

最常见的格式是这样的

```
xx.war
  WEB-INF
    classes
      com/xxx/A.class
      conf
         xx.properties
      log4j.xml
    lib
      *.jar
    web.xml
  *.jsp
```

可以看到是没有源代码的。那么

* 传说中java的class文件可以一次编译到处运行，那么源代码采用GBK还是UTF-8会有影响么?
* 如果lib有2个不同版本的jar，例如spring2.5,spring3，还能安心干活么?
* 如果classes有个class文件不小心被打到jar包去，遗忘在lib目录，以后更新classes会不会炸了?
* log4j.xml放到conf目录会有问题么? 有什么区别没有?
* 有人写了个Niubility的类放在yy.war, 为什么我就调用不到呢，明明同一个猫上跑的?
* 听说有servlet3支持异步可厉害了，但放个demo到tomcat6会挂了，我lib明明有高大上的servlet-api.jar?
* 听说web.xml里边可以配置监听器listener，但它监听什么?
* 为什么不建议把jsp放在war的根目录下?

如果可以轻松无压力，那么说明java web基础还是可以的。

@首发于公司内部群组
