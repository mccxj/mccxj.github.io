---
layout: post
comments: true
title: "hello,coffeescript"
description: "coffee script入门"
categories: ["资料", "javascript", "coffeescript"]
---

周五下午的时候，花了大半个小时在团队里边介绍了CoffeeScript，使用的是[Sam Stephenson的ppt][1]。
这里再把相关的内容总结一下。

## Why CoffeeScript
[CoffeeScript][2]关注的是Good Part Of Javascript,可以和javascript无缝结合。
CoffeeScript借鉴了ruby的语法，参考python的缩进规范，让代码更为简洁，规范性更强，适合以javascript为主的应用,如Node应用。
*以我的经历(弄个html5 canvas+node的应用)，语法入门很简单(如果熟悉ruby，简直就似曾相识)，代码量可以少30%~50%，但对像Extjs这样的巨无霸UI代码，由于嵌套太深，不是很合适。*

## JavaScript’s Good Parts
**It’s private by default. **  
所有编译后的代码都会在一个自执行的闭包中，所以不会渗漏出全局变量。还记得在js文件中定义的变量在外边还能访问到吧? 在coffeescript中，全局变量必须要显示指出。

**No more “var” keyword.**  
在js中变量只有在全局或函数内有效，没有局部变量这种，而且没有声明var的话，就会变成全局的。在coffeescript中遵循了这套规则，给所有变量加上var，并在全局开头或方法开始全部声明。
这样可以减少一些粗心导致的bug。

**Strict comparisons.**  
严格相等有了专门的语法is。对于javascript来说，相等这东西让不少初学者踩地雷，特别是对false这东西的理解。

## 有趣的语法
对着ppt或者coffeescript的官方指南看看就行了。我挑一些自己特别喜欢的：

* Significant white space 空白是有意义的，纠结于编码格式规范真是扯淡的事，这下好了不用争了
* Comprehensions 这是非常有用的语法糖
* Classes and Inheritance 内置的类和继承，不用纠结与用什么方式实现类比较好
* Bound functions 解决this应用的内置方案
* Conditionals 写过ruby的童鞋就知道了，后置的if和unless语句爽呆了，对可能为undefined和null的对象调用方法也优雅许多了
* String Syntax 多行字符串，可内嵌参数的字符串，拼接字符串的时代过去了

## How to Use
* CoffeeScript.org有一个[实时编译成js的工具][4](页面)
* Rails3.1已经支持CoffeeScript
* 使用[js版的编译器][3],并使用type为text/coffeescript的script标签，嵌入CoffeeScript代码或文件
* 使用Node.js,可以用npm install -g coffee-script,然后使用coffee命令编译成js文件
* 可以使用监控工具来实时编译js文件，例如使用watchr

{% highlight bash %}
gem install watchr
watchr project.watchr

# project.watchr
watch('src\/.*\.coffee') {|match| system "coffee --compile --output js/ src/"}
{% endhighlight %}

 [1]: http://sstephenson.s3.amazonaws.com/presentations/fowa-2011-coffeescript.pdf
 [2]: http://coffeescript.org/
 [3]: http://coffeescript.org/extras/coffee-script.js
 [4]: http://coffeescript.org/#try:1
