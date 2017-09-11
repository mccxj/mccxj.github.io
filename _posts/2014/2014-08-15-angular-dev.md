---
layout: post
comments: true
title: "angular开发总结"
description: "angular开发总结"
categories: ["angular", "MVVM"]
---

下面是XXX的HTML5版本开发总结。

### 开发思路

说好是MVVM了，当然和以前也页面为中心进行DOM操作的方式很不一样。这方面可以参考文章：[StackOverFlow精彩问答赏析：有jQuery背景的开发者如何建立起AngularJS的思维模式？](http://www.infoq.com/cn/news/2013/11/how-to-think-angularjs)

开发经验就是，以数据为中心来考虑。我通常是对着高保真，先把所需要的数据结构先构思了，看看接口是如何和数据进行对接的。

不过，即使是以数据为中心的开发模式，逻辑与视图的完全分开几乎是不可能完成的任务，只能尽量的分离。我们需要操作的数据模型，不是真正意义的业务模型，而是视图模型(业务数据和视图数据的混合体)。如果实现上需要更偏重业务模型的话，可能要更多借助于resource的概念(ng也有提供这个)，不过对于我们这种过程化的接口，意义不大。

### 关于Service

因为接口的原因，现在有的Service文件，按规范是一个接口对应一个，而且文件内容除了接口参数和响应结果获取，其他基本上是一样的。

根据接口数据的格式，我有时候会预先在service里边进行格式调整，例如一个带父子关系的列表数据，需要先整理为两层的数据结构，这个我会优先考虑在Service里处理。基本原则就是，如何让数据更贴近业务模型。

### 关于Controller

因为Service是一个很薄的一层，所以大多的页面流程控制，都是在Controller里边完成的。对于Controller如何组织视图模型，我通常是定义一个总的viewModel，来挂载所有的视图模型，大体上就是:

```coffeescript
# 定义视图的数据模型
$scope.viewModel =
  acttypes: null #活动类型
  acts: {} #可选的活动列表数据
  act: null #选择的活动
  levels: [] #可选的档次
  level: null #选择的档次
  rewards: [] #可选的赠品、赠品包
```

这个做法，有个最明显的好处，就是不容易踩到ng-model的scope的坑，关于angular的scope,请参考文章 [Understanding Scopes ](https://github.com/angular/angular.js/wiki/Understanding-Scopes)

### 关于ngmodel

像上面提到的，使用ngmodel,nginit这种会操作model的指令，需要注意scope的范围，一不小心就可能操作到不同的scope。具体原因请参考上述文章。有几个办法可以处理：
* 使用$parent,但是对于多层次的scope无效。
* 采用非原型类型的obj，可以避免scope的意外。如上述的viewModel

大多数情况在我们都是用的单向绑定，如果涉及ng-model这种双向绑定的话，有另外一个文章推荐 [Using NgModelController With Custom Directives](http://www.chroder.com/2014/02/01/using-ngmodelcontroller-with-custom-directives/), 因为这篇文章很好的描述了angular对双向绑定的处理流程。

另外，如果使用html5的验证或新类型，也会有些问题，例如你使用number类型，输入非number类型的内容的话，model获取到的值就无效了。

### 关于性能

可以参考一下这位UC童鞋的文章 [angular性能优化心得](http://atian25.github.io/2014/05/09/angular-performace/)

基本上该说的都在上面的了。这里补充一下：

* bindonce可以减少一些无谓的watch，主要用于ng-repeat上，目前还没使用上，但试过是可用的。
* 在watch方法里边不能操作dom，并及时unwatch。操作dom对性能影响很大。
* 像$interval，$timeout虽然有个invokeApply这个参数，却是无效的，这个问题在1.3.0beta14才修复，如果不需要变更模型，建议用原生的。
* filter因为经常计算，所以要尽量的快，选择合适的数据结构很重要，例如对大列表进行筛选不如key/value的查找快。曾经做过字典结构的调整。
* 目前项目没有lazy loading的问题，这迟早是需要考虑的，网上已有一些方案，借助了route的resolve特性来实现的。

### 复杂页面流程

* 是分单个route还是多个route，主要考虑是否需要独立入口，因为一个route对应一个url。这个主要参考URL的设计原则。
* 跨页面参数传递，如果参数少建议用路径参数，如果共享数据多，可以使用服务来数据共享。

复杂页面的处理，我主要考虑使用状态机来处理。现在使用[statechart.js](https://github.com/burrows/statechart.js)

关于状态机的介绍，请参考[Statecharts and Angular.js](http://burrows.github.io/statechart-angular-pres)

### 开发工具支持

项目涉及多种工具、概念，需要掌握他们的应用场景：

* 基础设施 node
* 包管理 npm
* 模块化，require，sea,amd,cmd
* 资源依赖，bower
* 打包自动化，grunt
* angular相关 插件batarang
* 语言改进，coffeescript

特别说明关于coffeescript的使用，上述关于viewModel的代码就是coffeescript，大体上可以减少1/3的代码行。

### 调试的支持

chrome的调试工具很强大，能够模拟手机操作，基本可以解决大多数问题。

不过真机、webview之类的调试仍然是个大问题。当然方案也是有的，如google官方的android+chrome方案、weinre的方案。
不过，我页面做的少，实在没什么经验可以发表的。

### 其他

自己对css并不熟悉，主要关注逻辑编写，个人认为和web打交道，还是得多理解http协议，浏览器原理。