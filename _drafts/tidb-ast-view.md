---
layout: post
comments: true
title: "tidb源码学习之ast可视化"
description: "对tidb中ast包的源码学习记录，增加可视化方便调试"
categories: ["tidb", "ast", "可视化", "源码"]
---

ast是通过parser.y生成的，一个sql的ast结构还是比较多的，不是那么直观，如果有个可视化工具会比较好。

大概实现思路如下:
- 文本化，实现一个Visitor针对不同的ast记录可视化信息(关联字段)，直接输出
- 图形化，实现类似上面的Visitor,生成plantuml文本，送在线生成地址生成图片
- 提供一个网页接口
