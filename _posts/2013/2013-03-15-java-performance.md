---
layout: post
comments: true
title: "java性能--降低时间与空间消耗"
categories: ["java", "性能", "时间", "空间"]
---

### 译者的话

断断续续的翻译完了，有些地方翻译得比较纠结，很难选择一个合适的表达。
中文和英文在表达上还是有很多习惯上的不同，我又不想掺杂太多自己的意译上去，搞得有点进退两难。
有能力的同学还是尽量阅读英文的吧，毕竟这篇文章的英文并不深。

**作者: Reter Sestoft(sestoft@dina.kvl.dk)**   
**对应版本:2005-04-13 Version2**  
[原文链接][1]

### 前言
我们提出一些通过降低时间与间消耗来改进java程序运行时间的建议。
这里没有什么魔术的技巧，仅仅在避免常见问题上提出建议。

### 1 降低时间消耗

#### 1.1 基本代码优化

不要期望java编译器(例如javac或jikes)去做许多聪明的优化。
因为java有比较严格的语句次序和线程语义，所以相对于C或者Fortran等比较少严格定义的语言，
要安全的提高java程序的性能，编译器能做的事情很有限。但是你可以改进你自己的java源代码，来做到这点。

**注: 理解为语句上的次序**   
**注: 严格定义指语法上限制**

* 把循环不变量的计算移到循环之外。例如，避免重复计算一个for循环里的边界，像这样:

``` java
for (int i=0; i<size()*2; i++) { ... }
```

应该仅计算一次循环边界，然后把结果赋值给一个本地变量，像这样:
``` java
for (int i=0, stop=size()*2; i<stop; i++) { ... }
```

* 不要重复计算同样的子表达式:

``` java
if (birds.elementAt(i).isGrower()) ...
if (birds.elementAt(i).isPullet()) ...
```

应该计算子表达式一次，然后把结果赋值给一个变量，并且重用这个变量:
``` java
Bird bird = birds.elementAt(i);
if (bird.isGrower()) ...
if (bird.isPullet()) ...
```

* 每一次数组访问需要一个索引检查，所以降低数据访问次数是值得的。另外，通常java编译器不能
自动优化多维数组的索引。例如，内循环(j)的每次迭代，重新计算索引rowsum[i]和arr的第一维索引arr[i]:

``` java
double[] rowsum = new double[n];
for (int i=0; i<n; i++)
  for (int j=0; j<m; j++)
    rowsum[i] += arr[i][j];
```

相反，在外循环的每次迭代中只计算一次这些索引值:
``` java
double[] rowsum = new double[n];
for (int i=0; i<n; i++) {
  double[] arri = arr[i];
  double sum = 0.0;
  for (int j=0; j<m; j++)
    sum += arri[j];
  rowsum[i] = sum;
}
```

注意的是，初始化中arri = arr[i]并没有拷贝数组的第i行;它仅仅把数组引用(4个字节)赋给arri.

* 把不变属性声明为final static，让编译器可以inline它们和预计算不变表达式。
* 把不变的变量声明为final，让编译器可以inline它们和不变表达式。
* 如果可以的话，把一个长的if-else-if链替换为switch；它要快很多.
* 加入一个长的if-else-if链不能替换为switch(例如因为它检测一个String)，
假如它执行很多次的话，通常值得替换为一个final static的HashMap，或类似的结构。
* 使用'聪明的'C惯用法没有什么用(除了让代码更隐晦)，例如一个while循环的循环条件中进行所有的计算工作:

``` java
int year = 0;
double sum = 200.0;
double[] balance = new double[100];
while ((balance[year++] = sum *= 1.05) < 1000.0);
```

#### 1.2 属性和变量

* 访问本地变量和方法中参数，要比访问静态属性或实例属性要快得多。
对于一个循环中的属性访问，在循环之前可能值得拷贝属性的值到本地变量，
然后在循环中只是引用本地变量。
* 在方法里边的嵌套代码块或循环中，定义变量是没有运行时开销的。
尽可能的声明为本地变量(尽可能使用小的作用域)通常有助于代码清晰，
甚至可以帮助编译器改进你的程序。

