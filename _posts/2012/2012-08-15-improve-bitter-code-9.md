---
layout: post
comments: true
title: "improve bitter code: 没有行为的封装"
description: "关于编码规范，代码评审，改进质量方面的总结"
categories: ["质量改进", "improve bitter code", "可读性", "javabean", "封装", "行为", "值对象"]
---

__备注: 示例中的代码并不是真实代码的完全拷贝__

### 常见的对象
javabean这种java类是非常常见的，它有一些属性和相应的get/set方法。
我们还会看到pojo,vo这些概念，还有我们使用的一种特殊对象cbo，在外观上都是这种形式。

在项目的代码中，我们经常大量这种对象，它有私有变量，对每个私有变量都提供了get/set方法,除此之外，没有其他方法。
这种情况下，通常只需写好变量，然后用工具生成相应的访问器方法。

在最近修改一处代码的时候，就看到下面的逻辑，并且这段代码在多个文件出现过。
{% highlight java %}
PrepaidInput prepaid = ...// load from request

String start = prepaid.getStartTime();
String end = prepaid.getEndTime();

if(StringUtils.isNotEmpty(start)){
	String newstart = start.replaceAll("-", "");
	if(newstart.length() > 8){
		newstart = newstart.substring(0, 9);
		prepaid.setStartTime(newstart);
	}
}

// 继续处理结束时间end
{% endhighlight %}

### 封装还需要行为
上面的代码是很常见的处理方式，对象只是传值的作用。给变量封装到方法里边，提供了get/set方法。
本质上来说，就像换了个马甲，和直接暴露数据差不多。类似的代码出现很多，先看下面简单点得例子：
{% highlight java %}
if("1".equals(subs.getStatus())){
         //TODO
}
{% endhighlight %}
没有行为，顶多算是基于对象的编程。
从行为及职责考虑，这里暴露了状态值1，因为调用方必须理解这个值的作用。
封装，除了封装数据，还得封装出行为的样子来。如：
{% highlight java %}
if(subs.isActive()){
         //TODO
}
{% endhighlight %}

对于最前面的代码，修改方式有很多，有一种方式就是在set里边, 组装对象的时候调用set方法顺便把格式给处理了。  
没有规定说get.set都会对应一个私有变量，也没有规定get.set要成对出现，如果这是在内部使用的变量，为什么要暴露出来?  
同样除了get/set也没有规定不能提供其他的方法。
{% highlight java %}
void setStartTime(String startTime){
    String newstart = startTime;
	if(StringUtils.isNotEmpty(startTime)){
		String newstart = startTime.replaceAll("-", "");
		if(newstart.length() > 8){
			newstart = newstart.substring(0, 8);
		}
	}
	this.startTime = newstart;
}
{% endhighlight %}

### 现实情况
可惜的是，系统中重要的数据载体是cbo，由工具生成的，类似于vo。所以cbo没什么像样的行为。   
对于开发java web的同学，习惯cbo,mvc这样的流水线作业，使用面向对象编程，对封装行为并不习惯。  
引用郑大的话:我们需要更好的封装，通常的做法是封装出行为。行为从哪来，从实际需求来。  
所以，还是得在平时工作中有意识、有针对性的实践才行呀!

{% assign series_list = "improve bitter code" %}
{% include series_list.html %}
