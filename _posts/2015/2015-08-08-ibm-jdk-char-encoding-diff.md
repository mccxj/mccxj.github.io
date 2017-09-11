---
layout: post
comments: true
title: "was中奇怪的生僻字乱码案例"
description: "was字符编码"
categories: ["was", "乱码"]
---

## 问题描述

这个今天早上提供的一个生产问题。大体是说，改资料的时候，有个客户的名字有生僻字，叫"刘",保存之后就乱码了，变成"刘?"

## 分析过程

乱码需要确认数据传输过程中编码方式。

1. 数据是通过jQuery的ajax过来的，并且没有提前处理数据(只有组装了一个js对象)，所以是采用encodeURIComponent进行处理的，对于中文可以很粗糙的理解成UTF-8编码过。这一点通过抓包工具是可以确认的。
2. 到了服务端之后会通过getParameter获取参数，由于带charsetEncoding的过滤器，并且是采用UTF-8的，那么这里拿到的字符串应该也是不会乱码的。

到了这里，代码并没有特别之处。按我的理解，只要字符集能够支持这个生僻字，就不会出现乱码。  
难道保存到数据库的时候乱码了? 目前数据库是用GBK的，我去查了一下GBK的字符表，的确是有这么个字的。

我在本机上测了一下这个字的各种功能编码转换，都是正常的。  
难道又是IBM的坑? 后来我又在服务器上测试了各种情况的输出，发现有另外一个字"䶮",除了字体大小有点不一样之外，几乎一模一样的。

下面整理了一个简单的测试程序，来说明这个奇怪的问题。

## 测试结果

首先要说明的是，这里有2个字,一小一大,还有它们对应的unicode和utf-8编码。  
测试结果是采用secureCRT的GB18030编码显示。

```text
有两个字:       小        大
unicode        \uE863    \u4dae
浏览器(utf-8)   %EE%A1%A3  %E4%B6%AE
```

下面的测试代码，为了编译时不关心字符集，所以换成utf-8字节来生成字符串。
```java
public class Test {
    public static void main(String[] args) throws java.io.UnsupportedEncodingException {
        new Test().test();
    }

    public void test() throws java.io.UnsupportedEncodingException {
        byte[] bbs = {-18,-95,-93,-28,-74,-82};
        String x = new String(bbs, "utf-8");
        String utf8 = new String(x.getBytes("utf-8"), "iso-8859-1");
        //byte[] bs = utf8.getBytes("iso-8859-1");  //test case 1
        //byte[] bs = x.getBytes("GBK");  //test case 2
        for(byte b : bs){
            System.out.println(b);
        }
        System.out.println(x);
    }
}
```

对于Test Case 1, 测试一下字符串是不是本来就乱了。测试结果显示，2个字都正常，要输出成GB18030才是可以的(secureCRT设置GB18030编码)。
```text
>/tools/jdk1.6.0_20/bin/java -Dfile.encoding=GBK Test
-18
-95
-93
-28
-74
-82
䶮?
>/opt/IBM/WebSphere/AppServer/java/bin/java -Dfile.encoding=GBK Test
-18
-95
-93
-28
-74
-82
?䶮
```

```text
>/opt/IBM/WebSphere/AppServer/java/bin/java -Dfile.encoding=GB18030 Test
-18
-95
-93
-28
-74
-82
䶮
>/tools/jdk1.6.0_20/bin/java -Dfile.encoding=GB18030 Test
-18
-95
-93
-28
-74
-82
䶮
```

对于Test Case 2，主要测试一下转换成GBK字节的情况,因为这是保存到数据库的必要转换。  
测试结果显示，ibm的jdk下，第一个字会编程乱码(对应的是63)。
```text
>/tools/jdk1.6.0_20/bin/java -Ddefault.client.encoding=GBK -Dfile.encoding=GBK Test
-2
-97
63
?
>/opt/IBM/WebSphere/AppServer/java/bin/java -Ddefault.client.encoding=GBK -Dfile.encoding=GBK Test
63
-2
-97
?
```

## 现象总结

1. 在GBK字符表中，第一个字是存在的，第二个字不存在。在GB18030中两个都存在。从显示上，也证明了GBK和GB18030并不完全兼容。
2. IBM的jdk为找不到第一个字，但能找到第二个字。oracle的jdk刚好相反。
3. 尝试使用百度拼音输入的时候，是可以找到2个字的。如下图的第2和第6个字。
4. 客户需要的是小的字(第一个)，但使用IBM的jdk转换GBK是找不到这个字的，一定会乱码。
5. 假设从前台输入的是第二个字，IBM的jdk应该是可以正常转换并得到的"正确"的字(正确的小字)，从而保证数据库不乱码。

![yan](/assets/images/2015/yan.png)

规避方法，选择输入第二个字(大字，截图中的第二个字，应该看不出有什么区别)。话说回来，感觉这是ibm的jdk的bug，字符对应错了。

## 相关资料

* [各种字符集编码表](https://github.com/willonboy/ChineseToPinYin)
* [GBK编码表](http://ff.163.com/newflyff/gbk-list/)
