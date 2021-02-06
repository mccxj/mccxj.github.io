---
layout: post
comments: true
title: "Java类静态代码块并发案例"
description: "关于代码静态块的问题"
categories: ["Bug反馈", "静态块"]
---

我们某个服务，请求报文会有一个操作码的一段，代码中会根据操作码，找到对应的执行类，然后执行对应的接口方法。而大多数的执行类实现是会根据操作码，再分发到不同的方法去的。

但是生产系统偶尔出现一些找不到处理方法的情况，本地测试又没有遇到过。而且，有时候服务重启一下，就正常了。

## 问题分析

我们排查了所有的操作码，没发现有漏配置的情况，也就是说，方法是存在的，只是找不到。

再分析，有可能的就是哪里有并发的问题。

这里再说一下我们的数据结构和处理流程是怎样的。

这个代码是从一个大规模c++业务系统转过来的，通常在java里边是配置文件来处理这些处理方法的对应关系的，为了保持代码不大调整，就用了一些取巧的方式。

- 在基类上定义了一个protected静态的hashmap
- 在多个子类上用静态代码块给这个map进行方法注册
- 在应用启动的时候，扫描所有的类，把类名和操作码关联上(因为配置上还要兼容原c++代码，所以用的是simpleName，不能直接反射)
- 在请求过来后，通过操作码拿到执行类，反射new一个实例，这样就触发静态块的执行，达到注册的目的
- 可能很多人也注意到了，这里用到hashmap会不会有问题。按以前的习惯，一般静态块主要是初始化数据，在真正时候的时候，都是固定的值，用hashmap是可以的。

不过，这里是有些特殊的。我们知道一个类的静态块是可以保证线程安全的，只会执行一次，但是多个类的静态块是不是顺序执行的，这个以前是没有特别注意的。找了一些java语言规范，也没有提到这个内容。既然没规定，隐隐觉得这可能还真是问题。

写个例子测试一下。
```
class A {
    static {
        System.out.println("A =>" + new Date());
        try {
            Thread.sleep(1000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("A =>" + new Date());
    }
}

class B extends A {
    static {
        System.out.println("B =>" + new Date());
        try {
            Thread.sleep(10000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("B =>" + new Date());
    }
}

class C extends A {
    static {
        System.out.println("C =>" + new Date());
        try {
            Thread.sleep(10000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("C =>" + new Date());
    }
}

public class Test {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                new B();
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                new C();
            }
        }).start();
    }
}
```

结果如下,的确是可以并行执行的。

```
A =>Wed Jan 06 10:42:33 CST 2021
A =>Wed Jan 06 10:42:34 CST 2021
C =>Wed Jan 06 10:42:34 CST 2021
B =>Wed Jan 06 10:42:34 CST 2021
C =>Wed Jan 06 10:42:44 CST 2021
B =>Wed Jan 06 10:42:44 CST 2021
```

这样就可以解释这个现象了。我们知道hashmap在并发情况下有可能会死循环cpu100%，还有一个可能就是并发put可能丢数据。以1.6的hashmap实现为例子(相对1.8比较简单)。addEntry(hash, key, value, i)，如果有产生哈希碰撞， 导致两个线程得到同样的bucketIndex去存储，就可能会出现覆盖丢失的情况：
```
    /**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V put(K key, V value) {
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key.hashCode());
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
```

解决办法也很简单，改成线程安全的concurrenthashmap.

## 经验总结

- 多个类的静态代码块是可以并行执行的，不要在多个类的静态代码块中共享数据
- 确实要共享数据，要注意线程安全问题，考虑使用线程安全的对象