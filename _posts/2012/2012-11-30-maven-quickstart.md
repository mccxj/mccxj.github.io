---
layout: post
comments: true
title: "maven学习要点(旧)"
description: "以前写的关于maven的内容"
categories: ["实践", "maven"]
---

我感觉maven这东西比ant容易上手(ant每个项目规则都不一样),看那个100页的迷你书+例子(推荐basecrm或spring3.0)基本就ok了~~
哎，本来想写的很多，发现写出来的却很少。多多包涵!

* maven是一个项目管理工具，跟ant相比亮点在于它遵守了COC规则，只要遵守就好处多多
* maven可以分为三部分:maven本身(提供基本功能),仓库(提供jar),maven插件(提供更多的集成功能)
* maven的配置文件是settings.xml,maven项目的配置文件是pom.xml
* settings.xml主要是配置仓库镜像,本地仓库,网络访问设置等信息,常用配置:
{% highlight xml %}
 <localRepository>D:/repository</localRepository> --本地仓库
   <proxy>  --网络代理
     <id>optional</id>
     <active>true</active>
     <protocol>http</protocol>
     <username>proxyuser</username>
     <password>proxypass</password>
     <host>proxy.host.net</host>
     <port>80</port>
     <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
   </proxy>
  <mirror>  --仓库镜像
    <id>nexus</id>
    <mirrorOf>*</mirrorOf>
    <name>Mirror for maven central.</name>
    <url>http://10.137.27.223:8080/nexus/content/groups/public</url>
  </mirror>
{% endhighlight %}

* pom.xml主要分为三部分:项目本身的信息,使用的插件,依赖的jar
* 项目本身的信息,用于唯一标识(groupid-artifactId-version, 通常是这3部分)，例如
{% highlight xml %}
 <modelVersion>4.0.0</modelVersion> --maven3.x都是使用4.0.0
 <groupId>com.huawei.boss</groupId>
 <artifactId>boss-common</artifactId>
 <version>0.0.1-SNAPSHOT</version>
 <packaging>jar</packaging>
{% endhighlight %}
类似这个，默认就会被打包成boss-common-0.0.1-SNAPSHOT.jar。同样，项目的依赖库也是通过这种坐标在仓库中寻找的。 

* artifactId通常推荐使用项目名当前缀,当项目是打包成war的时候,通常需要制定发布后的命名:
{% highlight xml %}
 <build>
   <finalName>bossbase</finalName>
 </build>
{% endhighlight %}
* 可以通过properties定义一些常量,并通过${spring.version}之类来引用
{% highlight xml %}
 <properties>
       <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       <spring.version>2.5.6</spring.version>
 </properties>
{% endhighlight %}
* jar包依赖常用的有如下配置(仅举例说明):
{% highlight xml %}
 <dependencies>
     <dependency>
           <groupId>org.springframework</groupId>
               <artifactId>spring-core</artifactId>
             <version>${spring.version}</version>  --常量引用,在引用同个项目多个模块的时候相当有效
           <type>jar</type>  --默认就是jar
             <scope>compile</scope>  --默认是compile,表示对编译,测试,运行都有效
       </dependency>
    <dependency>
          <groupId>geronimo-spec</groupId>
            <artifactId>geronimo-spec-jta</artifactId> --这个是为了不依赖sun的专用东东
           <version>1.0.1B-rc4</version>
                <type>pom</type>
          <scope>provided</scope>  --在运行的时候不依赖，我们常见的时候j2ee规范之类的api,如servlet-api
  </dependency>
       <dependency>
             <groupId>junit</groupId>
               <artifactId>junit</artifactId>
           <version>4.8.2</version>
             <type>jar</type>
               <scope>test</scope>  --只在测试的时候生效,另外还有个ruuntime的scope,用于运行时依赖(如jdbc驱动等)
      </dependency>
   <dependency>
     <groupId>org.hibernate</groupId>
     <artifactId>hibernate</artifactId>
     <version>3.2.5.ga</version>
     <exclusions>  --这里是排除依赖，因为jta官方版是在maven仓库中找不到的
       <exclusion>
         <groupId>javax.transaction</groupId>
         <artifactId>jta</artifactId>
       </exclusion>
     </exclusions>
   </dependency>
 </dependencies>
{% endhighlight %}

* 当项目是分模块进行的时候,通常会考虑使用modules,如:
{% highlight xml %}
   <modules>
       <module>basecrm-parent</module>
       <module>common</module>
       <module>systemmgr</module>
       <module>prodmgr</module>
   </modules>
{% endhighlight %}
这个时候至少会有两级pom.xml,这样像properties,dependency等都是可以继承的(可参考basecrm项目或者spring3.x) 

* plugin资源非常多，简单举几个常用插件(配置请上网找)  
maven-compiler-plugin 主要是配置编译方面的选项(例如使用什么版本的jdk)  
maven-surefire-plugin 跟测试有关的,例如配置测试失败是否继续,是否跳过测试等  
build-helper-maven-plugin 用于配置项目的目录结构,例如配置多个源代码目录等  
maven-shade-plugin 我只知道可以用来给打包加点料~~  
tomcat-maven-plugin 顾名思义,用于集成tomcat  
maven-jetty-plugin 顾名思义,用于集成jetty  

* others   
maven仓库代理推荐nexus(网内已经有个现成的:http://10.137.27.223:8080/nexus)  
eclipse中的maven插件推荐m2eclipse(update-site: http://m2eclipse.sonatype.org/sites/m2e)  
maven实战迷你书(http://www.infoq.com/cn/minibooks/maven-in-action)  

