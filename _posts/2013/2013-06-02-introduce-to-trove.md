---
layout: post
comments: true
title: "性能优化:Trove集合库"
categories: ["trove", "collections", "性能", "优化"]
---

## 1 初见Trove

昨天在[Startup News](http://news.dbanotes.net/news)上看到一篇文章:[优化技巧分享：把内存消耗降低至原来的1/20](http://blog.jobbole.com/40666/)。里边提到了一个案例，Java应用中如何降低内存消耗，总结了他的优化过程:

* 开始时,存放1.3M个Person对象，消耗堆空间1.5GB
* 修改为java.util.HashMap<Long, Person>进行缓存,从1.5GB降低为214MB
* 修改HashMap为Trove的TLongObjectHashMap<Person>，从214MB降低为143MB
* 优化Person内部结构,减少重复字段,从143MB降低为93MB
* 开启64位JDK的XX:UseCompressedOops参数进行指针压缩,从93MB降低为73MB

优化经常得针对具体的场景、数据特性来优化，上述提到的[Trove](http://trove.starlight-systems.com/)集合库就是这么一个典型例子,它针对的是JDK集合类中处理原生类型的场景。

## 2 使用Trove

* 如果使用Maven的话,可使用下面的配置

``` xml
<dependency>
	<groupId>net.sf.trove4j</groupId>
	<artifactId>trove4j</artifactId>
	<version>3.0.3</version>
</dependency>
```
* 常用方法和JDK集合类是一样的，方便迁移

``` java
TIntObjectMap<String> ints = new TIntObjectHashMap<String>();
ints.put(100, "John");
ints.put(101, "Tom");
System.out.println(ints.get(100));
```

Trove相当于把JDK集合类都针对原生类型处理了一遍，例如int，常见的类有
TIntList、TIntObjectMap<V>、TObjectIntMap<K>、TIntSet，可想而知，**维护Trove的工作量是挺大的**。

Trove还提供了开放寻址法的Map,Set,LinkedList实现,可以参考[Enhance Collection Performance with this Treasure Trove](http://www.onjava.com/pub/a/onjava/2002/06/12/trove.html?page=2)的做法,类似于:

``` java
public class CollectionFactory {
	static boolean useTrove = true;

    /**
	 *  Return a hashmap based on the properties
	 */
	public static Map getHashMap() {
		if ( useTrove ) return new THashMap();
		else            return new HashMap();
	}

	/**
	 *  Return a hashset based on the properties
	 */
	public static Set getHashSet() {
		if ( useTrove ) return new THashSet();
		else            return new HashSet();
	}

	/**
	 *  Return a linkedlist based on the properties
	 */
	public static List getLinkedList() {
		if ( useTrove ) return new TLinkedList();
		else            return new LinkedList();
	}
}
```

* 迭代集合中的元素

**Trove不推荐JDK的entryXX的做法，而是采用了forEach的回调方式**。
代码显得更好看些，另外内存方面也有优势，因为使用entryXX的做法，需要创建一个新的数组。

``` java
TIntObjectMap<String> ints = new TIntObjectHashMap<String>();
ints.put(100, "John");
ints.put(101, "Tom");
ints.forEachEntry(new TIntObjectProcedure<String>() {
    public boolean execute(int a, String b) {
        System.out.println("key: " + a + ", val: " + b);
        return true;
    }
});
ints.forEachKey(new TIntProcedure() {
    public boolean execute(int value) {
        System.out.println("key: " + value);
        return true;
    }
});
ints.forEachValue(new TObjectProcedure<String>() {
    public boolean execute(String object) {
        System.out.println("val: " + object);
        return true;
    }
});
```

* 自定义Hash策略

我们知道，在JDK集合类里边，有时候是没法自定义Hash策略的，例如String。
**不过Trove提供了自定义Hash策略的功能，让你可以根据数据特性进行优化**。

``` java
public static void main(String[] args) {
    char[] foo = new char[]{'a', 'b', 'c'};
    char[] bar = new char[]{'a', 'b', 'c'};
    TCustomHashMap<char[], String> ch = new TCustomHashMap<char[], String>(new CharArrayStrategy());
    ch.put(foo, "John");
    ch.put(bar, "Tom");
}

class CharArrayStrategy implements HashingStrategy<char[]> {
    public int computeHashCode(char[] c) {
        // use the shift-add-xor class of string hashing functions
        // cf. Ramakrishna and Zobel, "Performance in Practice
        // of String Hashing Functions"
        int h = 31; // seed chosen at random
        for (int i = 0; i < c.length; i++) { // could skip invariants
            h = h ^ ((h << 5) + (h >> 2) + c[i]); // L=5, R=2 works well for
                                                  // ASCII input
        }
        return h;
    }

    public boolean equals(char[] c1, char[] c2) {
        if (c1.length != c2.length) { // could drop this check for fixed-length
                                      // keys
            return false;
        }
        for (int i = 0, len = c1.length; i < len; i++) { // could skip
                                                         // invariants
            if (c1[i] != c2[i]) {
                return false;
            }
        }
        return true;
    }
}
```

## 3 Trove内幕

Trove是以减少内存消耗为主要目的的，同时也要保持性能。我们这里简单描述一下Trove的实现内幕。
这里有有另外一篇文章可以参考：[性能观察: Trove 集合类](http://www.ibm.com/developerworks/cn/java/j-perf09284.html)

* 直接使用原生类型,而不是包装类型

JDK5的自动封箱机制，让我们可以暂时忽略原生类型和包准类型的区别。自动封箱机制只是一种语法糖，实际上并没有提高效率。
直接使用原生类型替代包装类型，明显可以占用更小的内存、运行起来也更有效率。对于基本类型的集合组合，Trove都提供了
等价的集合类。

* 使用开放寻址法，而不是链地址法

大多数的JDK集合类都是采用链地址法实现的，它需要一个地址表，并且元素之间需要链表结点，而Trove采用开放寻址法，
虽然需要保持足够的空闲位置(装载因子小于0.5),但因为不需要链表结点，所以总体上内存占用要更少，性能还要更快一些。

* HashSet不再通过内置HashMap实现

JDK的HashSet是通过内置一个HashSet来实现的，所以白白浪费了value的空间。
Trove提供的THashSet和其他基本类型的HashSet,都不再采用这种方式，直接使用开放地址存储。

* 采用素数长度大小的数组

为了最大程度避免hash冲突，除了保持较小的装载因子，还采用了素数长度大小的数组。具体见gnu.trove.impl.PrimeFinder

* 采用代码生成进行维护

虽然这个和性能没什么关系。但是我们也知道要维护这么多的原生类型集合类，重复的逻辑多但没法重用，是个很纠结的事情。
Trove采用代码模板，生成大量的类，通过这种方式，可以大大减少维护的工作量。

## 4 总结

JDK作为通用集合类，大多数情况下我们还是会优先选择的。不过，在一些性能敏感的地方，或者Trove可以提供更好的选择。
作为靠谱的java开发人员，Trove应该像apache commons、google guava那样，存放在你的工具箱里边。