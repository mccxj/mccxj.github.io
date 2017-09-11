---
layout: post
comments: true
title: "AES256算法遇到 Illegal key size or default parameters的解决办法"
description: "AES256算法遇到 Illegal key size or default parameters的解决办法"
categories: ["aes256", "ibm", "jdk"]
---

```java
java.security.InvalidKeyException: Illegal key size or default parameters
	at javax.crypto.Cipher.a(DashoA13*..)
	at javax.crypto.Cipher.a(DashoA13*..)
	at javax.crypto.Cipher.a(DashoA13*..)
	at javax.crypto.Cipher.init(DashoA13*..)
	at javax.crypto.Cipher.init(DashoA13*..)
	at com.xxx.AESWithFileKey.encrypt(AESWithFileKey.java:81)
	at com.xxx.AESWithFileKey.main(AESWithFileKey.java:148)
```

Google到问题原因，链接地址如下：http://stackoverflow.com/questions/6481627/java-security-illegal-key-size-or-default-parameters

根据回答找到下载新jar包链接地址如下：http://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html  
把里面的两个jar包：local_policy.jar 和 US_export_policy.jar 替换掉原来安装目录C:\Program Files\Java\jre6\lib\security 下的两个jar包就可以了

上面是在oracle的jdk出现的。
如果是IBM的jdk出现问题，参考[调整 Version 7.0 应用程序的 Web Services Security](http://www-01.ibm.com/support/knowledgecenter/SSAW57_7.0.0/com.ibm.websphere.nd.multiplatform.doc/info/ae/ae/twbs_tunev6wss.html)