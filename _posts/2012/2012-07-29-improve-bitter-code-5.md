---
layout: post
comments: true
title: "improve bitter code: 拘泥于单出口方法"
description: "关于编码规范，代码评审，改进质量方面的总结"
categories: ["质量改进", "improve bitter code", "可读性", "单出口", "单赋值"]
---

__备注: 示例中的代码并不是真实代码的完全拷贝__

### 究竟是哪个日期
前阵子代码评审的时候发现一段代码，逻辑是用于查找一个最大日期，代码逻辑大约是这样的：
如果没有相关的日期记录，则返回当前日期，否则，当日期值为空时，是业务限制异常，并且配置的
相关日期小于当前日期的话，还是应该选择当前日期。代码如下：
{% highlight java %}
public String getPrivMaxDate(){
    String currDate = DateUtil.parse(new Date());
    String maxDate = currDate;
    
    List<String> privMaxDates = ...; //load data from database
    if(Objects.isNotEmpty(privMaxDates)){
        maxDate = privMaxDates.get(0);
        if(StringUtils.isEmpty(maxDate)){
            throw new BusinessException("...");
        }
        else if(maxDate.compareTo(currDate) < 0)
            maxDate = currDate;
        }
    }
    return maxDate;
}
{% endhighlight %}

大家应该被我的描述搞晕，我也觉得说得很乱。从代码看，maxDate有好几处地方可以修改，有时候是当前日期，有时候是配置的日期，的确比较乱。
接下来我们尝试对代码进行一些简单的修改，看看效果。

### 移动代码，快速逃离
接下来，我们要做一些小的调整:

1. 反转条件，让小逻辑先行，如Objects.isNotEmpty(privMaxDates)的判断
2. 避免修改变量，让代码简单化，如直接使用currDate
3. throw本身属于返回值的一种，所以在它之后的代码可以简化，例如代码中的else if

{% highlight java %}
public String getPrivMaxDate(){
    String currDate = DateUtil.parse(new Date());
    
    List<String> privMaxDates = ...; //load data from database
    if(Objects.isEmpty(privMaxDates)){
        return currDate;
    }

    String maxDate = privMaxDates.get(0);
    if(StringUtils.isEmpty(maxDate)){
        throw new BusinessException("...");
    }

    if(maxDate.compareTo(currDate) < 0)
        return currDate;
    }
    return maxDate;
}
{% endhighlight %}

经过改造，由于maxDate只赋值一次，代码变得好理解了。

### 灵活的结构调整
借助eclipse提供的重构功能，可以对代码进行调整。对于我们常见的if语句，可以ctrl+1就有很多神奇的提示。
例如if反转，添加else，切换成三元表达式等手法。

#####对于if语句里边有很大块的代码，可以考虑使用反转，让另外一个分支先处理。
例如上面的例子。另外，对有些只有if没有else的代码块，在操作手法上可以先用ctrl+1添加else，再进行反转操作。

#####对于很小的if-else语句，可以考虑转换成三元表达式。如下面代码示例
{% highlight java %}
String forward = null;
if(success){
    forward = "success";
}
else{
    forward = "fail";
}

// or this way
String forward = success ? "success" : "fail";
{% endhighlight %}
#####对于存在多次赋值的情况，如果发现已经是最终的返回值，在调整时可以使用return，理清逻辑。
例如上面的例子。同时，经过分解也便于对复杂的代码进行封装抽取更小的方法。

后话：单赋值还有其他一些好处，例如便于调试定位，
在eralng中所有的变量都是单赋值的，没接触过是很难想象是怎样的一种场景。有兴趣可以去了解一下。

{% assign series_list = "improve bitter code" %}
{% include series_list.html %}
