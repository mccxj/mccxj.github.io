---
layout: post
comments: true
title: "reflection in java"
description: "java反射方面的讲义"
categories: ["培训", "反射"]
---

## 什么是反射
Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；
对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取信息以及动态调用
对象方法的功能成为java语言的反射机制。

## 典型应用
* spring生成Bean对象
* struts2使用配置进行对象校验
* hibernate,ibatis进行数据库对象映射
* xml与java对象的相互转化
* springmvc进行方法参数注入
* more......

## 理解类型

### 原型类型vs包装类型(XXX.TYPE)
{% highlight java %}
Void -> void -> V
Byte -> byte -> B
Character -> char -> C
Short -> short -> S
Integer -> int -> I
Long -> long -> J
Float -> float -> F
Double -> double -> D
Boolean -> boolean -> Z
{% endhighlight %}

### 类与接口

* 接口(Interface)是一种特殊的类
* 注解(Annotation)是一种特殊的接口
* 枚举(Enum)是一种特殊的类

{% highlight java %}
class java.lang.String //类
class java.util.AbstractList //抽象类
interface java.lang.Runnable //接口
{% endhighlight %}

### 数组类型
{% highlight java %}
//基本类型数组
int[].class -> [I
int[][].class -> [[I

//普通类型数组
String[].class -> [Ljava.lang.String;
Integer[][].class -> [[Ljava.lang.Integer;
{% endhighlight %}

## 理解类型判断方法
{% highlight java %}
//是否某个类型的实例
i instanceof Integer
Integer.class.isInstance(i)
Person.class.isInstance(new SuperTeacher(""))

//是否数组
x.getClass().isArray()

//是否父子类/接口关系
Number.class.isAssignableFrom(Integer.class)

//是否原型类型
i.getClass().isPrimitive()
Integer.class.isPrimitive()
Integer.TYPE.isPrimitive()

//是否注解
Logger.class.isAnnotation()

//是否枚举
Status.class.isEnum()

//是否接口
List.class.isInterface()
{% endhighlight %}

## 理解修饰符
* 手段:Class#getModifiers()和Modifier工具类
* 适用于Class,Method,Field,Constructor

示例代码如下:
{% highlight java %}
int mod = Teacher.class.getModifiers();
assertTrue(Modifier.isPublic(mod));
{% endhighlight %}
常见的修饰符判断方法有:
{% highlight java %}
//可见性
isPublic,isProtected,isPrivate
//类型属性相关
isInterface,isAbstract,isFinal,isStatic
//特殊作用
isSynchronized,isNative,isTransient,isVolatile
{% endhighlight %}

## 理解类型关系
{% highlight java %}
//类->父类(单继承,只有一个)
Class#getSuperclass()

//类、接口->实现接口(多继承,可能有多个)
Class#getInterfaces()

//类->内部类(可能有多个)
Class#getDeclaredClasses()

//内部类->外部类(只有一个)
Class#getEnclosingClass()
{% endhighlight %}

### 相关练习
* Test:输出某个类实现的所有接口
* Test:输出某个类实现的继承结构
* Test:能够获取到之类的信息么?

## 理解对象生成与调用

### 对象生成
可以采用无参数构造方法或有参数方式
{% highlight java %}
//需要存在无参构造方法
String s = "com.xxx.Student";
Class t = Class.forName(s);
Teacher teacher = (Teacher)t.newInstance();

//直接调用构造方法
Constructor[] cons = SuperTeacher.class.getConstructors();
Object obj = cons[0].newInstance("hello");
{% endhighlight %}

### 获取成员
成员一般指Field,Method,Constructor等信息
{% highlight java %}
//属性
Field[] fs = Teacher.class.getDeclaredFields();
Field[] fs = Teacher.class.getFields();//只有公有的

//方法
Medhod[] ms = Teacher.class.getDeclaredMethods();
Medhod[] ms = Teacher.class.getdMethods();//只有公有的

//构造方法
Constructor[] cs = Teacher.class.getDeclaredConstructors();
Constructor[] cs = Teacher.class.getConstructors();//只有公有的
{% endhighlight %}

获取成员可以通过参数直接定位,以属性和方法为例:
{% highlight java %}
SuperTeacher.class.getField("name");

SuperTeacher.class.getMethod("setBounds", Integer.TYPE);//参数是int
SuperTeacher.class.getMethod("setBounds", Integer.class);//参数是Integer
{% endhighlight %}

相关练习
* Test:找到某个类所有的非private的普通方法

### 调用方式
拿到成员之后，就可以进行调用了。像Field需要使用get/set方法，
Method使用invoke方法，Constructor使用newInstance方法。如：
{% highlight java %}
Field#set(obj);
Field#get(obj);

Method#invoke(obj, args);
{% endhighlight %}

有时候需要绕过可见性进行调用，需要通过setAccessible方法进行处理，如
{% highlight java %}
Field#setAccessible(true);// if not set, fail
{% endhighlight %}

对于参数数组的话，通过args并不能区分是数组类型还是不定参数类型(都是通过数组进行传递)。
如果需要继续区别，应该对Method进行检测，检测方式是Method#isVarArgs

### 处理返回值
对于方法调用，可以对返回值进行处理。即使返回为void或数组类型也是可以识别的。如:
{% highlight java %}
Class<?> returnType = method.getReturnType();
assertEquals(void.class, returnType);
assertTrue(returnType.isArray());
{% endhighlight %}
如果是数组的话，由于返回值是Object类型，需要通过Array工具类进行处理(见java api文档)

### 其他:关于内部类调用的秘密

#### 普通内部类
{% highlight java %}
public class A{
    private List stus;
    public int test() { return new B().test(); }
    class B{
        public int test() { return stus.size(); }
    }
}
{% endhighlight %}
关于上面的内部类，为什么B能够调用到A的stus属性，通过观察生成的字节码，得到的实际结果是:
{% highlight java %}
public class A{
    private List stus;
    static List access$0(A a) { return a.stus; }
    public int test() { return new A$B(this).test(); }
}

class A$B{
    A this$0;
    A$B(A a) { this$0 = a; }
    public int test() { return A.access$0(this$0),size(); }
}
{% endhighlight %}
我们可以发现:

1. A对B类需要访问的每一个私有属性生成对应的一个静态访问方法,B的访问就是通过这个私有方法的
2. 生成B类实例的时候把A类实例引用传入并进行保存,所以每一个B类实例都有一个A类实例关联
3. 通过反射可以看到新方法access$0和新内部变量this$0

#### 静态内部类
{% highlight java %}
public class A{
    private static List stus;
    public int test() { return new B().test(); }
    static class B{
        public int test() { return stus.size(); }
    }
}
{% endhighlight %}
对于静态内部类，情况就比较简单，得到的实际结果是:
{% highlight java %}
public class A{
    private static List stus;
    static List access$0() { return stus; }
    public int test() { return new A$B().test(); }
}

class A$B{
    A$B() {}
    public int test() { return A.access$0(),size(); }
}
{% endhighlight %}
我们可以发现:

1. A对B类需要访问的静态私有属性生成一个静态访问方法,B的访问就是通过这个私有方法的
2. 由于是static方法，所以不需要保存实例的引用
3. 通过反射可以看到新方法access$0

所以，如果可以的话，优先使用静态内部类(特别是没有使用到内部普通变量的情况)。

## 理解泛型
java的泛型是擦除式,运行时没有泛型的概念.所以下面的代码会编译通过但执行报类型转换错误:
{% highlight java %}
List<String> strs = new Array<String>();
add2List(strs);
String str = strs.get(0);

//add2List的实现
void add2List(List lt){
    lt.add(1);
}
{% endhighlight %}

通过javap -c Test.class可以看到类似下面的指令:
{% highlight java %}
// String str = strs.get(0);
7: invokeinterface #31, 2; //interfaceMethod java/util/List.get:(l)Ljava/lang/Object;
12: checkcast #37; //class java/lang/String

// lt.add(1);
2: invokestatic #41; //Method java/lang/Integer.valueOf:(l)Ljava/lang/Integer;
5: invokeinterface #47, 2; //InterfaceMethod java/util/List.add:(Ljava/lang/Object;)Z
{% endhighlight %}
很明显，在checkcast的时候会出错。

虽然说运行时不关注是否泛型，但是还是有办法可以找到一些泛型相关信息的。和泛型类型的类图如下面所示：
![java泛型类型结构][1]

下面以Method为例进行描述(关于field之类的就不描述了)：
{% highlight java %}
public T method2(List<T> a, List<String> b, List<?> c, float[] f) throws Exception
{% endhighlight %}

### 分析入口
{% highlight java %}
Method#getGenericParameterTypes()
Method#getGenericReturnType()
Method#getGenericExceptionTypes()
{% endhighlight %}

通过上面一些方法可以得到下面的类型关系：
{% highlight java %}
T -> TypeVariable
List<String> -> ParameterizedType
List<?> -> ParameterizedType
float[] -> GenericArrayType
Exception -> Class
{% endhighlight %}

ParameterizedType描述的是一种多级泛型的关系，可以继续通过getActualTypeArguments继续进行处理,可以得到
{% highlight java %}
List<String> -> Class
List<?> -> WildcardType
List<T> -> TypeVariable
{% endhighlight %}

处理Type是比较麻烦的事情，每种具体类型都有自己特殊的处理方法(见图所示)，但平时这个东西很少用到。

## New in JDK7
jdk7以前有4种方法调用指令:
{% highlight java %}
invokestatic -> 静态方法调用
invokespecial -> 特殊方法(如构造方法)调用
invokevirtual -> 普通方法调用
invokeinterface -> 接口方法调用
{% endhighlight %}

在jdk7中实现了新的规范jsr292，增加了新的字节码指令invokedynamic,通过下面的选项可以开启。
{% highlight java %}
XX:+UnlockExperimentalVMOptions
XX:+EnableInvokeDynamic
XX:+EnableMethodHandles
{% endhighlight %}

该指令只需要制定方法名称，只要求引用非空对象而已，但javac还没法生成这样的指令。调用方式如下:
{% highlight java %}
invokedynamic #10; //DynamicMethod java/lang/Object.lessThan:(Ljava/lang/Object;)
{% endhighlight %}

但是可以有相关的api可以模拟出这种指令调用方式，如下(看上去和普通反射调用没什么明显不同)：
{% highlight java %}
MethodType type = MethodType.methodType(int.class, int.class, int.class);
MutableCallSite callSite = new MutableCallSite(type);
MethodHandle invoker = callSite.dynamicInvoker();

MethodHandles.Lookup lookup = MethodHandles.lookup();
MethodHandle mhMax = lookup.findStatic(Math.class, "max", type);

callSite.setTarget(mhMax);
invoker.invokeWithArguments(3,5);
{% endhighlight %}

这种调用方式有几个关键的概念:
{% highlight java %}
method handle 方法句柄，是callsite的目标
bootstrap method 引导方法
CallSite 调用点
method type 方法类型
{% endhighlight %}

这个指令可以增强很多基于JVM的动态语言的方法调用性能，所以潜力很大，更多具体信息，可以参考下面的一些链接:
* http://www.infoq.com/cn/news/2011/01/invokedynamic
* http://icyfenix.iteye.com/blog/1392441
* http://book.51cto.com/art/201205/339215.htm
* http://www.from0to1.net/%E5%86%8D%E6%8E%A2-invokedynamic/

 [1]: /assets/images/java_types.png

