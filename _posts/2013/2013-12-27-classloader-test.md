---
layout: post
comments: true
title: "考察对类加载的理解(问题篇)"
categories: ["java", "classloader"]
---

类加载和程序运行是有些关系的，不妨来测试一下。  
难度:中级

## 独立进程篇

假设有下面的类文件：

```java
// Main.java
package com.github.mccxj.test;

public class Main {
  public static void main(String[] args){
    new TestServlet().test();
  }
}

// TestServlet.Java
package com.github.mccxj.test;

public class TestServlet { 
  public void test() {
    InputStream is = TestServlet.class.getClassLoader().getResourceAsStream("config.properties");
    if(is == null){
      throw new RuntimeException("couId not found config.properties");
    }
  }
}
```
假设目录结构是这样的，其中jar下面的表示是在jar包里边的内容:

``` bash
test
    -lib
        -test.jar
          -com/github/mccxj/test/Main.class
    -main.jar
        -com/github/mccxj/test/TestServlet.class
        -config.properties
```

请问：

1. 执行java -cp main.jar;lib/test.jar com.github.mccxj.test.Main会出错么？  
2. 执行java -cp main.jar -Djava.ext.dirs=./lib com.github.mccxj.test.Main结果是怎样？

继续调整目录结果如下：

``` bash
test
    -lib
        -test.jar
            -com/github/mccxj/test/Main.class
        -main.jar
            -com/github/mccxj/test/TestServlet.class
            -config.properties
```

再请问

3. 执行java -Djava.ext.dirs=./lib com.github.mccxj.test.Main结果是怎样？                                                                                    
                                                                                                                                            
继续调整一下TestServlet的代码：

```diff
// TestServlet.Java
package com.github.mccxj.test;

public class TestServlet { 
  public void test() {
-    InputStream is = TestServlet.class.getClassLoader().getResourceAsStream("config.properties");
+    InputStream is = Thread.currentThread().getContextClassLoader().getResourceAsStream("config.properties");
    if(is == null){
      throw new RuntimeException("couId not found config.properties");
    }
  }
}
```

把目录结构恢复成：

``` bash
test
    -lib
        -test.jar
          -com/github/mccxj/test/Main.class
    -main.jar
        -com/github/mccxj/test/TestServlet.class
        -config.properties
```

请问:

4. 执行java -cp	main.jar;lib/test.jar com.github.mccxj.test.Main会出错么？  
5. 执行java	-cp main.jar -Djava.ext.dirs=./lib com.github.mccxj.test.Main结果又是怎样？

最后调整目录结果如下：

``` bash
test
    -lib
        -test.jar
            -com/github/mccxj/test/Main.class
        -main.jar
            -com/github/mccxj/test/TestServlet.class
            -config.properties
```

6. 执行java -Djava.ext.dirs=./lib com.github.mccxj.test.Main结果是怎样？

## Web应用服务器篇

下面的例子，以tomcat为例。
假设有下面的Servlet文件，并打包成test.jar:

```java
// TestServlet.java
package com.github.mccxj.test;

public class TestServlet extends HttpServlet {
    private static Atomiclnteger al = new AtomicInteger();
    private Atomiclnteger a2 = new AtomicInteger();

    @Override
    public void service(ServletRequest arg0, ServletResponse arg1) throws Servlet Exception, IOException {
        System.out.printIn(String.valueOf(al.incrementAndGet()));
        System.out.printIn(String.valueOf(a2.incrementAndGet()));
    }
}
```

并部署两个应用程序appa、appb,在他们的WEB_INF/web.xml添加了下面的内容

```xml
<servlet>
  <servlet-name>test</servlet-name>
  <display-name>test servlet</display-name>
  <servlet-class>com.huawei.test.TestServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>test</servlet-name>
  <url-pattern>/test</url-pattern>
</servlet-mapping>
```

大家应该听说过Servlet是单例的概念，也可能听过Web应用服务器有共享类的机制。那么，请问：

1. 现在把test.jar扔到appa和appb的WEB_INF/lib目录中，启动tomcat，先访问/appa/test两次，再访问/appb/test, 此时会输出什么？  
2. 继续把test.jar都移除掉，只添加到TOMCAT_HOME/lib目录中，启动tomcat，先访问/appa/test两次，再访问/appb/test, 此时会输出什么？  
3. 最后把test.jar拷贝一份到appa的WEB_INF/lib目录中，启动tomcat，先访问/appa/test两次，再访问/appb/test, 此时会输出什么？  