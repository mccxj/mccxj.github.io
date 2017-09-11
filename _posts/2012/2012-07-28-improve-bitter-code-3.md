---
layout: post
comments: true
title: "improve bitter code: 看不懂的正则表达式"
description: "关于编码规范，代码评审，改进质量方面的总结"
categories: ["质量改进", "improve bitter code", "可读性", "正则表达式", "封装"]
---

__备注: 示例中的代码并不是真实代码的完全拷贝__

### 偶然的发现
今天好奇浏览了一下N项目的代码变更历史，发现有人提交了一段关于校验文件格式的代码。
其中包括一段校验日期格式的java代码。代码是这样的：
{% highlight java %}
String validDateStr = // read from file lines
String regex = "(([0-9]{3}[1-9]|[0-9]{2}[1-9][0-9]{1}|[0-9]{1}[1-9][0-9]{2}|[1-9][0-9]{3})(((0[13578]|1[02])(0[1-9]|[12][0-9]|3[01]))|((0[469]|11)(0[1-9]|[12][0-9]|30))|(02(0[1-9]|[1][0-9]|2[0-8]))))|((([0-9]{2})(0[48]|[2468][048]|[13579][26])|((0[48]|[2468][048]|[3579][26])00))0229)";
if(validDateStr.matches(regex)){
    // do something
}
{% endhighlight %}

看到这个正则表达式，我立马纠结了，这个正则表达式不知是什么意思。
虽然前几天写代码的同事来问过怎么写校验日期的正则，我当时比较忙，叫他找找有没有现成的。
这次看到这个正则，还是被雷了一把。

于是我问了一下，原来这个正则是校验日期格式，不过加了闰年的判断，所以变得相当复杂。
我晚上还去搜了一下，大概是出自[这里][1]的吧！不同的是文中判断的是YYYY-MM-DD的格式，而同事的代码
是判断YYYYMMDD的格式，显得更为难懂。

### 保持代码的可读性、可维护性
对于这种拿来的复杂代码，的确很cool，不过即使今天你看懂了，别人不一定看得懂，也难保过些日子自己也看不懂了。

所以通常需要一些保持代码可读性、可维护性的手段：
1. 加多几段注释,或者把来源url标注一下，就像有人喜欢标注那个需求一样。
2. 把正则弄成常量，并把验证方法封装起来，只需调用method就可以了。
3. 选择另外一种比较清晰的实现方式, another way, 或许有惊喜。

应该说，这几种情况都应该考虑一下，例如对于上面的例子来说，要使用这么复杂的正则，加上一些简单的注释
是相当有必要的，至少要说明你是想验证什么样的格式。更进一步，封装到方法里边去，例如
{% highlight java %}
public static boolean isStrictYYYYMMDD(String datestr){
    return datestr.matches(STRICT_YYYYMMDD_REGEX);
}
{% endhighlight %}
不过这里有个缺点，只能校验一种日期格式，因为日期格式不像邮箱地址，它的形式多样，这样处理能得到的收益并不是很高。
如果我们可以传递校验日期的格式就更好了。

### 换个实现方式
换个思路，如果不使用正则表达式会怎样，例如[SimpleDateFormat][2]就提供了严格验证的格式，示例代码如下：
{% highlight java %}
public static boolean isStrictYYYYMMDD(String datestr){
    SimpleDateFormat format = new SimpleDateFormat("yyyyMMdd");  
    //设置为严格验证模式
    format.setLenient(false);
    try{
        format.parse(str);
        return true;
    } catch (ParseException e) {
        // ignore exeeption
    }
    return false;
}
{% endhighlight %}
如果没有设置为严格验证模式的话，20090230这种日期就会变成20090302。

相对于上面正则的方式来说，代码是多了几行，但是因为格式可以变，灵活性有所提高，代码也容易理解了。
另外一方面，由于SimpleDateFormat非线程安全，必须每次都定义一个，在多次处理的情况下显得有些多余。
当然有个折中的方法就是由客户端代码构造format作为传输传递，这样做还有个好处就是，验证日期格式的方法完全就是通用的。

例如，我们可以提供下面的api和调用方式:
{% highlight java %}
//client
SimpleDateFormat format = // 由客户端代码构建format
boolean isDate = DateUtil.isDateFormat(datestr, format);

//DateUtil api
boolean isDateFormat(String datestr, SimpleDateFormat format);
boolean isDateFormat(String datestr, String formatstr);//单次调用
boolean isStrictDateFormat(String datestr, String formatstr);//单次使用,用于严格处理
{% endhighlight %}

在不改变接口的情况下，最初的代码可以调整成以下形式
{% highlight java %}
public static boolean isStrictYYYYMMDD(String datestr){
    return isStrictDateFormat(datestr, DateUtil.YYYYMMDD);
}
{% endhighlight %}

### 总结

1. 隐藏某些复杂的细节是必要的，提供的接口要simple, clear。
2. 封装有助于焦距局部代码，即使要更换实现方式，也更加easy。
3. 可以提供通用可定制接口和常用特殊化接口,方便client调用。
4. [commons-lang][3]和[joda-time][4]开源库提供了非常成熟的解决方案。

 [1]: http://www.cnblogs.com/mgod/archive/2007/04/26/728628.html
 [2]: http://docs.oracle.com/javase/1.4.2/docs/api/java/text/SimpleDateFormat.html
 [3]: http://commons.apache.org/lang/api-2.5/org/apache/commons/lang/time/DateUtils.html
 [4]: http://joda-time.sourceforge.net/

{% assign series_list = "improve bitter code" %}
{% include series_list.html %}
