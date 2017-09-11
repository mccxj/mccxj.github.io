---
layout: post
comments: true
title: "improve bitter code: 判空的处理"
description: "关于编码规范，代码评审，改进质量方面的总结"
categories: ["质量改进", "improve bitter code", "可读性", "空对象", "设计模式", "约定", "assert", "接口"]
---

__备注: 示例中的代码并不是真实代码的完全拷贝__

### 预料之外
预料之外的情况，防御性编程经常会让整洁划一的代码变得混乱。
我们经常会担心各种各样的空值，例如null不行，换成空字符串，集合为空或取不到值，怎么办？
字符串都是空格，要不要trim一下？
往一个很深的属性中取值，判空的逻辑太多了？
{% highlight java %}
if(Objects.isNotEmpty(subs)){
	List<Attr> subsAttrs = subs.getAttrs();
	if(Objects.isNotEmpty(subsAttrs)){
		for(Attr attr : subsAttrs){
		 	if(attr != null){
		 		//do something with attr
			}
		}
	}
}
{% endhighlight %}
诸如此类的情况每天都会出现，的确是很麻烦的。
更多材料可以查看一下[Google Guava关于避免null的描述][4]

### 用assert减少麻烦
有一种叫assert的技巧，用来保证参数必须满足一定的先决条件，不满足则无法继续。
这作为和客户端代码之间的协议，即使有异常情况，也应该在客户端先处理好。
当然，assert通常用来防御不正常的调用。以spring这种ioc框架为例，通过get,set无法保证某些必填的值都设对了，
所以在调用业务接口的时候再assert一把。看下面的代码。
##### 方式1
{% highlight java %}
public Map queryWokrfResult(String recnum){
	Assert.isNotNull(recnum,  "业务流水号不能为空!");
	// query with recnum
}
{% endhighlight %}

##### 方式2
{% highlight java %}
public Map queryWokrfResult(String recnum){
	if(StringUtils.isEmpty(recnum)){
		throw new ReceptionException("业务流水号不能为空!");
	}
	// query with recnum
}
{% endhighlight %}

我是这么看待的，方式1表示你可以传空，但我会做检测，不会让你得逞。相当于防御性编程。
方式2，你就不应该传空，你的参数不符合接口规范。
虽然assert的实现或许就是判断的简单封装，但这里这么不能简单理解。
__assert与其说是编程技巧，不如说是编程模式。__

对于java来说，本身是支持assert的，但这个特性一般没人使用。大多数人选择用异常来封装，例如[spring assert][3],[commons-lang validate][5]等都提供了自己的一套assert api。需要注意的时候，assert里边的条件应该都是非常快的，不要把业务逻辑混淆进去。

### 封装,封装,由底层处理
实际上，提供一些工具类(如常用的判空操作和带默认值操作)可以减轻痛苦，
还可以根据具体应用在底层框架上处理，在提高容错性的同时，让代码更优雅。

在项目的老代码中，我们会看到这样的判空处理。
{% highlight java %}
// example 1
MapVo vo = new MapVo();
if(StringUtils.isNotEmpty(name)){
	vo.setAttribute("name", name);
}
if(StringUtils.isNotEmpty(subsid)){
	vo.setAttribute("subsid", subsid);
}
// ... other conditions

DBUtils.queryByVo("queryXXX", vo);
{% endhighlight %}

为什么这么写，是因为框架的sql生成配置只提供了null的配置方式，而不支持常见的空字符串，无内容的字符串等常见模式。
为此，通过增加特性，像ibatis那样支持isEmpty模式的配置，就可以省略大量显式判断。
修改后的代码如下：
{% highlight java %}
// example 2
MapVo vo = new MapVo();
vo.setAttribute("name", name);
vo.setAttribute("subsid", subsid);
// ... other conditions

DBUtils.queryByVo("queryXXX", vo);
{% endhighlight %}

再看，例如老页面jsp里边这样的逻辑也很多(嵌入java代码)
{% highlight java %}
<% Subscriber subs = request.getAttribute("subscriber"); %>
<% if(subs != null) {
     out.print("<span>" + subs.getServnumber() + "</span>");
<% } %>
{% endhighlight %}
对于这种代码有很好的方式，如el表达式，又或者velocity等模板解决方案，都具有很好的容错性。

再来看看其他语言框架的解决模式，如javascript框架jquery，对错误保护就很好，对不存在的东西，提供了默认行为: 视而不见。
通过基础框架提供的保障，避免一些琐碎的工作，代码可以更关注于逻辑处理。

### 空对象模式
[空对象模式][1]就是用一个特殊的对象代替空对象，这个对象和正常对象有着同样的接口，但调用的时候会表现出异常处理逻辑。[wiki上有关于这个模式的各种编程语言的描述][2]。
通过这种模式，把异常处理逻辑封装起来，对于客户端代码来说，调用方式和正常情况是一样的。

例如，对于数据查询接口返回空的列表而不是null，使用一些空集合对象。
{% highlight java %}
java.util.Collections#emptyList()
java.util.Collections#emptyMap()
java.util.Collections#emptySet()
{% endhighlight %}
对于返回列表的接口，使用空列表对象，很多判空的逻辑就没有必要存在了。
又例如，返回不可修改或线程安全的特殊列表，也可以看成是这种模式的变化
{% highlight java %}
Arrays#asList()

java.util.Collections#unmodifiableList()
java.util.Collections#unmodifiableMap()
java.util.Collections#unmodifiableSet()

java.util.Collections#synchronizedList()
java.util.Collections#synchronizedMap()
java.util.Collections#synchronizedSet()
{% endhighlight %}

### 总结
__对接口进行约定，约束参数和返回值，可以简化判空的处理。__

__在框架层面进行统一封装，可以专注于业务逻辑。__

 [1]: http://www.cs.oberlin.edu/~jwalker/nullObjPattern/
 [2]: http://en.wikipedia.org/wiki/Null_Object_pattern
 [3]: http://static.springsource.org/spring/docs/1.2.x/api/org/springframework/util/Assert.html
 [4]: http://code.google.com/p/guava-libraries/wiki/UsingAndAvoidingNullExplained
 [5]: http://commons.apache.org/lang/api/org/apache/commons/lang3/Validate.html

{% assign series_list = "improve bitter code" %}
{% include series_list.html %}
