---
layout: post
comments: true
title: "java常见工具库培训"
description: "一些常见工具库的用法"
categories: ["工具库", "培训", "Java"]
---

目前项目中常见的工具库有apache commons,google guava,再算上spring的话，需要自己从头开始写工具类的情况大大减少。
为了给广大童鞋普及一下工具库用法，减少无用功(还可能因为实现的不好留后遗症的)，这里简单的介绍一下相关工具类。**google guava大家应该比较陌生，这里先不介绍，:)**

## apache commons

官方地址： http://commons.apache.org/

apache commons历史悠久，涉及范围也是最广的，在官网上分了数十个模块，但有些模块是新开发的，就不要贸然使用啦。

这里只是介绍最最常用的commons库，排名不分先后，如下：

### commons-codec

包括常见的编码、解码算法，例如MD5,Base64，举例如下:

* Base64#encode 加密成base64串
* Base64#decode 解密base64串
* DigestUtils.md5Hex 进行MD5加密，注意得到的是小写的MD5(MD5标准不区分大小写),在比较的时候需要注意
* DigestUtils.shaHex 进行SHA1加密 SHA256,512之类也是支持的，可以自行查阅

### commons-collections

包括一堆增强的集合类（我了解不多，大家可以自行学习），各种和集合类相关的工具类，举例如下：

* CollectionUtils.isEmpty 是否null或空集合，这一类的方法很多，看看有个大概印象
* MapUtils.isEmpty 是否null或空Map
* ListUtils.removeAll 从某个列表中删除存在于另外个列表的元素

同类型的还有SetUtils、IteratorUtils等，大体上是集合相关的操作，如过滤、是否相等、交集、差集、转换(变同步、变不可变)等，其实这个用到的机会也不是很大。

### commons-net

实现了一些常见的网络协议，可能关系最大的要数ftp、smtp的实现了。而jdk带的sun.net.ftp，这个尽量就少用拉。

这套api的实现用法得google一下了，看[官方文档的例子](http://commons.apache.org/proper/commons-net/),
又或者别人的经验代码，例如这个http://my.oschina.net/hly3825/blog/33657

### commons-httpclient

http客户端实现，貌似已经从commons独立出去了。3.x版本和4.x版本变化比较大，大家要使用的时候自行查阅资料。
尽量避免使用HttpURLConnection去直接搞。

### commons-io

io方面的工具类，主要包括文件处理、流处理,常见的类有IOUtils、FileUtils、FilenameUtils。举例如下：

* IOUtils.closeQuietly 安静关闭输出输出流，常用于finally关闭流的时候
* IOUtils.copy 把某个输入流拷贝到某个输出流中去
* IOUtils.toString 把某个输入流、URI的内容转换成字符串
* IOUtils.readLines 按行读取流
* Charset.UTF_8 有一些常见的、系统都会支持的字符集，已经定义成常量
* FileUtils.readLines 按行读取文件
* FileUtils.readFileToString 读取文件保存在一个字符串中

IOUtils针对的是stream，FileUtils针对的是File对象，相应的有文件拷贝、删除等操作。  
注意的是，**使用字符流格式的时候，务必指定编码**

### commons-lang

这个是使用最多的库了，有lang2.x和3.x版本，尽量使用3.x版本。

常见的有StringUtils、SystemUtils、RandomStringUtils、DateFormatUtils、DateUtils、各种Builder、Validate，举例如下：

* StringUitls.isEmpty 判空，和isBlank的区别在于它不进行trim
* StringUtils.join 按分隔符合并，这个很常用
* StringUtils.repeat 重复某个字符或字符串，有些需要格式化的是会用到
* StringUtils.startsWith  和endsWith那样，是增强版本，还有endsWithAny、endsWithIgnoreCase等
* SystemUtils 主要是一些常见系统环境变量，如临时目录、用户目录、分隔符等
* RandomStringUtils 用来生成各种随即字符串，例如全字母、全数字或混合型的
* DateFormatUtils、DateUtils 一个是字符串变日期，一个是日期相关的操作
* 各种Builder 主要用实现常见的toString、compareTo、equals、hashcode等常见类，例如ReflectionToStringBuilder就很方便实现toString方法。同理，CompareToBuilder、EqualsBuilder、HashCodeBuilder都很好理解。
* Validate 实现一些assert，例如Validate.notNull可以用来做前置校验，和spring的Assert类是类似的。

### 其他commons库

* commons-fileupload 仅限于在文件上传的类中使用，虽然它也有一些工具类，但是就不要在其他地方使用啦。
* commons-dbcp 一个数据库连接池，现在就比较少用了
* commons-pool 一个java对象池实现，通常用来缓存一些耗时较大的对象，dbcp也是基于它的，一般也少直接用。
* commons-logging 日志包装实现，在开源项目中使用广泛，项目中一般直接用log4j等。
