---
layout: post
comments: true
title: "初始化httpClient失败原因分析"
description: "httpClient使用的时候出现问题,看看分析"
categories: ["java", "httpclient", "jdk"]
---

## 问题描述

最近有个程序上线，启动失败，堆栈提示使用httpClient进行网络请求，初始化失败。具体如下:

使用httpClient进行网络请求，当使用IBM J9(JDK6实现)进行运行的时候，会有以下情况:

* 使用32位版本，在初始化DefaultHttpClient的时候出错，详细情况如下。
* 使用64位版本，可以正常启动。


测试代码如下:
```java
public class TestSSL
{
  public static void main(String[] args)
  {
    HttpClient httpClient = new DefaultHttpClient();
    HttpPost httpPost = new HttpPost("http://10.132.10.88:81/xxx/Receiver4XXX");
    httpPost.setHeader("contentType", "multipart/form-data");
    MultipartEntity reqEntity = new MultipartEntity();
    httpPost.setEntity(reqEntity);
    try
    {
      httpClient.execute(httpPost);
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}
```

启动命令如下:
```bash
java -Djava.ext.dirs=./lib/  TestSSL
```

出错堆栈如下:
```java
java.lang.IllegalStateException: Failure initializing default SSL context
        at org.apache.http.conn.ssl.SSLSocketFactory.createDefaultSSLContext(SSLSocketFactory.java:211)
        at org.apache.http.conn.ssl.SSLSocketFactory.<init>(SSLSocketFactory.java:333)
        at org.apache.http.conn.ssl.SSLSocketFactory.getSocketFactory(SSLSocketFactory.java:165)
        at org.apache.http.impl.conn.SchemeRegistryFactory.createDefault(SchemeRegistryFactory.java:45)
        at org.apache.http.impl.client.AbstractHttpClient.createClientConnectionManager(AbstractHttpClient.java:294)
        at org.apache.http.impl.client.AbstractHttpClient.getConnectionManager(AbstractHttpClient.java:445)
        at org.apache.http.impl.client.AbstractHttpClient.createHttpContext(AbstractHttpClient.java:274)
        at org.apache.http.impl.client.AbstractHttpClient.execute(AbstractHttpClient.java:797)
        at org.apache.http.impl.client.AbstractHttpClient.execute(AbstractHttpClient.java:754)
        at org.apache.http.impl.client.AbstractHttpClient.execute(AbstractHttpClient.java:732)
        at TestSSL.main(TestSSL.java:21)
Caused by: java.lang.NullPointerException
        at org.apache.harmony.security.fortress.Services$NormalServices.createDefaultProviderInstance(Services.java:286)
        at org.apache.harmony.security.fortress.Services$NormalServices.loadAllProviders(Services.java:218)
        at org.apache.harmony.security.fortress.Services$NormalServices.access$400(Services.java:141)
        at org.apache.harmony.security.fortress.Services$NormalServices$2.run(Services.java:207)
        at org.apache.harmony.security.fortress.Services$NormalServices$2.run(Services.java:205)
        at java.security.AccessController.doPrivileged(AccessController.java:202)
        at org.apache.harmony.security.fortress.Services$NormalServices.getProviderList(Services.java:205)
        at org.apache.harmony.security.fortress.Services$NormalServices.access$1300(Services.java:141)
        at org.apache.harmony.security.fortress.Services.getProvidersList(Services.java:645)
        at sun.security.jca.GetInstance.getProvidersList(GetInstance.java:79)
        at sun.security.jca.GetInstance.getInstance(GetInstance.java:232)
        at javax.net.ssl.KeyManagerFactory.getInstance(KeyManagerFactory.java:16)
        at org.apache.http.conn.ssl.SSLSocketFactory.createSSLContext(SSLSocketFactory.java:184)
        at org.apache.http.conn.ssl.SSLSocketFactory.createDefaultSSLContext(SSLSocketFactory.java:209)
        ... 10 more
```

分析有点冗长，所以分了几个阶段来说明。

## 问题分析(阶段1)

整理一下堆栈中各部分的功能:
```java
        at org.apache.harmony.security.fortress.Services$NormalServices.createDefaultProviderInstance(Services.java:286)
        at org.apache.harmony.security.fortress.Services$NormalServices.loadAllProviders(Services.java:218)
        at org.apache.harmony.security.fortress.Services$NormalServices.access$400(Services.java:141)
        at org.apache.harmony.security.fortress.Services$NormalServices$2.run(Services.java:207)
        at org.apache.harmony.security.fortress.Services$NormalServices$2.run(Services.java:205)
        at java.security.AccessController.doPrivileged(AccessController.java:202)
        at org.apache.harmony.security.fortress.Services$NormalServices.getProviderList(Services.java:205)
        at org.apache.harmony.security.fortress.Services$NormalServices.access$1300(Services.java:141)
        at org.apache.harmony.security.fortress.Services.getProvidersList(Services.java:645)   -- 类Services在JAVA_HOME/security.jar,尝试加载所有的密码算法提供类
        at sun.security.jca.GetInstance.getProvidersList(GetInstance.java:79)    
        at sun.security.jca.GetInstance.getInstance(GetInstance.java:232)          -- 类GetInstance在JAVA_HOME/rt.jar
        at javax.net.ssl.KeyManagerFactory.getInstance(KeyManagerFactory.java:16)   -- 类KeyManagerFactory在JAVA_HOME/ibmjssefw.jar.这里需要获取一个算法实现，具体算法是通过java.security的ssl.KeyManagerFactory.algorithm指定
        at org.apache.http.conn.ssl.SSLSocketFactory.createSSLContext(SSLSocketFactory.java:184)     -- 基于TLS/SSL协议，需要一个KeyManagerFactory来管理密钥
        at org.apache.http.conn.ssl.SSLSocketFactory.createDefaultSSLContext(SSLSocketFactory.java:209)   -- 默认会先注册http、https的处理类
```

