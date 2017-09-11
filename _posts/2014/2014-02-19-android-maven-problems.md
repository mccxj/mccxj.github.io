---
layout: post
comments: true
title: "android+maven问题记录"
description: "android+maven问题记录"
categories: ["android", "maven"]
---

## 参考材料

* https://code.google.com/p/maven-android-plugin/wiki/GettingStarted
* http://books.sonatype.com/mvnref-book/reference/android-dev.html
* http://www.ikoding.com/build-android-project-with-maven/
* https://github.com/mosabua/maven-android-sdk-deployer
* http://rgladwell.github.io/m2e-android/
* http://wiki.eclipse.org/M2E_plugin_execution_not_covered

## 前提条件

* JDK 1.6+
* Android SDK r21.1+
* Maven 3.1.1+
* Set environment variable ANDROID_HOME to the path of your installed Android SDK and add $ANDROID_HOME/tools as well as $ANDROID_HOME/platform-tools to your $PATH. (or on Windows %ANDROID_HOME%\tools and %ANDROID_HOME%\platform-tools)

特别注意maven的版本号

## maven配置

请设置环境变量M2_HOME，并把settings.xml放到M2_HOME/conf中去。

## eclipse配置

对于eclipse来说，除了要maven插件，还需要[m2e-android](http://rgladwell.github.io/m2e-android/)插件。

## dependency中support-v4的版本号只有很旧r7

其实除了support-v4,像android也有类似的问题。有一种解决方案是采用[maven-android-sdk-deployer](https://github.com/mosabua/maven-android-sdk-deployer)。
我测试过之后，发现这个解决方案虽然可行，但实际上比较麻烦。我直接在公司内的代理仓库上安装了新版本的。

## Plugin execution not covered by lifecycle configuration

pom.xml很可能出现下面的错误提示:

```bash
Plugin execution not covered by lifecycle configuration: 
 com.jayway.maven.plugins.android.generation2:android-maven-plugin:3.8.2:consume-aar 
 (execution: default-consume-aar, phase: compile)
```

虽然不影响编译，但是很怪，可以通过下面的配置进行排除:

```xml
<pluginManagement>
	<plugins>
		<plugin>
			<groupId>org.eclipse.m2e</groupId>
			<artifactId>lifecycle-mapping</artifactId>
			<version>1.0.0</version>
			<configuration>
				<lifecycleMappingMetadata>
					<pluginExecutions>
						<pluginExecution>
							<pluginExecutionFilter>
								<groupId>com.jayway.maven.plugins.android.generation2</groupId>
								<artifactId>android-maven-plugin</artifactId>
								<versionRange>3.8.2</versionRange>
								<goals>
									<goal>manifest-update</goal>
									<goal>generate-sources</goal>
									<goal>proguard</goal>
									<goal>consume-aar</goal>
								</goals>
							</pluginExecutionFilter>
							<action>
								<ignore />
							</action>
						</pluginExecution>
					</pluginExecutions>
				</lifecycleMappingMetadata>
			</configuration>
		</plugin>
	</plugins>
</pluginManagement>
```

可以参考[M2E_plugin_execution_not_covered](http://wiki.eclipse.org/M2E_plugin_execution_not_covered)

## OutOfMemory或创建不了虚拟机

有时候会出现内存溢出或创建不了虚拟机的错误。考虑设置内存大小

```xml
<plugin>
	<groupId>com.jayway.maven.plugins.android.generation2</groupId>
	<artifactId>android-maven-plugin</artifactId>
	<configuration>
		<dex>
			<jvmArguments>
				<jvmArgument>-Xms256m</jvmArgument>
				<jvmArgument>-Xmx512m</jvmArgument>
			</jvmArguments>
		</dex>
	</configuration>
</plugin>
```

## 出现maven打包太慢的情况
 
经过测量，在dex成classes.dex的阶段比较慢，dx工具有提供一些参数进行优化.
 
* incremental 增量打包，开发阶段可以开启，可以比较明显的缩短打包时间
* optimize 是否优化classes.dex，开发阶段可以关闭
 
```xml
<dex>
  <incremental>true</incremental>
  <optimize>false</optimize>
</dex>
```

## libpng error: Not a PNG file

如果直接把jpg格式换个名字，变成png，编译会报下面的错误，导致后面编译的.9图片也出问题(混淆问题的原因)

```bash
[INFO] libpng error: Not a PNG file
[INFO] ERROR: Failure processing PNG image E:\projects\G3ESOP\ESOP-Hubei2\res\drawable-xhdpi\more_about_pic1.png
```

## 'build.plugins.plugin.version' is missing

```bash
[WARN] 'build.plugins.plugin.version' is missing fororg.apache.maven.plugins:maven.compiler.plugin
It is highly recommended to fix these problems because they threaten the stability of your build.
For this reason, future Maven versions might no longer support building such malformed projects.
```

很简单，给maven.compiler.plugin这个插件添加version属性。
其实所有引用的插件都应该指定版本，不然都会有类似的提示。

## 关于编码

对于源码的编码格式和编译版本，应该进行指定:

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-compiler-plugin</artifactId>
	<version>3.1</version>
	<configuration>
		<source>1.6</source>
		<target>1.6</target>
		<encoding>UTF8</encoding>
	</configuration>
</plugin>
```

对于资源处理的话，可能出现下面的提示:

```bash
[WARNING] Using platform encoding (GBK actually) to copy filtered resources, i.e. build is platform dependent!
```

这个应该设置成UTF-8，如下所示:

```xml
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-resources-plugin</artifactId>
				<version>2.6</version>
				<configuration>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
```

## maven-jarsigner-plugin对带特殊字符的口令的处理

这个弄了好久，最后发现得把密码用引用引起来。切记切记。

后来，发现在linux这样又不能支持。所以只能用profile解决。

## 部分代码在jdk7中编译后dex出错

参考[jdk7编译的bug记录](/blog/20140225_android-jdk7-bug.html),暂时只用jdk6编译

## jdk6不支持android-19的proguard

原因是android-19的API实现了一些jdk7的特性，在proguard会找不到这些api。
由于和上一个问题有些冲突，暂时不考虑proguard。后续考虑考虑上jdk7。

## 如何添加.so支持

例如下面的百度地图SDK，需要加入一个so文件，在百度SDK里边是这样调用的：

```java
System.loadLibrary("BaiduMapSDK_v2_3_1");
```

如果要用maven集成的话，可以用下面的配置(已经部署到代理仓库):

```xml
<dependency>
	<groupId>com.baidu</groupId>
	<artifactId>libBaiduMapSDK_v2_3_1</artifactId>
	<version>2.3.1</version>
	<classifier>armeabi</classifier>
	<scope>runtime</scope>
	<type>so</type>
</dependency>
```

## 如何转换成eclipse项目

项目目录中只有pom.xml，如果要导入eclipse的话，可以考虑使用下面的命令生成.project和.classpath文件

```bash
mvn eclipse:eclipse
```

生成之后可能会有M2_REPO变量找不到的问题，可以在eclipse中通过window>Preferences>Maven>Installations>Add进行添加maven安装位置。

否则的话，可以按以下方法添加M2_REPO: Window > Preferences > Java > Build Path > Classpath Variables
新增一个M2_REPO变量指向你maven本地仓库。

## 常用命令

mvn clean package  
打包，但不部署。

mvn clean install  
打包，部署并运行。

mvn clean package android:redeploy android:run  
这个命令通常用于手机上已经安装了要部署的应用，但签名不同，所以我们打包的同时使用redeploy命令将现有应用删除并重新部署，最后使用run命令运行应用。

mvn android:redeploy android:run  
不打包，将已生成的包重新部署并运行。

mvn android:deploy android:run  
部署并运行已生成的包，与redeploy不同的是，deploy不会删除已有部署和应用数据