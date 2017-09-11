---
layout: post
comments: true
title: "使用svn命令进行每日codediff"
description: "在公司很多人对svn命令不了解，对我开发的每日codediff工具也不甚清楚"
categories: ["实践", "svn", "codediff", "工具"]
---

## 什么是codediff?
每日codediff是一项简单易用，行之有效的敏捷实践，可以保证项目进展，让项目中的新人得到学习的机会，让代码规范得到更有效贯彻。
具体做法是，每天早上站立晨会之后，几个人围在一起，使用版本控制工具的diff功能，
逐行浏览前一天提交的代码，提交者对修改的代码进行介绍，讲讲代码是什么功能，为什么要做这样的修改。
当然其他人可以提出异议，并且通常会记录评审意见，以便后续跟踪。

和每周的代码评审相比，每日codediff虽然局限于每个代码片段，但时效性更强，两者互补不足，都是保证代码质量的有效手段。

## better svn, better codediff
在我们的项目团队里边，一直也是使用这种方式进行处理。
公司使用的是svn，不过由于环境的限制，svn性能并不是特别理想。
经常把大量时间用在svn log,diff的等待过程中，整个codediff也因此需要比较多时间，有时需要1个多小时。
长此以往，很多人对每日codediff不重视，参与度不高，对于我来说，评审问题也很不好跟踪。

后来我想找一专门为codediff服务的工具，考察了一些开源项目，但总觉得不是特别合适我们团队的需求。
我的要求也很简单，为svn服务，操作简单，便于跟踪，而不需要正规的评审流程。后来，我决定自己开发一个。

我的做法也很简单: 定时从svn上取出最新提交记录，并把相关文件记录到数据库中去。
再弄个网页展示出来，codediff和评审通过网页来进行，

原型一出来，就在团队中试用了一段时间，效果挺好的，通常只需要10来分钟就可以了，大大提高了效率。
现在几个同事还把这东西推广到其他项目中去，反应都还不错，让我着实感到非常有面子:)。

## 更多细节
后来，有些同事比较感兴趣提交记录是怎么弄出来的。
因为很多人平时只是使用海龟来操作，命令行基本没使用过，所以不清楚也不奇怪。所以解释一下是怎么做的，也是我写这篇博客的重要原因。

整个工程主要使用[ruby on rails][1]和[twitter bootstrap][2]做页面.
通过[jruby][3]来跑，定时任务使用[quartz-jruby][4]来做，提交记录通过svn命令来获取，我这里主要说说svn命令。

当然首先需要安装svn命令行客户端，然后通过[Runtime.exec()][5]来获取输出信息，解析这些信息入库即可。

{% highlight bash %}
# 获取最新版本号,使用正则/Last Changed Rev: (\d+)/得到最新版本
svn info http://hustoj.googlecode.com/svn/trunk
# 获取svn log，这个解析比较麻烦，需要用正则分组
svn log -l 1 -v http://hustoj.googlecode.com/svn/trunk
# 获取diff文件，对于更新(M)的文件
svn diff -c 1659 http://hustoj.googlecode.com/svn/trunk/web/admin/contest_list.php
# 获取完整文件，对于增加(A)删除(D)的文件，需要直接取版本
svn cat http://hustoj.googlecode.com/svn/trunk/web/admin/contest_list.php@1655
{% endhighlight %}

不过在实际处理的时候，默认的diff取出来的上下文太少，所以我使用了外部diff工具:
{% highlight bash %}
svn diff -c 1659 --diff-cmd /usr/bin/diff -x "-U20" http://hustoj.googlecode.com/svn/trunk/web/admin/contest_list.php                    
{% endhighlight %}

不过通过程序执行命令获取数据时还存在一点问题:获取不到diff数据。
为了解决这个问题，我再封装了一个shell文件来调用。
{% highlight bash %}
#!/bin/bash
svn diff -c $1 --diff-cmd /usr/bin/diff -x "-U$2" $3
{% endhighlight %}

总的来说，实现的难度并不大，只是遇到了一些小问题，能够把一个想法化成现实，并且确实提高了效率，还是挺满意的。

 [1]: http://rubyonrails.org/
 [2]: http://twitter.github.com/bootstrap/
 [3]: http://jruby.org/
 [4]: https://github.com/mccxj/quartz-jruby
 [5]: http://docs.oracle.com/javase/6/docs/api/java/lang/Runtime.html#exec(java.lang.String)
