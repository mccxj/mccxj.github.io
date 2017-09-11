---
layout: post
comments: true
title: "log4j源码心得"
description: "learn log4j source"
categories: ["java", "log4j", "source reading"]
---

log4j是一个非常流行的日志框架，最近研究了一下log4j的源代码，这里记录一下心得体会。

### Logger、Appender、Layout之间的结构关系?

这是log4j的基本数据结构，之间的关系还是很好理解的。首先，一个Logger都要一个名字(通常是类名或包名),它是有父子结构的，相关的实现是Hierarchy。  
一个Logger会对应多个Appender，表示可以日志输出的地方。  
Appender的输出格式是通过Layout来实现的，Logger在输出的时候是包装一个LogEvent对象，有Layout来格式化成一个字符串。

简单点:
```xml
Logger 1--* Appender 1--* Layout
```

### 加载log4j的配置文件

先考虑加载的是log4j.xml, 然后才是log4j.properties。我平时主要使用log4j.properties.

### log4j果然是年代久远

以前都说log4j在jdk1.1版本就存在了，现在从源码的细节上看，的确是这样的。例如:

1. 加载配置文件的时候，使用context class loader，它不是用标准的api，而是采用反射。
2. 处理FileAppender的时候，针对父级目录不存在的时候需要创建目录，如果是我就直接使用mkdirs，它确是手工处理。
3. Logger的名字是通过CategoryKey作为key存储到HashTable的，按理说用String作为key就可以了，估计是很早之前String的hashcode是不带缓存实现的吧。
4. 

### 其他细节

#### 没有配置文件的时候，如何优雅处理?

正常的时候，会在static初始化的时候得到LogRepositorySelector，如果失败的话，这个值就是空的。  
然后就生成一个repositorySelector = new DefaultRepositorySelector(new NOPLoggerRepository());，这个处理方式就是进行空输出。

#### log4j配置中的占位符处理

log4j配置中的占位符处理是在OptionConverter的substVars方法处理。优先使用系统变量，然后才是properties配置的定义
log4j.threshold用来配置log level的阀值，就是最低级别。默认是不设置。

#### log4j在重复加载文件时的处理

log4j在重复加载的时候，有个做法可以学习一下。这个做法和commons-logging的方式是一样的。

```
  void doConfigure(java.net.URL configURL, LoggerRepository hierarchy) {
    Properties props = new Properties();
    LogLog.debug("Reading configuration from URL " + configURL);
    InputStream istream = null;
    URLConnection uConn = null;
    try {
      uConn = configURL.openConnection();
      uConn.setUseCaches(false);
      istream = uConn.getInputStream();
      props.load(istream);
    }
```

#### layout或者appender的参数设置

log4j配置中的有时候组成一个layout或者appender，都是可以设置参数的，这些是怎么对应到类中的变量的? 实现在
```
 PropertySetter.setProperties(layout, props, layoutPrefix + ".");
```
通常还需要实现OptionHandler来做一些后续的初始化动作.

#### appender filter如何排序?

log4j的appender filter是可以配置多个的，处理顺序是按照对应的key的排序来的

```
    // sort filters by IDs, insantiate filters, set filter options,
    // add filters to the appender
    Enumeration g = new SortedKeyEnumeration(filters);
```

但是它为什么不使用内置的Arrays sort或者Collections sort呢?
```
  public SortedKeyEnumeration(Hashtable ht) {
    Enumeration f = ht.keys();
    Vector keys = new Vector(ht.size());
    for (int i, last = 0; f.hasMoreElements(); ++last) {
      String key = (String) f.nextElement();
      for (i = 0; i < last; ++i) {
        String s = (String) keys.get(i);
        if (key.compareTo(s) <= 0) break;
      }
      keys.add(i, key);
    }
    e = keys.elements();
  }
```

#### 输出编码没有设置会有什么问题? 

如果默认编码不支持中文，可能就乱码。其他不会乱码，只是写的文件是不同编码。编码这块参考WriterAppender
```
  protected
  OutputStreamWriter createWriter(OutputStream os) {
    OutputStreamWriter retval = null;

    String enc = getEncoding();
    if(enc != null) {
      try {
	retval = new OutputStreamWriter(os, enc);
      } catch(IOException e) {
          if (e instanceof InterruptedIOException) {
              Thread.currentThread().interrupt();
          }
	      LogLog.warn("Error initializing output writer.");
	      LogLog.warn("Unsupported encoding?");
      }
    }
    if(retval == null) {
      retval = new OutputStreamWriter(os);
    }
    return retval;
  }
```

#### bufferedIO和bufferSize、immediateFlush有什么关联?

bufferSize只有在bufferIO开启的时候才生效，对应的是BufferedWriter对象。  
如果开启了bufferIO那么immediateFlush就会重置成false。

#### RollingFileAppender多进程写的情况?

RollingFileAppender存在一个rollOver的操作，如果文件大小超过限制，就会进行切换。  
如果存在多个进程写的时候，就很可能出现文件大小明显超过限制的情况。  
原因在于RollingFileAppender初始化的时候就记录当前文件的大小，每次比较的依据是文件大小+曾经写入的文件大小。

```
  protected
  void subAppend(LoggingEvent event) {
    super.subAppend(event);
    if(fileName != null && qw != null) {
        long size = ((CountingQuietWriter) qw).getCount();
        if (size >= maxFileSize && size >= nextRollover) {
            rollOver();
        }
    }
   }
```

另外，这里的文件大小限制是字节，但统计大小的时候，用的是字符串的长度，两者是有些区别的。

```
  public
  void write(String string) {
    try {
      out.write(string);
      count += string.length();
    }
    catch(IOException e) {
      errorHandler.error("Write failure.", e, ErrorCode.WRITE_FAILURE);
    }
  }
```

#### DailyRollingFileAppender切换

基于一种时间窗口的算法。  
首先，如何知道是按分钟、小时还是天数来切换呢？就是靠猜，例如猜测是否是小时，那么就是给一个时间去格式化，再加上一小时去格式化。
如果两个格式化的字符串一样，那么说明不是。按时间间隔，从小到大判断就可以做到。  
接下来，根据日期格式，就可以计算下次切换的时间点(文件名)。  
最后，判断切换的时候，根据当前时间去格式化，看看是不是不一样了，如果是，那么就开始切换了。

不过，如果多个进程写的话，就很容易出现问题。因为下面的renameTo很容易失败，然后就会继续写到原来的文件。

```
    File file = new File(fileName);
    boolean result = file.renameTo(target);
    if(result) {
      LogLog.debug(fileName +" -> "+ scheduledFilename);
    } else {
      LogLog.error("Failed to rename ["+fileName+"] to ["+scheduledFilename+"].");
    }
```
