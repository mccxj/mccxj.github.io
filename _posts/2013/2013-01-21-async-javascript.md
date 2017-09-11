---
layout: post
comments: true
title: "Async Javascript读书笔记"
description: "读书笔记"
categories: ["读书笔记", "异步", "javascript", "笔记"]
img: "/images/2013/async-javascript.png"
---

![Async Javascript][1]

这篇读书笔记的主要内容是在家里用ipad上输入的，感觉输入还是比较吃力呀。

本书的副标题是Build More Responsive Apps with Less Code，
关键词是异步、响应式，主要是对如何书写异步js代码提供一些指导意见，描述js的异步特性，回调代码的书写，还介绍了PubSub、Promises等通用模式。
最后介绍了worker多线程模型，js异步加载等内容。书的页数可谓少得可怜，内容还算比较实在，
提供了大量的开源库实现供读者参考，对于框架选型的话倒是可以参考一下。

## js事件
主要是了解事件背后的本质。提供了象setTimeout document.onready都是事件的典型案例。
因为js基本都是单线程模型，所以事件的触发和代码的执行有时候并不一致，而是有些时间差。即使是setTimeout(fn,0)这样的代码。
这样的逻辑也不是立即执行的。它的执行模型大概是这样的(在ipad上手绘的)。
![js执行模型][2]

任何回调方法的执行都是需要按队列顺序来执行。如果当前代码没有执行完,就不会执行其他回调方法。
因此理解js的异步处理模型非常重要，**代码应该尽快执行完毕，避免阻塞其他代码的执行**。

## 哪些是异步逻辑
主要是一些io操作。例如

1. ajax调用
2. webkit里边的console.log
3. dom操作，浏览器可能会延迟的效果出现
4. Node服务端里边比比皆是

当然还有些是时间片操作。例如

1. setTimeout或setInterval
2. node的[process.nextTick][7]
3. html5提供的requestAnimationFrame等特性

## 异步逻辑注意
如果使用回调方法进行异步处理，就不要在后面的代码中按同步的逻辑来考虑。  
异步回调方法中用到的对象也应该先定义，避免由于缓存等造成奇怪的问题。  
可以使用web worker api或异步调用来生成缓存数据。  
避免超过两层的回调方法。   

## 处理错误
使用异步调用时，stack trace可能会不全  
使用异步调用时，try-catch可能会失效  
使用err来作为回调方法的参数，或者区分success和error的回调方法  
处理uncaught异常，可以使用:
{% highlight javascript %}
# Broswer
window.onerror = function(err){
}

# Node 
process.on('uncaughtException', function(err){
})
{% endhighlight %}

## 流程控制
回调方法太多，流程控制变得异常困难，很容易造成错误，应该避免超过两层的回调方法。
比较常用的办法有PubSub、Promises等解决方案/模式。

### PubSub
就是publish/subscribe,发布订阅，常见的监听器模式。通常是把回调方法组织为带名字的事件，模拟事件触发。例如

1. Node的[EventEmitter][4]
2. Backbone的[Event][5]模型
3. jQuery的[自定义事件][6]

需要注意的是，**PubSub模式的逻辑不一定是异步的**。
如果trigger是同步逻辑，应该注意避免在回调方法中再次trigger某个事件而造成死循环，
例如jQuery就可能出现这个问题，而像[backbone][14]在值没有变化时不再触发change事件，也提供了slient的选项。而[Ember][13]之类是采用setTimeout的方式
加入队列进行处理的方式，就是采用了异步的方式。


### Promises
书中主要使用了jquery做例子，显示了[deferred][15]的用法，对callback的用法有个比较。
熟悉jquery的童鞋通过Deferred的用法就有个大体的印象了。我个人的感觉就是，Promises模型是对PubSub常用功能的抽象，
有点规范的意思，事实上也的确有commonjs的[Promises/A][18]规范。

介绍了promises和deferred的区别，deferred是promises一个子集，它不能自己控制状态变更，需要有由其他事件出触发。
对于node来说，现在是采用回调方法的方式，要采用deferred方式，主要做些改变。例如：
{% highlight javascript %}
deferredCallback = function(deferred) {
  return function(err) {
    if (err) {
      deferred.reject(err);
    } else {
      deferred.resolve(Array.prototype.slice.call(arguments, 1));
    };
  };
}

var fileReading = new $.Deferred();
fs.readFile(filename, 'utf8', deferredCallback(fileReading));
{% endhighlight %}

## Async.js
介绍了async.js的用法，用来解决异步处理中的流程控制问题，例如iterator的问题。包括串行和并行的控制。
这个库还提供了更加复杂的并发性控制。具体的用法还是看[官方的介绍][3]吧。还提供了另外一个库[step][16]的介绍

## Worker
提供[worker][8]的功能，在浏览器上worker还是有一些受限制的地方，例如不能涉及DOM。
而在node上，可以用[cluster][9]来支持类似worker的功能。

## 异步加载js
主要探讨了几种方案，如在head加载，缺点是页面显示太慢。在body末尾加载，问题是还是要等js结束还能让事件生效，并且同样需要顺序加载。
解决方案有[defer][10]/[async][11]和ajax加载等方案。

对于比较现代的浏览器，支持defer关键字用于异步js加载，还有async关键字，区别是后者不会顺序加载，先加载完先处理。
async在有些插件化的js时async可能可以用到，其他就很难用到，因为很多js都是存在依赖的关系。

当然，还有一些介绍了自定义的异步加载方案，如html5对象有onready事件,还有ajax加载方式。
不过业界已经有些比较成熟的异步加载库了，如[yepnope][17]

最后还介绍了一些异步加载的其他方案，例如增强的语法来支持异步逻辑。这种预编译的方式，我感觉不如在coffeescript上增加关键字的预处理方式，
类似coffeescript的class关键字。另外还介绍[js1.7][12]关于异步逻辑的新特性:Generator。

 [1]: /assets/images/2013/async-javascript.jpg
 [2]: /assets/images/2013/runjs.jpg
 [3]: https://github.com/caolan/async
 [4]: http://nodejs.org/api/events.html
 [5]: http://backbonejs.org/#Events
 [6]: http://api.jquery.com/on/
 [7]: http://nodejs.org/docs/latest/api/process.html
 [8]: http://www.w3.org/TR/workers/
 [9]: http://nodejs.org/api/cluster.html
 [10]: http://www.w3schools.com/tags/att_script_defer.asp
 [11]: http://www.w3schools.com/tags/att_script_async.asp
 [12]: https://developer.mozilla.org/en-US/docs/JavaScript/New_in_JavaScript/1.7?redirectlocale=en-US&redirectslug=New_in_JavaScript_1.7
 [13]: http://emberjs.com/
 [14]: http://backbonejs.org/
 [15]: http://api.jquery.com/jQuery.Deferred/
 [16]: https://github.com/creationix/step
 [17]: http://yepnopejs.com/
 [18]: http://wiki.commonjs.org/wiki/Promises/A