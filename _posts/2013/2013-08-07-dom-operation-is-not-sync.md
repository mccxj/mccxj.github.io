---
layout: post
comments: true
title: "Dom操作和渲染并不是同步的"
categories: ["浏览器", "ie", "dom", "javascript", "render"]
---

### 一个在ie6不兼容的线上问题

昨天，有同事找我看一个线上问题，说一个在ie8测试通过的功能，在ie6上不能正常使用。
一开始，他用alert定位到是哪部分代码出问题，并认为后面有部分代码没有执行。奇怪的是，在这中间加上alert语句的话，
在ie6上也是能够正常运行的。

这部分代码简化后大概是这样的:

```html
<select onclick="javascript:selectbank(this);">
<option value="">--all--</option>
</select>
```

```javascript
function(obj){
  // get val and desc from a pop up page
  var opt = "<option value='" + val + "'>" + desc + "</option>";
  $(obj).append(opt);
  $(obj).val(val);
  // alert("test");
  // ...
}
```

我发现没有加alert的时候，页面会报js错误。并且，这种情况在ie8不会出现。

### Dom操作和渲染并不是同步的

为什么会出现这种情况，这主要因为Dom操作比普通js操作的消耗大很多，因为它需要改变页面元素，
导致页面出现重新渲染的情况，这取决于浏览器的实现。像ie8等比较高级的浏览器，渲染、js速度都要快许多，
所以不会出现这种情况。而ie6太慢了，很可能还没反应过来。

类似的情况是，在一个js调用过程中(某次事件)，如果你频繁修改dom，浏览器很可能不会逐个生效，
而是几个操作一起生效，跟期望的有点不一样。在一些低端浏览器，可以做的优化是，让调用过程尽量快，
减少Dom操作，或者中间使用setTimeout休息一下。

而像上面的问题，修改成下面的代码就可以了:

```javascript
function(obj){
  // get val and desc from a pop up page
  var opt = "<option selected value='" + val + "'>" + desc + "</option>";
  $(obj).append(opt);
  // alert("test");
  // ...
}
```
