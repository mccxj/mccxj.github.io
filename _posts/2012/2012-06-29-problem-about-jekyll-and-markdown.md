---
layout: post
comments: true
title: "使用jekyll和markdown遇到的问题"
description: "问题和解决方法"
categories: ["jekyll", "markdown", "博客"]
---

### 不能正常生成页面
我是使用windows来弄的，在git shell里边操作。发现有时候页面根本不生成，
即使我把_site下面的文件全部生成，还是没动静。后来发现，需要设置一下环境的字符编码，例如export LC_ALL=zh_CN.UTF-8
可以在启动脚本里边修改,例如修改RUBY_HOME/bin/jekyll.bat，增加
> SET LANG=zh_CN.UTF-8  
> SET LC_ALL=zh_CN.UTF-8

最近发现一个新的方法
>chcp 65001


### tags不能正常发布
tags如果列举多个的话，除了用逗号分割之外，必须在逗号之后加上空格，否则不能在github上正常发布。
但奇怪的是，在本地却是没有问题的。

### <s>列表不能正常显示</s>
<s>这个也比较奇怪，无论是有序列表还是无序列表，只能有部分能正常显示。
我试了很久，发现如果列表项里边没有数字或英文字符就不能显示。</s>
**这个问题已经解决了，原来在每个列表前面需要空行**

### 高亮代码有问题
高亮代码不容易呀，需要python支持，还有安装easy_install Pygments。
但对于某些python版本来说，这样还不行，出现Liquid error: Bad file descriptor。
需要修改[gems\\albino-1.3.3\\lib\\albino.rb][1]

另外，有时候高亮不是很合适，可以选择引用，只要每行前面至少加4个空格或1个tab就可以了。我习惯在每个引用块前后加一个空行。

### 在段落中硬换行
我原来以为不能硬换行，原来markdown也支持，只需要在需要换行的地方后面加至少两个空格就可以了。

### 页面生成太慢
当你的文章越来越多的时候，页面生成的时间会变得非常夸张，jekyll没有提供参数来控制。
为了解决这个问题，有个办法就是过滤某些没有变化的文件，因为大多数情况下，我们只是对特定的文件进行修改而已。
摸索了很久，最后我直接拿jekyll源码开刀，见[<s>让include和exclude支持正则</s>][2]和[add glob support to include,exclude option][3]。

 [1]: https://gist.github.com/1185645
 [2]: https://github.com/mccxj/jekyll/commit/44822a252e2ce1142e3293f91285e0ced3ba2fe1
 [3]: https://github.com/mojombo/jekyll/pull/743
