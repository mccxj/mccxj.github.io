---
layout: post
comments: true
title: "improve bitter code: 更友好的链式写法"
description: "关于编码规范，代码评审，改进质量方面的总结"
categories: ["质量改进", "improve bitter code", "编码规范", "参数", "可读性", "接口"]
---

__备注: 示例中的代码并不是真实代码的完全拷贝__

### 无意出现的链式代码
还是前几天的事情，群里边有同事提到一个账单相关的需求，其中涉及到组装打印用数据。
有个新同事在Service里边写了一段这样的代码：
{% highlight java %}
public XXService fillItem(String content, String tag) {
    // some codes
    this.printitems.add(new Item(tag, content));
    return this;
}
{% endhighlight %}

有同学不是特别理解为什么要这么写，甚至发出return this是什么对象的疑问。

这段代码原来是这么写的:
{% highlight java %}
public void fillItem(String content, String tag, List printitems) {
    // some codes
    printitems.add(new Item(tag, content));
}
{% endhighlight %}

有同学奇怪，为什么再提交一次printitems就会重新加一遍，认为第一段代码的问题出现在return this上面。
这样要说明一下，我们使用的是struts1，所以如果Service实例作为action的实例变量，那也是只有一个对象来的。
所以每次刷新一次重复加一遍就没什么奇怪的了。

回头来看那段新写的代码，出发点是好的。毕竟采用这种方式可以使用链式写法（如下所示），代码有时候会变得很sexy。
{% highlight java %}
XXService.fillItem("a", "A").fillItem("b", "B");
{% endhighlight %}
这段代码的问题不在于是否采用链式写法，而是对这种单例，变量生命周期了解不足造成的。
链式写法在代码中很少会使用到，但在设计api，也是可以考虑的。设计的好，可以有效提高代码的编写效率和api的友好型。
例如，我就对java collection api里边的add方法深恶痛绝,就是不能连续add，要add多少个就要写多少行，还真的挺烦的。

### 采用链式写法的代码
在很多有名的开源框架中，链式写法也不是很少见，下面举几个例子:

很常见的有jquery
{% highlight javascript %}
$("#name").attr("readonly", true).val("test");
{% endhighlight %}

还有mock框架mockito
{% highlight java %}
when(mockedList.get(0)).thenReturn("first");
when(mockedList.get(1)).thenThrow(new RuntimeException());
{% endhighlight %}

再看看rails的写法
{% highlight ruby %}
Post.where('id > 10').limit(20).order('id desc').only(:order, :where)
Client.limit(5).offset(30)
{% endhighlight %}

### 我的看法
那么，如果你想采用链式写法，有什么地方需要注意呢?
1. 先写一下客户端代码，看调用方式合理么，符合sexy api么?
2. 采用链式操作的interface相关性比较强，经常一起出现。
3. 参数parameter一般较少，因为参数多了，代码很容易变得模糊不清。
4. 链式操作要么容错强(像jquery)，要么就得直接抛出异常，对返回值不关心。

{% assign series_list = "improve bitter code" %}
{% include series_list.html %}
