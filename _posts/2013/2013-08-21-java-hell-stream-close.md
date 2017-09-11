---
layout: post
comments: true
title: "从流关闭说起"
categories: ["java hell", "stream", "java"]
---

### 基本原则: 谁生产谁销毁

这个是用来解决责任权的问题，例如你的方法接收一个InputStream作为参数，通常就不应该在方法内去关闭它，而由客户端代码去处理。如果要关闭，通常应该在方法签名上明确说明，具体样例参考commons-io的IOUtils类。

还有另外一个例子，就是经常使用的Servlet的输入输出流，根据这个原则，也是不应该在代码中进行关闭的，这个工作是由Web容器负责的。

### 关闭的是什么?

java本身是带GC的，所以对象在消除引用之后，按正常是能够被回收的，那么为什么会有关闭操作?

这是为了回收系统资源，主要是端口(网络IO),文件句柄(输入输出)等，通常涉及的场景也是操作文件，网络操作、数据库应用等。对于类unix系统，所有东西都是抽象成文件的，所以可以通过lsof来观察。

为了更详细的说明这点，我们可以测试一下下面的代码:

```java
public class Hello {
    public static void main(String[] args) throws Exception {
        for (int i = 0; i < 100; i++) {
            FileInputStream is = new FileInputStream("/tmp/data/data"+i);
            byte[] bs = new byte[512];
            int len;
            while ((len = is.read(bs)) != -1) ;
            Thread.sleep(1000L);
        }
        Thread.sleep(10000L);
    }
}
```

运行之后，通过losf或进程目录查看相关的文件句柄数量是不断增长的:

```bash
lsof -p 25945 |grep /tmp/data | wc -l
88

ls  /proc/25945/fd | wc -l
93
```

如果有关闭操作的话，就会发现打开文件数一直都处于很低的位置。如果持续出现未关闭的情况，积累到一定程度就可能超过系统限制，出现too many open files的错误。

### 如何确保关闭

关闭通常是调用close()方法来实现的，并且需要在finally来进行处理。另外，我们经常会遇到多个资源的关闭情况，因为IO库是采用修饰器模式的，所以基本原则是先关闭外层对象，再关闭内层对象，每个close调用都需要处理异常情况，例如:

```java
InputStream is = null;
OutputStream os = null;
try{
   // ...
}
finally{
  if(is != null)
     try{
       is.close();
     }
     catch(IOException e){}

  if(os != null)
     try{
       os.close()
     }
     catch(IOException e){}
}
```

### 实践

* 上面的关闭处理的确是比较繁琐的，可以考虑进行封装或者直接使用IOUtils.closeQuietly方法，节约不少代码行。
* 自从JDK5之后，需要进行关闭处理的对象可以考虑实现java.io.Closeable接口。这个可以让资源关闭使用同一套代码。

### JDK7改进及其他思路

在JDK7里，你只需要将资源定义在try()里，Java7就会在readLine抛异常时，自动关闭资源。
但是资源类必须实现java.lang.AutoCloseable接口，同时支持管理多个资源,释放次序与声明次序相反。

```java
try (BufferedReader br = new BufferedReader(new FileReader(path)) {
    return br.readLine();
}
```

虽然感觉总是很繁琐，语法糖味道重，但比以前倒是进步不少了。
不过我们还是来看看Go中的做法，它提供了defer操作，用于在脱离方法作用域的时候自动调用，有点析构的味道。
看下面的示例:

```go
func main() {
    files, err := os.Open("testqq.txt")        
    if err != nil {
        fmt.Printf("Error is:%s", "Game Over!")
        return
    }
    defer files.Close()
}
```