#### 1.3 字符串操作

* 不用通过重复的字符串连接来构建字符串。下面的循环在每次迭代会多花一倍的时间(较上次).
并且也很可能造成堆碎片化(见第二节):

``` java
String s = "";
for (int i=0; i<n; i++) {
  s += "#" + i;
}
```

应该使用StringBuilder对象和它的append方法。这样在每次迭代中的实现消耗都是线性的(较上次)，
可能有几个数量级速度提升。
``` java
StringBuilder sbuf = new StringBuilder();
for (int i=0; i<n; i++) {
  sbuf.append("#").append(i);
}
String s = sbuf.toString(); 
```

* 另一方面，一个包含一序列字符串连接的表达式会被自动编译成使用StringBuilder.append(...)的形式，所以也是可行的:
``` java
String s = "(" + x + ", " + y + ")";
```

* 不要通过重复查询或修改一个String或StringBuilder来处理字符串。
而重复使用String的substring和index方法，可能是合理的，但是应该用怀疑的眼光来看待。

**注: 这一段的翻译很纠结**

#### 1.4 数组中常量表的排序

* 在方法中声明一个初始化数组变量，在方法的每次执行中都会导致一个新数组被分配：

``` java
public static int monthdays(int y, int m) {
  int[] monthlengths = { 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };
  return m == 2 && leapyear(y) ? 29 : monthlengths[m-1];
}
```

一个初始化数组变量或者类似的表格应该仅仅声明和分配一次，并且在一个封闭的类中作为一个final static的属性存在：

``` java
private final static int[] monthlengths = { 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };
public static int monthdays(int y, int m) {
  return m == 2 && leapyear(y) ? 29 : monthlengths[m-1];
}
```

* 更复杂的初始化可以使用一个静态代码块static { ... }，例如来预计算一个数组的内容:

``` java
private final static double[] logFac = new double[100];
static {
  double logRes = 0.0;
  for (int i=1, stop=logFac.length; i<stop; i++)
    logFac[i] = logRes += Math.log(i);
}
public static double logBinom(int n, int k) {
  return logFac[n] - logFac[n-k] - logFac[k];
}
```

静态初始化块在这个封闭的类被加载的时候执行。
在这个例子中，它预先计算一个存放阶乘函数(n! = 1.2...(n-1)*n)的对数的表logFac。
所以方法logBinomial(n,k)可以有效的计算二项式系数的对数。作为实例，
从52张卡中选择7张卡，是Math.exp(logBinom(52, 7))，得到133784560。

#### 1.5 方法

* 声明方法为private,final或者static可以让调用更快。当然，只有应用需要时你才应该这么做。
* 作为例子，通常一个访问器方法如getSize，当子类无需覆盖它的时候，在类里边有理由作为final存在:

``` java
class Foo {
private int size;
...
public final int getSize() {
  return size;
}
}
```

这样可以让o.getSize()和直接访问公有属性o.size一样快。
做些适当的封装(让属性变成private)不会造成额外的性能影响。

* 虚方法调用(Virtual method calls) (调用实例方法)是很快的，应该用来替换instanceof测试和强制转换。
* 在现代java虚拟机实现中，像Sun的HotSpot JVM和IBM的JVM，接口方法调用和抽象方法调用实例方法是一样快的。
因此使用接口，而不是他们的实现类作为方法参数，这样的友好编程方式是没有额外的性能影响的。

#### 1.6 排序和搜索

* 永远不要使用选择排序、冒泡排序或插入排序，除非是一个非常小的数组或列表。
使用堆排序(对于数组)或合并排序(对于双向链表)或快速排序(对于数组，但是你需要找个好的参考值)
* 更好的选择是，使用能够保证很快的内置的排序例程:对于n个元素是O(nlog(n))，
当数据接近排序好的时候，有时候乐意更快:

对于数组，使用java.util.Arrays.sort，是一个改进过的快速排序；
它不需要额外的内存，但是它不是稳定的(不能保证相等对象的顺序)。
现在有所有原生类型和对象的重载版本。

