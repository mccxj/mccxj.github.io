---
layout: post
comments: true
title: "关于编码与乱码问题"
description: "关于编码与乱码问题,主要涉及java和web"
categories: ["编码", "乱码"]
---

## 关于java的编码

* java的源代码编码格式和最终的运行是没什么关系的。你可以使用GBK或UTF-8来编程。
* java编译后的class文件都是使用UTF-16来存储和运行的。
* 在eclipse中是根据文件设置字符编码来编译的，所以可以对不同文件使用有不同的编码，但这个不推荐。
* 使用javac编译可以通过-encoding指定字符编码,如果不指定，会使用系统默认编码，这个跟平台有关。所以使用ant需要指定编码。

下面这种在ant中常见的警告，就是表示编译用的编码和编程的编码不一致。
```java
CCustGroupPrompt.java:43: 警告：编码 UTF-8 的不可映射字符
```

更多java相关的，见[java字符编码问题](20150114_java-charset-problem.html)

## 关于jsp的编码

* jsp内容字符编码是pageEncoding指定的，用于指导jsp的编译器进行编译成java/class文件。如果没设置会采用contentType。
* contentType是用于response的输出http报文时的编码，浏览器根据ContentType来采用何种字符编码显示。和使用response.setCharacterEncoding()是一个道理的。
```java
<%@ page contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
```

如果page设置的内容，和页面中的meta设置不一样，那又怎样?

* meta是用来设置当前网页后续处理的默认编码，和当前页面的响应无关。
* 只要page设置和jsp文件内容实际字符编码一致，就不会乱。

如果不设置page，那又怎样?

* 如果没有指定page的话， contentType默认是text/html，浏览器会根据meta指定的编码来解析报文。这个时候，如果jsp内容实际编码和meta指定的编码一样，就能够显示正常。
* 编译后的java文件内容，其实都是不能显示中文的。经测试，发现是用的ISO-8859-1读取的文件,并转换成UTF-8的java文件。
* 如果使用UTF-16来编写jsp，但是不指定page，编译后的java和class反编译都是能够显示中文的。并且在tomcat下(其他未测试)，即使meta设置的编码不一样，也能够显示中文，因为这个时候contentType变成text/html;charset=UTF-16BE，具体大家可以查看编译后的java文件。感觉在编译的时候能够优先识别到UTF-16一样。

上述情况只是在tomcat上测试过，并不代表其他在中间件也是同样的情况，实际应用中应该确保jsp内容的字符编码、page设置、meta设置保持一致，避免一些灵异事件。

## 关于URL的编码

这里指直接通过URL传递中文，或者手工拼接中文到URL的情况，究竟使用何种编码传递没有规定，看浏览器心情，不具可移植性。
对于IE来说，虽然高级选项上有个发送UTF-8 URL，但不一定会勾上。如果真的要使用，应该自行编码后传递。

## 普通表单提交

使用GET或者POST，对于编码来说，没有区别。
都会对中文进行编码，编码采用页面的字符编码。

例如"中文"的UTF-8编码是E4B8ADE69687，传递的内容就是%E4%B8%AD%E6%96%87。

页面编码是通过<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">指定的。
而html5页面可以通过<meta charset="utf-8">这种简化形式指定。

## 表单文件上传

需要使用POST，并在form中增加属性enctype="multipart/form-data"。
不会对文件名、输入框内容进行字符编码。
采用页面的字符编码，对内容原样传输。

区别可能不是很好理解，下面举例：
例如"中文"的UTF-8编码是E4B8ADE69687，传递的内容是字节E4B8ADE69687，或许在某些工具上可以直接看到“中文”.(像fiddler用utf-8来显示的)

## Java/Servlet/Struts2(commons-upload)对参数的处理

Java/Servlet对参数的处理
* 默认只能获取到普通表单的参数提交。
* 编码格式通过request.setCharacterEncoding("UTF-8")指定，这个已经有过滤器可以实现的了。
* 使用Struts2的话，对multipart/form-data的提交也是能够获取通过getParameter取到参数的。

注意的是，有些实现(如tomcat)，对参数的解析是延后处理的，设置了编码之后，获取一个参数(这个时候参数全部都解析了)，再设置编码是没有效果。ServletRequest的setCharacterEncoding描述也是这么说的。

标准的commons-upload，文件名的获取、输入框内容的获取使用的编码可能不一样。
* 文件名的获取，就是FileItem.getName(),解析编码需要通过ServletFileUpload#setHeaderEncoding这个方法设置，如果没有设置，采用平台编码(可以通过-Dfile.encoding=UTF-8来指定，否则win通常是ANSI(GBK),unix看locale)
* 输入框内容，就是FileItem.getString(),可以指定解析编码，如果不指定采用ISO-8859-1。

Struts2默认使用commons-upload进行文件上传的处理。
* 对于文件名的获取没有通过setHeaderEncoding设置，所以这个通常会依赖于平台编码(需要确保平台编码和页面编码一致)
* 对于输入框内容的获取，指定了编码格式为request.getCharacterEncoding()，否则采用默认的ISO-8859-1。所以这个需要提前设置一下CharacterEncoding，否则也可能会乱码。