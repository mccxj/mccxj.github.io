---
layout: post
comments: true
title: "jekyll插件:支持URL跳转"
description: "让github page支持redirect的功能"
categories: ["github", "alias", "博客", "alias_generator.rb"]
img: "/images/2013/jekyll-alias-generator.png"
---

![jekyll插件:支持URL跳转][3]

github page不支持.htaccess功能(参考[Blogging on Jekyll: URL Redirects][1])，
所以发生URL调整的时候，无法让原有路径自动跳转到新路径。
[Alias generator][2]这个插件提供了一个解决方案，就是生成多一个页面，采用auto refresh的方式跳转到新路径。

这次我也调整了一些博客的路径，我也采用了这个方式。不过我只对当前已有的页面生成一次，以后的就不用这个插件了。
另外，我不想去修改每个页面的alias标签，所以我调整了代码，只对我的url规则进行处理。下面是我使用的版本。

{% gist 4645990 alias_generator.rb %}

 [1]: http://rawsyntax.com/blog/blogging-on-jekyll-url-redirects/
 [2]: http://github.com/tsmango/jekyll_alias_generator
 [3]: /assets/images/2013/jekyll-alias-generator.png