对于ArrayList<T>和LinkedList<T>,他们都实现了接口java.util.List<T>,可以使用
java.util.Collections.sort来排序，它是稳定的(保证相等对象的顺序)和平滑的(对于接近排好序的列表接近线性时间)，
但是它使用了额外的内存。

* 避免在数组和列表里边做线性查询，除非你知道他们非常短。假如你的程序需要频繁的查找某些内容，
可以从下面的方法中选择:

-在排好序的数据上做两分搜索

对于数组，使用java.util.Arrays.binarySearch。数组必须是排好序的，例如经过了java.util.Arrays.sort.
已经有了所有原生类型和对象的重载版本。

对于ArrayList<T>,使用java.util.Collections.binarySearch.数组列表必须是有序的，
例如经过了java.util.Collections.sort.

假如你还需要从一个set或者map中插入或删除元素，可以使用下面的方法:

-哈希化(Hashing): 假如你的key对象有很好的hashCode方法，可以使用java.uitl包里边的HashSet<T>或HashMap<K, V>。
这适用于String和原生类型的包准类Interger,Double等等。

-两分搜索树(Binary search trees)：假如你的key对象有一个很好的比较方法compareTo的话，可以使用
java.util包里边的TreeSet<T>或TreeMap<K, V>.这同样适用于String和原生类型的包装类Integer,Double等等。

#### 1.7 异常

* 异常对象的构建new Exception(...)会构造一个调用栈轨迹(stack trace),需要消耗时间和空间，特别
是在递归方法调用的时候。Exception类或子类的对象的创建要比普通对象慢30~100倍。另外一方面，使用
try-catch快活着抛出一个异常是很快的。
* 像下面演示的，你可以通过在Exception子类中覆盖方法fillInStackTrace，来阻止一个栈轨迹的生成。
这可以让异常实例的创建要快上大约10倍。

``` java
class MyException extends Exception {
  public Throwable fillInStackTrace() {
    return this;
  }
}
```

* 因此，你应该在真的打算抛出异常的时候才创建一个异常对象。另外，不要使用异常来实现流程控制
(结束数据处理，跳出循环)；异常应该只是用来通知错误和异常情况(文件没有找到，非法的输入格式等等)。
假如你的程序真的需要非常频发的抛出异常，可以重用一个预先构造的异常对象。

#### 1.8 集合类

在包java.util.*的java集合类是设计良好并实现的。使用这些类可以很好的改进你的程序的速度，但是你需要一些陷阱.

* 如果你使用HashSet<T>或HashMap<K,V>，确保你的key对象有一个好而快的hashCode方法，并且它和equals方法保持一致。
* 如果你使用TreeSet<T>或TreeMap<K,V>, 确保你的key对象有一个好而快的compareTo方法，
或者提供一个Comparator<T>.但创建TreeSet<T>或TreeMap<K,V>的时候需要一个明确的Comparator<K>对象。
* 注意通过索引定位ListedList<T>不是一个常量时间操作。因此在下面的循环中，假如lst是一个LinkedList<T>的话，
在列表1st里边需要花费两倍的时间。(**注:翻译很纠结，不准**)，所以不应该使用:

``` java
int size = lst.size();
for (int i=0; i<size; i++)
  System.out.println(lst.get(i));
```

应该使用增强型for语句来迭代元素，它其实是使用集合的迭代器，所以遍历花费线性的时间:

``` java
for (T x : lst)
  System.out.println(x);
```

* 应该避免重复调用LinkedList<T>或ArrayList<T>的remove(Object o)方法，因为它使用顺序查找。
* 应该避免重复调用LinkedList<T>的add(int i, T x)或者remove(int i)方法，除非i是链表的最后或第一个。
这些方法都是使用顺序来查找第i个元素的。
* 应该避免重复调用ArrayList<T>的add(int i, T x)或remove(int i)方法，除非i是ArrayList<T>的最后一个。
它需要移动i后面的所有元素。
* 尽量避免使用传统的集合类，如Vector,HashTable和Stack，因为它们的方法都是synchronized的，
每一个集合方法的调用都有获取锁的运行时消耗。假如你真的需要一个同步的集合，请使用synchronizedCollection
和java.util.Collection类的类似方法来创建。
* 集合只能存放引用类型的数据，所以原生类型的值，例如int,double等等在集合中存储或作为key使用的时候，
必须包准为Integer,Double等。这需要花费时间和空间，在内存受限的嵌入式应用中可能是不可接受的。需要注意的是，
字符串和数组是引用类型数据，在使用时不需要被包装。

