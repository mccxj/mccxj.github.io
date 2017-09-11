---
layout: post
comments: true
title: "improve bitter code: '不可避免'的重复"
description: "关于编码规范，代码评审，改进质量方面的总结"
categories: ["质量改进", "improve bitter code", "可读性", "算法", "重复"]
---

__备注: 示例中的代码并不是真实代码的完全拷贝__

### 一段分批处理的逻辑
周六做code diff的时候发现B项目一段颇长的处理逻辑(40行左右)。处理流程是这样的，
从页面上取到一批数据之后，用这批数据封装参数进行后台调用(远程调用)。为了避免数据调用超时，
对这批数据进行分批多次调用。代码如下所示(现实的代码比这个要复杂些，并且没有使用subList方法)：
{% highlight java %}
List batchdatas = ...;
int batchsize = 10; // load from parameter

int datasize = batchdatas.size();
if(datasize > batchsize){
    int batch = datasize / batchsize;
    for(int i=0;i<batch;i++){
        List inparams = new ArrayList();
        List batchdata = batchdatas.subList(batchsize*i, batchsize*(i+1));
        inparams.add(new CEntityList(batchdata));
        RemoteCall.commonInvoke(operator, inparams, "CheckCmd");
    }

    int left = datasize % batchsize;
    if(left != 0){
        List inparams = new ArrayList();
        List batchdata = batchdatas.subList(batchsize*batch, datasize);
        inparams.add(new CEntityList(batchdata));
        RemoteCall.commonInvoke(operator, inparams, "CheckCmd");
    }
}
else {
    List inparams = new ArrayList();
    inparams.add(new CEntityList(batchdatas));
    RemoteCall.commonInvoke(operator, inparams, "CheckCmd");
}
{% endhighlight %}

从代码的实现看，思路还是比较清晰的。如果不足一次，就一次提交。否则计算出一共要分多少次，
然后逐次处理提交，最后还要判断是否还有剩余的，如果有就再处理一次。

代码显现出来的问题也比较明显，就是远程调用的逻辑出现了重复。

### 算法小调整，避免重复
有没什么办法可以避免重复? 或许有些童鞋第一反应是给这几句代码抽取成小方法。
不过这里有更好的办法，首先细想一下就会发现不足一次的判断(datasize > batchsize)不是必要的，
如果计算批次而是计算每次的起始点和结束点，上面两个分支也可以合并一下。调整后代码如下：
{% highlight java %}
List batchdatas = ...;
int batchsize = 10; // load from parameter

int datasize = batchdatas.size();
for(int startidx=0;startidx<datasize;startidx+=batchsize){
    int endidx = (startidx+batchsize) > datasize ? datasize : (startidx+batchsize);

    List inparams = new ArrayList();
    List batchdata = batchdatas.subList(startidx, endidx);
    inparams.add(new CEntityList(batchdata));
    RemoteCall.commonInvoke(operator, inparams, "CheckCmd");
}
{% endhighlight %}

其中计算endidx使用了一个三元表达式，使用三元表达式用来替代一些简单的if-else语句是个实用的小技巧。
代码量缩小为原来的三分之一，代码少了，维护量也轻松了。

类似这样的代码也并不少见，例如计算总页数的分页逻辑有下面的写法
{% highlight java %}
// 常用做法
int totalpage = totalsize / pagesize;
if(totalsize % pagesize != 0){
    totalpage++;
}

// 另外一种写法
int totalpage = (totalsize + pagesize - 1) / pagesize;
{% endhighlight %}

### 基本功很重要
java是一门比较古板的语言，大多数情况下，写出来的代码也是大同小异的。
同时，java相关框架又特别的多，很容易拣了芝麻丢了西瓜。
以来面试的童鞋为例，连基本算法的时间复杂度都没弄清楚的人不在少数，所以
在项目代码中，经常看到化简为繁的代码，现在也很习惯了。

{% assign series_list = "improve bitter code" %}
{% include series_list.html %}
