---
layout: post
comments: true
title: "使用org.json库进行xml和json转换存在的问题"
description: "org.json的使用问题"
categories: ["json", "xml"]
---

org.json库中提供一个xml和json进行转换的工具类，XML.java

使用方式如下：
* xmlstr = XML.toString(jsonstr)
* jsonstr = XML.toJSONObject(xmlstr).toString()

中间层原有代码使用这种方式进行格式转换，不过存在一些问题：
* json转换为xml的时候，对带content字段的节点，是直接生成文本，而不是<content>xx</content>
* xml转换为json的时候，会对指为整形(还有true/false/null等)的字符串尝试进行转换，变成原生类型

为了避免这两个问题，对org.json库的XML.java进行了一些修改:
* 去掉content字段的特殊处理
* 去掉整形字符串尝试转换的逻辑

见https://github.com/mccxj/JSON-java

经验教训: 以后引用第三方库的时候，要小心呀，避免触碰到一些特殊开关。