假如你需要有原生类型元素或key的集合，考虑使用Trove library,它提供了特殊处理的集合，像用int的哈希Set等等。
因此，他相对于通用的java集合类，它更快并且使用更少的内存。Trove可以在[http://trove4j.sourceforge.net](http://trove4j.sourceforge.net)中找到。

#### 1.9 输入和输出

* 使用带缓冲的输入和输出(包java.io里边的BufferedReader, BufferedWriter, BufferedInputStream, 
BufferedOutputStream)可以让输入/输出提速20倍。
* 使用包java.util.zip里边的压缩流ZipInputStream和ZipOutputStream或者
GZIPInputStream和GZIPOutputStream，可以加速冗长的数据格式如XML的输入输出。压缩和解压缩
需要CPU时间，但是压缩的数据可以比没压缩的数据小很多，从硬盘或网络中读取的数据更少，
因此无论如何还是节约了时间。同时它还节约了硬盘上的存储空间。

#### 1.10 空间和对象创建

* 假如你的程序使用过多的空间(内存),那么它也会使用过多的时间。对象分配和垃圾回收需要时间，
并且使用过多的内存导致糟糕的缓存利用率，甚至可能需要使用到虚拟内存(使用硬盘空间而不是RAM)。
而且，依赖于JVM的垃圾回收器，使用太多内存可能导致长时间的回收停顿。这会让交互系统变得恼人，
在实时系统更是难以接受(catastrophic).
* 对象创建需要时间(分配，初始化，垃圾回收)，所以不要在没必要的时候创建对象。
然而不要考虑对象池(在工厂方法里边)，除非真的有需要。
最有可能的是,你只会添加代码和维护问题，你的对象池在回收一个池中对象的时候，可能引发微妙的错误，
虽然它仍然被引用着并且被在程序的其他部分中被修改。
* 小心不要创建从来没有被使用的对象。例如，创建一个错误消息字符串，当从来没有被真正使用过，这就是一个典型的错误。
因为嵌入这个消息的异常被一个try-catch捕获之后，但它忽略了这个消息。
* GUI组件(通过AWT或Swing创建)可能要求更多的空间，并且可能没有被充分的回收。
不要创建你不一定需要的GUI组件。

#### 1.11 大数组操作

对于在数组上进行大量操作，有一些特别的方法。它们通常要比等同的for循环快很多，
在某种程度上是因为它们只需要一次边界检查。

* static void java.lang.System.arrayCopy(src, si, dst, di, n)会把数组片段src[si..si+n-1]
的元素拷贝到数组片段dst[di..di+n-1].
* static bool java.util.Arrays.equals(arr1, arr2)会返回true，当数组arr1和arr2有同样的长度
并且它们的元素成对相等(pairwise equal)。还有参数类型为boolean[], byte[], char[], 
double[], float[],int[], long[], Object[]和short[]的重载方法。
* static void java.util.Arrays.fill(arr, x)会把数组arr的所有元素设置为x.这个方法拥有和
Arrays.equals一样的重载方法。
* static void java.util.Arrays.fill(arr, i, j, x)会把元素arr[i..j-1]设置为x.这个方法拥有和
Arrays.equals一样的重载方法。
* static int java.util.Arrays.hashcode(arr)会返回一个由数组元素的hashcode计算出来
的数组的hashcode。这个方法拥有和Arrays.equals一样的重载方法。

#### 1.12 科学计算

假如你需要在java中使用科学计算，Colt开源库提供了许多高性能和高质量的例程(routine)，用来处理
线性代数、稀疏/稠密矩阵、数据分析统计工具、随机数生成器、数组算法、数学函数和复数。
如果你需要的已经在这里存在了，就不要在写一个新的低效的、不精确的数值例程了。Colt在
[http://hoschek.home.cern.ch/hoschek/colt/](http://hoschek.home.cern.ch/hoschek/colt/)可以找到。

#### 1.13 反射

* 反射方法调用，反射属性访问和反射对象创建(使用包java.lang.reflect)要比普通方法调用、属性访问、对象创建慢非常多。
* 访问检查会拖慢一些反射调用；部分消耗或许可以通过把调用方法声明为public进行避免。这可以将反射调用加速8倍。

#### 1.14 编译器和运行平台

* 正如上面提到的，许多C或者Fortran编译器可以做到的优化，java编译器做不到。另外一方面，在执行字节码的时候，
Java虚拟机(JVM)里边的即时编译器(JIT)可以做到传统编译器不能做到的许多优化。
* 例如，在cast(C)后面的一个测试条件(x instanceof C)可以被JVM优化，以致于最多只有一个测试会被执行。
所以重写你的程序来避免instanceof测试或者cast的麻烦事，是没有必要去做的。
* 许多不同的Java虚拟机(JVM)有着非常不同的特性:

-Sun的HotSpot Client JVM会做一些优化，但是通常优先于快速启动，而不是深度优化。

-Sun的HotSpot Server JVM(使用-server参数，在微软Windows不可用)会牺牲很长的启动时间来做非常深度的优化。

-相对于Sun的HotSpot Server JVM，IBM的JVM会做非常深度的优化。

-J2ME(移动手机)和PersonalJava(一些PDA)实现的JVM不会包括JIT编译器，很可能不会做任何的优化。
所以，在这种情况下，在你的java代码中尽可能地做自我优化就显得更加重要了。

-对于Oracle的JVM，Kaffe JVM,Intel的Open Runtime Platform,IBM的Jikes RVM等等，我不知道它们的优化特性。

你可以命令行提示符中敲入java -version来看看你正在使用的什么样的JVM。

#### 1.15 性能分析(Profiling)

假如一个java程序好像很慢，尝试对程序的运行进行性能分析。假设1.3节里边进行重复字符串连接的例子存放在文件MyExample.java
里边。可以使用Sun的HostSpot JVM来进行编译和性能分析,像下面这样:

``` bash
javac -g MyExample.java
java -Xprof MyExample 10000
```

性能分析的结果会展示在标准输出里边(控制台):

```
Flat profile of 19.00 secs (223 total ticks): main
  Interpreted + native Method
  1.3% 1      + 0      java.lang.AbstractStringBuilder.append
  1.3% 1      + 0      java.lang.String.<init>
  2.6% 2      + 0      Total interpreted

  Compiled + native Method
  51.3% 0  + 40     java.lang.AbstractStringBuilder.expandCapacity
  29.5% 23 + 0      java.lang.AbstractStringBuilder.append
  10.3% 8  + 0      java.lang.StringBuilder.toString
  6.4% 0   + 5      java.lang.String.<init>
  97.4% 31 + 45     Total compiled

  Thread-local ticks:
  65.0% 145 Blocked (of total)

Flat profile of 0.01 secs (1 total ticks): DestroyJavaVM
  Thread-local ticks:
  100.0% 1 Blocked (of total)

  Global summary of 19.01 seconds:
  100.0% 929 Received ticks
  74.6%  693 Received GC ticks
  0.8%     7 Other VM operations
```

它表示有51.%的计算时间花费在原生(native)方法expandCapacity,
而且有29.5%花费在方法append上，这些方法都是出自类AbstractStringBuilder的。
这看上去是String的+和=惹的祸，因为它被编译成append方法调用。

但是下面部分的内容更有意义，它指出总时间的74.6%花费在垃圾回收上，
因此只有25%的时间用于实际计算。这指出一个严重的问题:几乎马上会变成垃圾的数据分配得太多了。

### 2 降低空间消耗

* 在一个JVM中,数据是分配在调用栈(方法参数和本地变量)和堆(对象,包括字符串和数组)上的。
对于每个线程的执行都有隔离的栈，并且所有的线程使用共同的堆。一个线程的栈随着方法调用的
深度增长和收缩。对象，字符串和数组通过执行中的线程在堆中分配。他们的回收(垃圾回收)是通过
一个自发的垃圾回收器来处理的。
* 关于空间使用，这里有三个重要的方面: 分配率、保留区和碎片化:

-分配率是指你的程序创建对象、字符串、数组的频率。
一个高的分配率会消耗时间(分配，对象初始化和销毁)和空间(因为垃圾回收可能会为了效率的原因留出更多的内存)，
即使分配的数据只有非常短的生命周期。

–保留区(Retention)是指活动的堆数据的总量，也就是说，在任何时间点，调用栈可以企及的堆数据。
高的保留区消耗空间(明显地)和时间(垃圾回收器必需对分配和消耗做更多的管理工作).

–碎片化是碎片的产生:小的、不使用的内存块。持续更大对象的分配，例如增长字符串或者数组，
可能会引起内存碎片化，留下很多没法使用的小内存片。
这样的碎片化消耗时间(在分配的时候寻找一个足够大的hole)和空间(因为碎片变得没用)。
大多数垃圾回收器小心避免碎片化，但是它可能需要花费时间和空间，在嵌入式JVM实现中可能做不到。

* 空间泄露属于不需要或者意外的保留区(retention)，常常随着运行时间的增长引起内存消耗也直线上涨。
空间泄露是由活动变量的可企及的对象，字符串或数组引起的，而那些对象事实上不会再被使用到。
例如，假如你在HashMap中使用缓存计算结果，这就有可能出现:即使你不再需要这些计算结果，
但是她们在HashMap中还是可访问的。这可以通过使用WeakHashMap替代来避免这个问题。

* 一个深度尾递归(tail-recursive)方法可能造成空间泄露，所有应该写成一个循环。
但一个java编译器不会自动优化一个尾递归方法为一个循环，所以执行堆栈中可企及的所有数据将会被保留着，
直到方法返回。

* 垃圾回收器(通用的，标记-清除, 引用计数, two-space(注: 不知怎么翻译), 增量式, 压缩式 ...)的类型
很影响分配率(allocation rate), 保留区(retention)和碎片化(fragmentation)的时间和空间效果。
然而，一个起作用的垃圾回收器就其本身而言，从来不会引起空间泄露。空间泄露是由你程序里边的错误引起的。

* 确保一个类所有对象中共享的不变属性是static的。这样始终只有一个属性被创建。
当所有Car对象有同样的icon时，不要这么写:

``` java
public class Car {
  ImageIcon symbol = new ImageIcon("porsche.gif");
  ...
}
```

应该像这样:

``` java
public class Car {
  final static ImageIcon symbol = new ImageIcon("porsche.gif");
  ...
}
```

* 当你不确定是否需要真的需要一个对象的时候，可以延迟分配:把分配推迟到它真正需要的时候(只分配一次)。
因为像下面这样的话，将会对每一个Car对象无条件创建一个Button，而Button可能永远不会通过getButton调用来访问:

``` java
public class Car {
  private Button button = new JButton();
  public Car() {
    ... initialize button ...
  }
  public final JButton getButton() {
    return button;
  }
}
```

你可以在getButton中延迟分配Button:

``` java
public class Car {
  private Button button = null;
  public Car() { ... }
  public final JButton getButton() {
    if (button == null) { // button not yet created, so create it
      button = new JButton();
      ... initialize button ...
    }
    return button;
  }
}
```

这样会节省空间(Button对象)和时间(分配和初始化)。另一方面，假如button早知道是一定需要的话，
提前分配和初始化会更有效，可以避免getButton里边的判断(判空)。

### 3 其他资源

J.Noble和C.Weir的著作Small Memory Software,Addison-Wesley 2001,展示了一些受限内存系统的设计模式。
虽然不是所有建议都适用于java(例如它需要指示字运算(pointer arithmetics))，
即使有点模式说(pattern-speak)的问题，但是大多数还是有用的。

### 4 致谢

感谢Morten Larsen，Jyrki Katajainen和Eirik Maus提供的有用的建议。

 [1]: http://www.itu.dk/people/sestoft/papers/performance.pdf
