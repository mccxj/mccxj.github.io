---
layout: post
comments: true
title: "jekyll插件:嵌入github gist"
description: "给博客增加gist支持"
categories: ["github", "gist", "博客", "gist_tag.rb"]
img: "/images/2013/gist-tag-blog.png"
---

![jekyll插件:嵌入github gist][1]

gist是gtihub的一个代码块功能，用来粘贴一些比较长的代码还是挺有用的。
github page可以直接嵌入gist，并且能显示高亮。
不过我不想太依赖gist，所以**修改成使用pygments高亮的方式**。
看看下面gist_tag.rb这个插件的效果:

{% gist 4648237 gist_tag.rb %}

 [1]: /assets/images/2013/gist-tag-blog.png