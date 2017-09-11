---
layout: post
comments: true
title: "improve bitter code: 迷惑的boolean参数"
description: "关于编码规范，代码评审，改进质量方面的总结"
categories: ["质量改进", "improve bitter code", "编码规范", "参数", "可读性", "接口"]
---

__备注: 示例中的代码并不是真实代码的完全拷贝__

### 群里的讨论
前两天在开发群里边，有同事讨论一个api: 增加一个取特定序列值的方法。并给出了以下的原型代码：
{% highlight java %}
public Object getSeqValue(String seqname, String maindb){
    if("1".equals(maindb)){
        DbRunContext.setRegion(DbRunContext.MAINDB);
    }
    // more codes
    return XXDao.queryForObject(seqname);
}
{% endhighlight %}

其中maindb是因为项目支持多数据库的缘故，传入参数maindb来判断是否走主库还是地市库。
我不知道这个"1"是怎么来的，我随口冒出一句：能不能不传递maindb,谁知道要传递什么呢?

于是很快冒出"改进"方案，换成一个布尔值，代码变成这样:
{% highlight java %}
public Object getSeqValue(String seqname, boolean isMaindb){
    if(isMaindb){
        DbRunContext.setRegion(DbRunContext.MAINDB);
    }
    // more codes
    return XXDao.queryForObject(seqname);
}
{% endhighlight %}

很显然这不是一个好的解决办法，我从使用者的角度来说，举了下面的例子:
{% highlight java %}
getSeqValue(seqname, true);
getSeqValue(seqname, false);
{% endhighlight %}
单单看上面的调用方式，谁能很清楚的说明分别是什么意思?

很明显，这相当的困难。所以我建议对方法进行重命名，例如
{% highlight java %}
getSeqValueFromMaindb(seqname);
getSeqValue(seqname);
{% endhighlight %}
当然，可以考虑把原来的方法换成私有的，这样避免代码重复而不会对调用者造成困惑。

### 迷惑人的参数随处可见
在我们的代码里边，类似这样的参数困惑随处可见，除了这种布尔值，还有"1"和"0"
这种魔法数字，字符串，因为调用太频繁了，很多人不愿意给他取个好听的名字。

例如区分调用后台逻辑是否起事务，用得太多了，加上老代码就这么用，所以很多同事
没有意识加个IS_TRANSACTION这类的静态变量。看，代码就变成下面的样子：
{% highlight java %}
commonInvoke(operator, cmd, subcmd, "1");
commonInvoke(operator, cmd, subcmd, "0");
{% endhighlight %}

参数这东西，对看代码的人来说没什么特别的帮助，参数越多越迷惑。
当两个方法调用就差一个布尔值不一样的时候，那是相当痛苦的。
所以最好还是从方法名上进行区分，保持代码的可读性。对于接口方法，公用方法，更是应该如此。

{% assign series_list = "improve bitter code" %}
{% include series_list.html %}
