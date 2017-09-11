---
layout: post
comments: true
title: "结合状态机的开发风格"
description: "结合状态机的开发风格"
categories: ["状态机", "statechart", "angular"]
---

本文主要以XXX的html5版本为蓝本，讨论结合状态机开发的思路和实践方式。状态机选型使用[statechart.js](https://github.com/burrows/statechart.js)。

### 起步知识

* 状态机介绍，请参考[Statecharts and Angular.js](http://burrows.github.io/statechart-angular-pres)
* statechart.js的基本使用方法，请参考[statechart.js](https://github.com/burrows/statechart.js)

特别是状态机介绍，内容非常好，强烈推荐。

### 适用场景

* 主要用于某个具体业务的复杂页面流控制
* 简单的业务流程是不需要的。例如只有一两个页面(列表+详情)
* 适用于多步骤多页面(包括弹出框)、各种跳转的场景

### 如何定义状态?

根据页面流、步骤来定义状态。可以参考以下步骤：

* 对照保真、流程图，划分每个独立页面
以个人营销活动为例，主要页面包括活动页面、选档次页面、奖品页面、奖品包选择页面、缴费页面、发票页面。
那么可以考虑定义为list、level、reward、giftpack、charge、invoice

* 对于有多种弹出窗口的情况，可以考虑定义子状态
以推荐业务为例，在菜单页面上，可能会弹出反馈窗口，或者产品订购窗口。
那么可以考虑定义为menu/index、menu/feedback、menu/prod，这样的话，通过下面的页面控制，就可以让在值状态的情况下，菜单页面一直显示。

```html
<div ng-show="fsm.isCurrent('/menu')" ng-include="'app/partials/recommended/recommended_menu.html'"></div>
<div ng-show="fsm.isCurrent('/menu/prod')" ng-include="'app/partials/recommended/recommended_orderprod.html'"></div>
<div ng-show="fsm.isCurrent('/menu/feedback')" ng-include="'app/partials/recommended/recommended_feedback.html'"></div>
```

* 对于页面显示，有较多共性的页面，可以考虑定义子状态，方便共享逻辑和事件处理
以上述的个人营销活动为例，奖品页面、奖品包选择页面的页面很类似，功能操作也比较实现，可以定义成子状态，如order/reward、order/giftpack

```html
<div ng-show="fsm.isCurrent('/list')" ng-include="'app/partials/personalMarketCamp/personalMarketCamp_list.html'"></div>
<div ng-show="fsm.isCurrent('/level')" ng-include="'app/partials/personalMarketCamp/personalMarketCamp_level.html'"></div>
<div ng-show="fsm.isCurrent('/order/reward')" ng-include="'app/partials/personalMarketCamp/personalMarketCamp_reward.html'"></div>
<div ng-show="fsm.isCurrent('/order/giftpack')" ng-include="'app/partials/personalMarketCamp/personalMarketCamp_giftpack.html'"></div>
<div ng-show="fsm.isCurrent('/invoice')" ng-include="'app/partials/personalMarketCamp/personalMarketCamp_invoice.html'"></div>
<div ng-show="fsm.isCurrent('/charge')" ng-include="'app/partials/personalMarketCamp/personalMarketCamp_charge.html'"></div>
```

### 如何控制页面的显示、如何响应页面操作?

* 页面显示与否，通过状态机的状态，而不是数据的状态。这里用的是isCurrent方法
* 页面操作，通过状态机的事件发送，而不是直接使用绑定在$scope的方法。这里用的是send方法
* 页面跳转，通过状态机的状态变化来驱动。这里用的是goto方法，是在send方法之后的event逻辑中处理的。

页面显示与否，例子上面已经说了。而对于ng-click这种事件触发，直接用send方法即可。

```html
<div class="Feedback-btn">
  <a ng-click="fsm.send('feedback', 'hesitate')" class="accept-btn"><span>考虑</span></a>
  <a ng-click="fsm.send('feedback', 'refuse')" class="refuse-btn"><span>拒绝</span></a>
</div>
```

可以看到，事件也可以捎带参数的，这样可以在该状态的event中进行处理，如下：

```coffeescript
# 反馈
@state 'feedback', ->
  # 进行反馈操作
  @event 'feedback', (operationtype) ->
    product = $scope.viewModel.product
    if product.opertype == '1'
      new Toast(
        context: $('body')
        message: "该产品不可推荐"
      ).show();
      return

    qryfeedbackService.event
      "userseq": $scope.viewModel.product.userseq
      "servnumber": $scope.telnum
      "operationtype": operationtype
    .then (ok) =>
        @goto '/menu/index'
      , (err) ->
        new Toast(
          context: $('body')
          message: err
        ).show()
```

需要注意的是，event是挂靠在某个状态下的，如果你是子状态的话的，event会先在子状态中找，如果没有找到会在父状态上找。
通过这种方式，就可以实现多个子状态共享event，例如奖品页面、奖品包页面都有选择功能，就可以把这个操作放到父状态的event中去。

### 更多状态机的细节

很多状态机都实现了某些特殊状态，如进入状态，退出状态这种事件。statechart也实现了，对应的是enter和exit，代码大体上是:

```coffeescript
@state 'menu', ->
  @enter ->
    #TODO

  @exit ->
    #TODO
```

但是需要注意的是，重复进入这个状态的话，是会重复执行的。所以对于A -> B -> C这样的业务流程，从A到B和C回退到B，都会执行这个enter，
就无法区分这种情况了。因为通常，从A到B是进行初始化，而从C回到B得保留原来B的数据状态。所以实际上我很少使用这些特殊事件，除非：

* 无需区分的情况，这样写会让代码风格更统一。
* 没有接口交互，本地操作的话，因为这种消耗小很多。
* 存在直接跳转到该状态的情况(例如由另外的业务跳转过来)，这种特殊情况下前面的步骤都被忽略。而且这种情况下，需要通常需要接口交互(例如补充某些必要信息)，而为了区分回退的情况，我通常会根据业务特性考虑一些数据缓存处理。