问题就出现在加载密码算法提供类的过程，在java的安全体系中，这些提供类是通过JAVA_HOME/security/java.security这个配置文件指定的。

```bash
# 格式如下: security.provider.<n>=<className>, 序号代表优先级
security.provider.1=sun.security.provider.Sun
security.provider.2=sun.security.rsa.SunRsaSign
security.provider.3=com.sun.net.ssl.internal.ssl.Provider
security.provider.4=com.sun.crypto.provider.SunJCE
security.provider.5=sun.security.jgss.SunProvider
security.provider.6=com.sun.security.sasl.Provider
security.provider.7=org.jcp.xml.dsig.internal.dom.XMLDSigRI
security.provider.8=sun.security.smartcardio.SunPCSC
security.provider.9=sun.security.mscapi.SunMSCAPI
```

在IBM的实现中，会配合另外两个配置文件(在security.jar中的org.apache.harmony.security.fortress这个包里边): services.properties和providerClassName.properties。  
其中providerClassName.properties指定(提供者的标识, 实现类名)的对应关系。services.properties指定(算法,提供者的标识)的对应管理。

**这样就可以实现"寻找DES算法"，找到“提供者的标识", 最后找到"具体的实现类”，然后就可以调用了，整个过程对开发来说是透明的。**太具体的匹配逻辑就不说了，知道这点就可以了。

明显，加载这些类肯定用的是反射技术。不过从反编译的源码上看，IBM在具体的实现细节上有差异。

32位的NormalServices#createDefaultProviderInstance实现
```java
    private static Provider createDefaultProviderInstance(Services.ProviderInfo paramProviderInfo) {
      paramProviderInfo.setLoading();
      String str = paramProviderInfo.getProviderClassName();
      Provider localProvider = createProviderInstance(str, defaultNameProviderMap);

      for (Services.ProviderInfo localProviderInfo : defaultOrderedProviderInfoList) {
        if (localProviderInfo.getProviderClassName().equals(str)) {
          localProviderInfo.setProviderName(localProvider.getName());
          localProviderInfo.setLoaded();
        }
      }
      return localProvider;
    }
```

64位的NormalServices#createDefaultProviderInstance实现
```java
    private static Provider createDefaultProviderInstance(Services.ProviderInfo paramProviderInfo) {
      synchronized (Services.loadingAndRefreshLock) {
        if (paramProviderInfo.isLoaded()) {
          return (Provider)defaultNameProviderMap.get(paramProviderInfo.getProviderName());
        }
        paramProviderInfo.setLoading();
        String str = paramProviderInfo.getProviderClassName();
        Provider localProvider = createProviderInstance(str, defaultNameProviderMap);

        if (localProvider != null) {
          for (Services.ProviderInfo localProviderInfo : defaultOrderedProviderInfoList) {
            if (localProviderInfo.getProviderClassName().equals(str)) {
              localProviderInfo.setProviderName(localProvider.getName());
              localProviderInfo.setLoaded();
            }
          }
        }
        return localProvider;
      }
    }
```

其中createProviderInstance方法是通过classpath去加载类。**对于当前这个问题来说，最重要的区别在于localProvider是否有没有判空。因为一旦找不到提供类，localProvider将会为null。**
而对于目前32位的security.provider配置来说，下面两个类是放在JAVA_HOME/ext下面的:

* com.sun.security.sasl.Provider 在ibmsaslprovider.jar
* com.ibm.xml.enc.IBMXMLEncProvider 在ibmxmlencprovider.jar

**所以加载到这2个类的时候，localProvider会变成null，导致后面出现空指针。而64位只是忽略不加载而已。**

**尝试注释这两个security.provider，可以发现启动正常。**

## 问题分析(阶段2)

高大上的IBM JDK怎么会有这种问题呢? 再继续研究研究。

大家有没有注意到启动参数中有个 -Djava.ext.dirs=./lib/， 这个变量以前解释过.

大概就是说，设置classpath要一个个jar包都设置，实在麻烦，于是乎出现这个变量，大多数情况下的确很好很强大。  
在少数情况下，这个变量是可能有副作用的，上面提到的问题刚好就是一个例子，导致找不到提供类。

