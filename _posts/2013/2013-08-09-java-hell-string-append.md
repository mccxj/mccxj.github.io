---
layout: post
comments: true
title: "说说字符串拼接"
categories: ["java hell", "append", "java"]
---

### String对象是无状态的

String的内部属性在初始化的时候就固定好了，也没有提供方法进行修改(反射等极端方法除外)，并且类被定义成final，所以String对象都是是实实在在的无状态对象，是不可变的。

在通常的字符串拼接中，如果采用+运算符的话，通常会产生一个字符串对象，并把两个字符串的内部字符数组拷贝过去。
因此，在一个常见的频繁修改字符串的场景中，字符数组的拷贝开销是很大的，随之字符串的加长会越到后面越慢,例如下面的代码。

```java
String sb = "";
for(String str : strs){
  sb += str;
}
return sb;
```

### StringBuffer与StringBuilder

jdk早就考虑了这种场景，于是提供了StringBuffer，简单来说，就是一个可变的字符数组，开辟了一个字符数组缓冲区，增加(Append)时只是往缓冲区的空余地方写字符，当然也有可能缓冲区不够用，它的开销就集中在不够用的缓冲区扩展中(每次在现有基础上扩展一倍空间)。所以，最好能提前估计字符串的最终长度，减少扩展造成的消耗。不过，即便如此，通常也要把直接用String拼接的效率高许多，例如下面的代码。

```java
StringBuffer sb = new StringBuffer();
for(String str : strs){
  sb.append(str);
}
return sb.toString();
```

到了jdk5，增加了StringBuilder，相对于StringBuffer来说，虽然它不是线程安全的，但在绝大多数场景下都是适用的，并且理论效率更佳(从oracle jdk的实现看，两个类除了是否同步这点，实现是一致的)。因此，习惯使用StringBuffer的童鞋，应该多关注一下StringBuilder。

### 字符串拼接的编译优化

再回到+操作符，本身java是没有运算符重载的，+只会对基本数学运算有效，而字符串，这么写只是语法糖而已，会变成StringBuilder操作(jdk5之前是StringBuffer)。例如下面的代码:

```java
public String test(String a){
   return a + "b";
}
```

通过javap查看，可以看到是这样的(大意就是new一个StringBuilder对象然后用append进行连接);

```java
public java.lang.String test(java.lang.String);
  Code:
   Stack=2, Locals=2, Args_size=2
   0:   new     #2; //class java/lang/StringBuilder
   3:   dup
   4:   invokespecial   #3; //Method java/lang/StringBuilder."<init>":()V
   7:   aload_1
   8:   invokevirtual   #4; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   11:  ldc     #5; //String b
   13:  invokevirtual   #4; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   16:  invokevirtual   #6; //Method java/lang/StringBuilder.toString:()Ljava/lang/String;
   19:  areturn
  LineNumberTable:
   line 3: 0
```

因此，如果是像上面的情况，直接用+是合理的，对于其他情况，得考虑StringBuilder，同时要避免无意生成多余字符串的情况，例如append("s" + a)的写法，编译器是不会自动优化的，写代码的时候应该换成append("s").append(a)。

更多关于字符串不变量的讨论，请见[初探Java字符串](http://mccxj.github.io/blog/20130615_java-string-constant-pool.html)
