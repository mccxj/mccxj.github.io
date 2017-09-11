---
layout: post
comments: true
title: "git分支让github page用上jekyll插件"
description: "github page不支持page功能,使用分支来处理"
categories: ["github", "博客", "git", "jekyll"]
img: "/images/2013/jekyll-plugin-with-git-branch.png"
---

![git分支让github page用上jekyll插件][2]

**本博客已经在2013-3-9转换成octopress了,比这种手动方式要方便很多。**

github page是个不错的应用，可惜对jekyll有比较多的限制，特别是插件方面。
为了解决这个问题，我选择了分支来处理这个，大约就是source分支保存未编译的内容，
master分支保留生成的网站。下面是大概的操作过程，针对已有博客的迁移。

### 迁移过程

首先，到github上手动打一个分支出来，叫source分支。

接着，处理master分支，清除所有内容。注意git pull的功能是让本地可以识别到远程分支。
**.nojekyll文件**是让github page不启用jekyll生成网站，而是直接使用目录下的内容。
并把所有带下划线的目录都过滤掉。
{% highlight bash %}
cd mccxj.github.com
git rm -fr *

touch .nojekyll
git add .nojekyll

# add _*/* to .gitignore
vi .gitignore

git commit -a -m "remote all pages"
{% endhighlight %}

下面，继续处理source分支，其实基本保持不变就可以了，主要是生成网站内容。
**带t的参数是让source跟踪远程source分支**。我的jekyll是采用最新源码装的，命令参数有些变化，
请参考jekyll帮助，
{% highlight bash %}
git checkout -t origin/source

$ git branch -a
  master
* source
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/source

# generate page to _site
jekyll build
{% endhighlight %}

最后，切回master分支，并**拷贝网站内容**到根目录，然后把内容提交并push到远程即可。
{% highlight bash %}
git checkout master

cp -r _site/* .

# add then commit
git add / git commit

# push to remote branch
git push origin master
{% endhighlight %}

### 新写作流程
现在已经迁移完成了，下面介绍一些新写作流程。

首先，**注意要在source分支上工作**，在提交到远程之前都是一样。
{% highlight bash %}
git checkout source

# rake post title="xxxx"
# write something
git add xxxx.md
git commit -m "add new post"

# jekyll build
jekyll serve
{% endhighlight %}

当你确认完成，并生成网站内容后，切换到master分支处理。
**注意需要提交两个分支**，例如使用git push可以同时提交两个分支。
{% highlight bash %}
git checkout master

cp -r _site/* .

# add then commit
git add / git commit

# push to remote branch
git push
{% endhighlight %}

还不算麻烦吧，其实我是尝试切换到[octopress][1]，发现有不少地方出现问题，才采用这种方式的。Enjoy It!

 [1]: http://octopress.org/
 [2]: /assets/images/2013/jekyll-plugin-with-git-branch.png