---
layout: post
comments: true
title: "关于split的坑"
categories: ["java hell", "split", "java"]
---

经常在一些文本处理、字符串拆解的逻辑中，需要对字符串按照一定的分隔符进行拆解。这样的功能，大多数时候我们会求助于String的split方法。关于这个方法，有几个坑是需要注意的，而掉坑想象在代码中已经出现许多次，值得大家注意。

### split的参数是正则表达式

一个很常见的问题，就是忘记了split的参数不是普通的字符串，而是正则表达式，例如下面这么下是达不到预期目的的:

```java
"a.b.c".split(".");
"a|b|c".split("|");
```

因为.和|都是正则表达式里边的特殊字符，应该用转义字符先进行处理:

```java
"a.b.c".split("\\.");
"a|b|c".split("\\|");
```
类似的api在String类里边好几处出现了，例如replaceAll。

还有没有其他办法处理这个问题呢，因为我不想手动转换或者分隔符本来就是动态的。
这个的确没有直接的方法，但split的实现是调用Pattern的split方法，所以可以直接构造一个Pattern对象来调用，
如下所示，其中LITERAL参数就是表示把字符串当成普通字符串，而不是当成正则表达式来构建。

```java
Pattern pattern = Pattern.compile(".", Pattern.LITERAL);
pattern.split("a.b.c");
```
### split可能会忽略分隔后的空串

很多人只是用带一个参数的split方法，但是一个参数的split方法只会匹配到最后一个有值的地方，后面的会忽略，例如:

```java
"a_b_c_d".split("_"); // ["a", "b", "c", "d"]
"a_b__".split("_"); // ["a", "b"]
"a__c_".split("_"); // ["a", "", "c"]
```

这样其实是有点反常识的，因为像文件上传，有些字段可能是允许为空的，这样在程序处理上就会造成麻烦。

其实，split是有带两个参数的重载方法的。第二个参数是整形，代表最多匹配上多少个，用0表示只匹配到最后一个有值的地方(就是上述split真正调用的参数)，要强制全部匹配，用个负数吧(我通常选择-1)。换成下面的写法，代码就更期望的结果是一样了。

```java
"a_b__".split("_", -1); // ["a", "b", "", ""]
"a__c_".split("_", -1); // ["a", "", "c", ""]
```

### 关于字符串切割的其他API

对于字符串切割，早在jdk1.0就存在一个叫StringTokenizer的类，大概的用法如下所示(同样有带分隔符的构造方法):

```java
StringTokenizer st = new StringTokenizer("this is a test");
while (st.hasMoreTokens()) {
  System.out.println(st.nextToken());
}
```

不过，这个类是历史产物，属于遗留类来的，javadoc上已经说明了这一点，并且推荐使用String的split方法。

另外，如果对字符拼接有兴趣，请移步[说说字符串拼接](/blog/20130809_java-hell-string-append.html)