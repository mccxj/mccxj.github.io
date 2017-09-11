---
layout: post
comments: true
title: "Android-Universal-Image-Loader源码快扫"
description: "阅读android universal imager loader源码"
categories: ["android", "universal", "源码"]
---

## ImageLoaderConfiguration/ImageLoaderConfiguration.Builder学习

构造复杂对象的方式: Builder模式,用来创建ImageLoaderConfiguration对象,适用于链式写法。

常见结构如下：**关键点: 私有构造(拷贝而非应用，避免build复写)、内部类(影响局部化,内聚好)、返回this(支持链式)**
```java
class A {
  private A(Builder builder) {
    // create A with builder's copy
  }

  public static class Builder{
     Builder buildStep1(){
        //...
        return this;
     }
     
     Builder buildStep2(){
        //...
        return this;
     }   
     
     A build() {
        return new A(this);
     }  
  }
}
```

## ImageLoaderConfiguration支持的特性

```
-- 源码没什么养分，关注特性是如何表现在内部结构上。
-- 类层次的结构，能体会就体会，不体会就拉到。层次是渐进实现的，不能体会也没什么。
-- 如果有机会的话，可以在实践项目中调试进去学学。
```

如果想学习相关特性是怎么实现的，可以根据配置的去反推实现代码(搜索或调用关系):
```
基本特性（可配置）:
  基本控制:
    线程池大小 -- 不能太多
    线程优先级 -- 略低
    请求任务排队  -- 默认FIFO
    是否开启调试日志 -- DEBUG log
  多级缓存特性:
    内存缓存大小
    硬盘缓存大小，文件数量限制，文件名生成规则
    -- 关联硬盘图片处理器BitmapProcessor
  图片专用:
    缓存图最大宽高 --- 用于约束图片大小
    显示图的约束
    -- 关联下载器ImageDownloader(还通过networkDeniedDownloader/slowNetworkDownloader区分不同情况，这不就是Null Object模式么?)
    -- 关联解码器ImageDecoder
```
  
## 关于ImageLoader如何解决错乱问题

ImageLoader就是一个singleton实现，配合init+ImageLoaderConfiguration进行初始化，没什么说的。很常见的设计实现。

大多是helper method 关注displayImage/loadImage最终实现即可。这也是常见做法，便于使用。

问题: url -- 关联的view，问题在于url请求是异步的，而view可能被重复利用。  
这里用的ImageAware，看他们的实现类，有个ViewAware，就是一个view的包装，我想说的是WeakReference在android中很常用呀。 -- 这句是废话

```
使用ImageAware占位
  图片宽高
  实际的View --可重用
  id -- 尼玛，这是解决问题的关键
```
  
见DisplayBitmapTask有个isViewWasReused倒出重点，从它的实现就可以判断完整的算法拉。

内部结构 cacheKeysForImageAwares map<ImageAware.id, memorycachekey(由URI + 宽高生成)>

不过ViewAware的id是用view的hashcode来指定的， NonViewAware的id是用url来指定的。  
so， listview的时候，使用loadImage是可以避免错乱的，而用displayImage就呵呵拉。  
判断方式如下：根据id可以找到目前的url和当前的url比较即可。知道是不是被复用了。

相对于原理上的image#setTag(url)然后比较的方式，更加透明，侵入性少。

## 再看看ImageLoaderEngine在任务管理、线程方面的处理

原以为这货应该比较好处理，图样图森破呀，毕竟它支持多种状态(暂停，恢复，关闭等)

先说一下几个AtomicBoolean的变量，主要就是判断状态的boolean拉，当然这货是线程安全的，其实用boolean也行，毕竟也是原子的(注意可见性问题即可)
```
	private final AtomicBoolean paused = new AtomicBoolean(false);
	private final AtomicBoolean networkDenied = new AtomicBoolean(false);
	private final AtomicBoolean slowNetwork = new AtomicBoolean(false);`
