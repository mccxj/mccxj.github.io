---
layout: post
comments: true
title: "improve bitter code: 多掌握一门语言"
description: "关于编码规范，代码评审，改进质量方面的总结"
categories: ["质量改进", "improve bitter code", "可读性", "语言", "学习"]
---

__备注: 示例中的代码并不是真实代码的完全拷贝__

### 语言是有区别的
语言是有'好坏'之分的，在不同的应用背景下，有些语言就是比其他一些语言更为合适！
语言是有它的设计目的，有些是为了效率(c)，有些是为了快乐编程(ruby)，有些则是了并发做准备(erlang)。
学习不同类型的语言也有助于锻炼编程思维，像c,java,ruby,javascript,erlang都教会了我许多东西。

来到这边之后，主要工作语言是java，但并不是说其他语言没有了用武之地，我一直在寻找使用其他语言的机会。
下面举些例子，来说明一下。由于我以前大量使用ruby语言，的确我也是特别喜欢。
所以例子以ruby为主，但并不是说其他语言没法做到，或许有更好的做法也不一定。

##### 临时生成xml文件
前几天维护那边需要我提供一个xml文件，它提供了一组txt数据:每半个小时的登陆及操作统计。
我需要生成一个xml，我不熟悉excel操作(花了几分钟尝试了一下)，所以我选择使用脚本来生成。
脚本只花了一分钟,比我预想得还要顺利。
{% highlight ruby %}
open('a.xml', 'w') do |f|
  open('a.txt', 'r') do |ff|
    ff.readlines.each_with_index do |l, i|
      dl, cz = l.split(' ')
      f << "<data><seq>#{i+1}</seq><czvalue>#{cz}</czvalue><dlvalue>#{dl}</dlvalue></data>"
    end
  end
end
{% endhighlight %}
对于这种一次性的文本处理，java真是太繁琐了，没有任何优势可言。

##### 定位数据问题
曾经有次上版本之后发现一个菜单树不能使用。通过代码定位应该是菜单出现循环依赖了(菜单在表中是父子关系的)。
同事也把生产的数据拿过来了。数据格式类似下面的，第一列是菜单id，第二列是父菜单id，还有其他的数据列。
{% highlight java %}
ITEM1 ITEM2 OTHERS...
ITEM2 ITEM3 OTHERS...
ITEM4 ITEM2 OTHERS...
ITEM5 ITEM3 OTHERS...
{% endhighlight %}
问题已经变成：父子是否出现循环。当时我用了几分钟写了一段脚本来找到出现循环的菜单项，最后发现是其他项目组增加的菜单有问题。

##### 传数据的偷懒方法
以前还是老邮箱的时候，由于网络隔离，需要把测试相关的东西通过邮箱发到测试环境去。
我每次都是这么弄的：通过网页登录邮箱，新建邮件，把已经打包加密好的文件作为附件添加，保存为草稿，退出。
次数多了，真的很无聊，有时候一天要弄好几次。

我知道有种测试方式叫自动化测试，所以我用watir(一个用ruby写的自动化测试框架)模拟了整个过程。后来我把这个脚本调用集成到菜单右键上，
只需要把需要上传的文件选中，再触发一个右键菜单。打包加密，上传文件就搞定了。那时候的感觉就是，世界清净了。

##### 实现codediff应用
有些框架工具有非常高的生产效率，ruby on rails就是其中一个。在项目中进行codediff使用的工具，主要是通过这个框架开发的。
说真的，这么一个东西，用java实现也不是什么难事。但对我来说，1k行的ruby代码量，sql一行也没有写，维护上要省心得多。

只要能够对项目有意义，谁会关心我是用什么来实现的呢?

##### 部署应用
这是个cmo比较熟悉的领域。大多数情况下，我们会选择bash等shell工具。
不过我在开发codediff工具的时候，我想在本地直接就把应用部署到开发环境上去。

windows的bat很烂，shell等有比较多限制，所以我又选择了脚本语言。
使用perl,ruby等脚本来做管理脚本并不少见，我这里选择一个好几年前写的脚本作为例子：
{% highlight ruby %}
cmd = Proc.new do |shell|
  shell.cd("/opt/xxx")
  print "current directory:",shell.pwd.stdout
  print shell.sh("/opt/getupdate.sh").stdout
  shell.exit
end
 
Net::SSH.start('192.168.1.32', :username => 'root', :password=>'psword') do |session|
  shell = session.shell.sync
  cmd.call(shell)
end
{% endhighlight %}

上面的例子比较简单，现实的例子是本地打包，通过ftp上传，解压发布，重启服务器，每个步骤都带交互功能。
脚本语言和普通shell相比，具有完整语言和跨平台的优势，用来做系统管理是一个不错的选择。

### 多掌握一门语言
整篇内容没有涉及编程技巧上的东西，但这却是我最想表达的内容。
在这里，我用Martin Fowler中国行的时候，[图灵社区对他的采访内容][1]来结束，
里边有关于学习编程语言建议的内容。

___图灵社区：您曾经建议过程序员应该每年学习一门编程语言。___

___M___：这实际上是Pragmatic Programmers提出的建议，是他们出版的书中的一条建议，我本人很赞同。我认为应该有意地学习一些新的语言，特别是那些和你所熟知的语言的运作方式相差甚远的语言，所以不要在相似的语言上浪费时间。如果你是一位Java程序员，那么C#对于你来说就太过熟悉了，你需要学习一些很不同的语言，比如说Lisp，或者Clojure；如果你是一位Lisp程序员，你也不需要学Clojure，因为那就是另一种Lisp而已。总之你应该尝试一些很不同的语言。

 [1]: http://www.ituring.com.cn/article/2083

{% assign series_list = "improve bitter code" %}
{% include series_list.html %}
