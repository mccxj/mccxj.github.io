---
layout: post
comments: true
title: "java字符编码问题"
description: "java字符编码问题,尝试一下"
categories: ["编码", "乱码"]
---

1.假设文件用UTF-8保存了中文"操作计算机"，然后使用GBK编码进行读取?
```java
String str = FileUtils.readFileToString(new File("/myfile"), "GBK");
System.out.println(str);
str = new String(str.getBytes("GBK"), "UTF-8");
System.out.println(str);
```

可以发现，后续转成UTF-8仍然有部分乱码，如果保存的内容是"操作计算"就不会乱码。为什么?

2.继续上述问题，如果使用ISO-8859-1进行读取?
```java
String str = FileUtils.readFileToString(new File("/myfile"), "ISO-8859-1");
System.out.println(str);
str = new String(str.getBytes("ISO-8859-1"), "UTF-8");
System.out.println(str);
```

可以发现，可以发现无论是"操作计算机"还是"操作计算"、"操 作计算"，都不会乱码。为什么?

3.如果文件采用GBK编码保存中文，但是使用UTF-8读取，就会发现怎么转都是乱码? 为什么?

4.假设代码如下，为什么前面3行都是输出乱码?
```java
System.out.println(new String("123你".getBytes("ISO-8859-1"), "ISO-8859-1"));
System.out.println(new String("123你".getBytes("ISO-8859-1"), "GBK"));
System.out.println(new String("123你".getBytes("ISO-8859-1"), "UTF-8"));
System.out.println(new String("123你".getBytes("GBK"), "GBK"));
System.out.println(new String("123你".getBytes("UTF-8"), "UTF-8"));
```

5.请思考，下面的同样掺和了ISO-8859-1，为什么却能正常?
```java
System.out.println(new String(new String("123你".getBytes("GBK"), "ISO-8859-1")
        .getBytes("ISO-8859-1"), "GBK"));
```

6.假设使用http发送xml，那么xml报文采用何种编码发送和xml的编码头部指定的编码有什么关系?
```java
<?xml version="1.0" encoding="GBK" ?>
```