默认情况下，这个变量是指向JAVA_HOME/ext目录的，对应java中的扩展类加载器，使用这个变量就相当于覆盖了扩展类加载的路径。

**所以，有另外一种解决办法，就是把原来的扩展类路径添加上去，也是可以正常启动的。**
```bash
java -Djava.ext.dirs=/usr/java6/jre/lib/ext:./lib/  TestSSL
```

**其实这个变量还是比较少用的，大家可以看看其他比较有名的程序，他们的启动脚本都是读取某个目录的jar包，然后拼接成classpath再启动。**  
**直接指定目录，还有一个不好的地方就是，只要是jar格式的都会被加载(跟后缀名无关，很多人喜欢改名字进行备份的要注意了)**

## 问题分析(阶段3，可略过)

更多探讨，仅供有兴趣的童鞋参考

* 提供类com.ibm.crypto.provider.IBMJCE也在ext目录的ibmjceprovider.jar中，为什么没有报错

这个类很幸(悲)运(剧)的，因为有人在lib目录里边添加ibmjceprovider.jar这个包，所以它是能被加载到的。

具体可以添加-verbose:class参数，就可以看到的确是在ibmjceprovider.jar中加载到了。

```java
class load: java/util/jar/JarVerifier$VerifierStream
class load: com.ibm.crypto.provider.IBMJCE from: file:/home/hwcrm/caiqs/NGSENDWF/lib/ibmjceprovider.jar
class load: com.ibm.crypto.provider.f from: file:/home/hwcrm/caiqs/NGSENDWF/lib/ibmjceprovider.jar
class load: com/ibm/jsse2/IBMJSSEProvider2
```

* 在ext目录被加载很好理解，为什么其他ibm打头的类没在ext中也能被找到

在jre/lib/目录下面有个jars.cfg的配置，个人认为是IBM自己的特殊处理逻辑来的(未经证实)，我稍微加了点中文注释，大家可以看看。

```bash
# j2se的api被拆开很多个包进行开发，默认的rt.jar是会加载的(可以发现没有常见的集合类，sql类等)，欠缺的部分使用Harmony引入(不知大家有没有对印象，曾经的jdk开源实现)
# add Harmony jars 
annotation.jar
beans.jar
java.util.jar
jndi.jar
logging.jar
security.jar
sql.jar

# ORB，就是用于COBRA的开发api
# jars for the IBM ORB
# these must precede rt.jar unless we know that 
# the Sun ORB API has been removed from rt.jar 
ibmorb.jar
ibmorbapi.jar
ibmcfw.jar

# 大家可以发现很多ibm打头的包，这些就是提供类了，看下面的注释，可以发现这些类都是在启动的时候加载的，这样就可以被加载到了。
# List of bootclasspath jars, ordered, relative to jre/lib/
rt.jar
charsets.jar
resources.jar
ibmpkcs.jar
ibmcertpathfw.jar
ibmjgssfw.jar
ibmjssefw.jar
ibmsaslfw.jar
ibmjcefw.jar
ibmjgssprovider.jar
ibmjsseprovider2.jar
ibmcertpathprovider.jar
ibmxmlcrypto.jar
management-agent.jar
xml.jar
jlm.jar
javascript.jar
```

* 一定是httpclient引起的么? 会影响其他库或代码么?

从IBM的实现上看，只要使用到security.provider，都会出错。
例如只是简单的使用内置的DES算法实现，就像下面那样，同样也是会出错的。有兴趣可以试试。

```java
Cipher.getInstance("DES");
```

但是，对于其他JDK实现，例如oracle JDK，不一定有问题(测试一下，发现的确也是没问题的)。因为jdk api是标准规范，当具体的实现并没有做要求。

## 问题总结

* IBM JDK对security.provider的处理有所不同，对于找不到的提供类，可能报错也可能不报错。目前只是一个特例，不代表在其他版本或其他JDK中存在。
* 正常情况下(除非添加自己的实现或修改配置)，security.provider都是能够被加载到的。
* 使用-Djava.ext.dirs会修改扩展类加载路径，可能导致某些提供类找不到。
* 有以下方式可以修复，仅供参考:

1. 对httpclient进行定制，跳过https注册或自定义实现，如目前的规避代码。缺点在于仅仅是规避，对其他库可能不适用。
2. 通过classpath代替java.ext.dirs变量，这是标准的启动方式。缺点在于脚本要重写。
3. 在java.ext.dirs添加原来的ext目录，是一个方便又能解决问题的手段。缺点在于需要修改脚本, 并可能由于备份文件被加载而造成混乱。
4. 通过修改java.security配置文件，屏蔽没法使用的提供类。不推荐，影响全局。
5. 把ext中相关的jar包拷贝到lib目录中。目前有个jar包是这样的，不过不推荐，在不理解系统加载机制的情况下，很容易造成混乱。
6. 给IBM提意见，修改一下实现方式。说说而已，别想了。