```

另外，还有3个线程执行器，简单看看用途。至于为什么要这么多个，哥认为是这样的：

* 在loader中走进displayImage之后，它重要检查一下内存中有木有(耗时小，无需线程)。
* 如果没有才会扔给engine，engine首先检查一下硬盘上有木有(有io，所以走taskDistributor)。
* 如果硬盘上有，走taskExecutorForCachedImages(同理有io)
* 还是没有，走其他(maybe网络io)
* 在请求较多，兼顾命中、不命中的情况，它选择了采用多级的Executor

```
	private Executor taskExecutor; -- 木有在disk的情况，一般走网络
	private Executor taskExecutorForCachedImages; -- 在disk的情况
	private Executor taskDistributor;  -- 根据情况分发给上面2个
```

还有2个map，第一个就是用来保存id和图片url的对应关系(简单是这么理解)  
另外是用来控制不同view但有相同uri的并发请求。
```
	private final Map<Integer, String> cacheKeysForImageAwares = Collections
			.synchronizedMap(new HashMap<Integer, String>());
	private final Map<String, ReentrantLock> uriLocks = new WeakHashMap<String, ReentrantLock>();
```

不熟悉ReentrantLock的童鞋可参考 https://www.ibm.com/developerworks/cn/java/j-jtp10264/

LoadAndDisplayImageTask通过控制相同的uri持有同一个锁，这样执行的时候，后面的就会等待。  
具体可以参考LoadAndDisplayImageTask的实现，不过用ReentrantLock需要特别注意写法，避免死锁。  
不过我认为getLockForUri并没有同步，还是有存在相同url获取到不同lock的可能，不过这不影响功能，而且受限于并发大小也很难出现。

## 关于url的中文

网上有看到一些人说中文图片名会出错，可能是很久之前的版本吧。我这里要说的是，我认为这是个简单的问题，即使改源码也很容易处理。  
不过即使源码有相关处理，通常也不会关注的，不过今天刚好有人问我一个中文路径的问题，所以我就关注了一下它怎么实现的。

首先看BaseImageDownloader是如何请求中文图片的。由于http只是支持ascii的url编码，所以必须要编码的，通常用utf-8,虽然这个没规定。貌似我们的基线没有考虑这个问题，应该是没有掉过坑。
```
	protected HttpURLConnection createConnection(String url, Object extra) throws IOException {
		String encodedUrl = Uri.encode(url, ALLOWED_URI_CHARS);
		HttpURLConnection conn = (HttpURLConnection) new URL(encodedUrl).openConnection();
		conn.setConnectTimeout(connectTimeout);
		conn.setReadTimeout(readTimeout);
		return conn;
	}
```

另外，还有一个容易出现中文问题的就是保存到硬盘的情况(android貌似不是什么大问题，如果是做server开发的话，就得特别注意)  
带的实现有HashCodeFileNameGenerator(默认)和Md5FileNameGenerator，这2种处理都不会产生中文问题。  
不过我建议还是用Md5FileNameGenerator，hashcode做唯一性并不是很靠谱，如果是大量的固定文件名长度的图片，还是很容易冲突的。

## 小结

源码何其多，带着问题学习效果更好。挑几个疑惑看看别人怎么处理就是收获。    
从类的层次着手是很困难的，特别是大型源码。了解上层架构，学示例，然后调调源码或许更好。
大多数情况，相对于细节，应该更关注关键数据的结构、如何组织数据的结构来解决问题。  
如果自己设计，应该考虑的重要问题有: 如何使用? 用怎样的结构表示数据和状态?   
XX设计模式不要硬套，从过程式演变出来更加自然(经验性的除外)。推荐重构与模式。  
大而全的源码解读没有什么用，带问题分析的更有价值。  
学好基础，模仿起来也不容易掉坑。

-- 以上纯属肉眼分辨，并无调试过，不做正确性验证，仅供参考。