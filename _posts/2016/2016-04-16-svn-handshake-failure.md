---
layout: post
comments: true
title: "svn报文转发出现handshake_failure的问题"
description: "svn报文转发出现handshake_failure的问题"
categories: ["ssl", "svnkit", "handshake_failure"]
---

## 问题描述

昨天svn报文转发xx程序突然出现连不上svn的情况，报错信息如下:

```text
org.tmatesoft.svn.core.SVNException: svn: E175002: Connection has been shutdown: javax.net.ssl.SSLHandshakeException: Received fatal alert: handshake_failure
svn: E175002: OPTIONS request failed on '/XXXX'
	at org.tmatesoft.svn.core.internal.wc.SVNErrorManager.error(SVNErrorManager.java:106)
	at org.tmatesoft.svn.core.internal.wc.SVNErrorManager.error(SVNErrorManager.java:90)
	at org.tmatesoft.svn.core.internal.io.dav.http.HTTPConnection.request(HTTPConnection.java:798)
	at org.tmatesoft.svn.core.internal.io.dav.http.HTTPConnection.request(HTTPConnection.java:398)
	at org.tmatesoft.svn.core.internal.io.dav.http.HTTPConnection.request(HTTPConnection.java:386)
	at org.tmatesoft.svn.core.internal.io.dav.DAVConnection.performHttpRequest(DAVConnection.java:863)
	at org.tmatesoft.svn.core.internal.io.dav.DAVConnection.exchangeCapabilities(DAVConnection.java:699)
	at org.tmatesoft.svn.core.internal.io.dav.DAVConnection.open(DAVConnection.java:118)
	at org.tmatesoft.svn.core.internal.io.dav.DAVRepository.openConnection(DAVRepository.java:1049)
	at org.tmatesoft.svn.core.internal.io.dav.DAVRepository.testConnection(DAVRepository.java:100)
	...
	at java.lang.Thread.run(Thread.java:744)
Caused by: javax.net.ssl.SSLException: Connection has been shutdown: javax.net.ssl.SSLHandshakeException: Received fatal alert: handshake_failure
	at sun.security.ssl.SSLSocketImpl.checkEOF(SSLSocketImpl.java:1476)
	at sun.security.ssl.SSLSocketImpl.checkWrite(SSLSocketImpl.java:1488)
	at sun.security.ssl.AppOutputStream.write(AppOutputStream.java:70)
	at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
	at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
	at org.tmatesoft.svn.core.internal.io.dav.http.HTTPConnection.sendData(HTTPConnection.java:340)
	at org.tmatesoft.svn.core.internal.io.dav.http.HTTPRequest.dispatch(HTTPRequest.java:170)
	at org.tmatesoft.svn.core.internal.io.dav.http.HTTPConnection.request(HTTPConnection.java:497)
	... 14 more
```

## 初步检查

* 检查了svn用户名密码是正常的
* 检查了代码也没有变化，java版本也没有变更
* 初步怀疑是svn服务器做了配置更新，禁用了某些东西，由于是https，尝试开启debug信息看看

## 调试过程

通过设置vm参数javax.net.debug=all开启网络相关的全部调试信息，可以看到:

```text
*** ClientHello, SSLv3
RandomCookie:  GMT: 1460716796 bytes = { 214, 15, 157, 159, 83, 144, 248, 115, 164, 210, 12, 247, 143, 96, 117, 244, 202, 251, 111, 187, 109, 171, 81, 216, 101, 89, 33, 240 }
Session ID:  {}
Cipher Suites: [TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA, TLS_RSA_WITH_AES_128_CBC_SHA, TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA, TLS_ECDH_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_RSA_WITH_AES_128_CBC_SHA, TLS_DHE_DSS_WITH_AES_128_CBC_SHA, TLS_ECDHE_ECDSA_WITH_RC4_128_SHA, TLS_ECDHE_RSA_WITH_RC4_128_SHA, SSL_RSA_WITH_RC4_128_SHA, TLS_ECDH_ECDSA_WITH_RC4_128_SHA, TLS_ECDH_RSA_WITH_RC4_128_SHA, TLS_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_RSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA, TLS_ECDH_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA, SSL_RSA_WITH_RC4_128_MD5, TLS_EMPTY_RENEGOTIATION_INFO_SCSV]
Compression Methods:  { 0 }
Extension elliptic_curves, curve names: {secp256r1, sect163k1, sect163r2, secp192r1, secp224r1, sect233k1, sect233r1, sect283k1, sect283r1, secp384r1, sect409k1, sect409r1, secp521r1, sect571k1, sect571r1, secp160k1, secp160r1, secp160r2, sect163r1, secp192k1, sect193r1, sect193r2, secp224k1, sect239k1, secp256k1}
Extension ec_point_formats, formats: [uncompressed]
***
[write] MD5 and SHA1 hashes:  len = 149
0000: 01 00 00 91 03 00 57 11   C5 FC D6 0F 9D 9F 53 90  ......W.......S.
0010: F8 73 A4 D2 0C F7 8F 60   75 F4 CA FB 6F BB 6D AB  .s.....`u...o.m.
0020: 51 D8 65 59 21 F0 00 00   2A C0 09 C0 13 00 2F C0  Q.eY!...*...../.
0030: 04 C0 0E 00 33 00 32 C0   07 C0 11 00 05 C0 02 C0  ....3.2.........
0040: 0C C0 08 C0 12 00 0A C0   03 C0 0D 00 16 00 13 00  ................
0050: 04 00 FF 01 00 00 3E 00   0A 00 34 00 32 00 17 00  ......>...4.2...
0060: 01 00 03 00 13 00 15 00   06 00 07 00 09 00 0A 00  ................
0070: 18 00 0B 00 0C 00 19 00   0D 00 0E 00 0F 00 10 00  ................
0080: 11 00 02 00 12 00 04 00   05 00 14 00 08 00 16 00  ................
0090: 0B 00 02 01 00                                     .....
Thread-0, WRITE: SSLv3 Handshake, length = 149
[Raw write]: length = 154
0000: 16 03 00 00 95 01 00 00   91 03 00 57 11 C5 FC D6  ...........W....
0010: 0F 9D 9F 53 90 F8 73 A4   D2 0C F7 8F 60 75 F4 CA  ...S..s.....`u..
0020: FB 6F BB 6D AB 51 D8 65   59 21 F0 00 00 2A C0 09  .o.m.Q.eY!...*..
0030: C0 13 00 2F C0 04 C0 0E   00 33 00 32 C0 07 C0 11  .../.....3.2....
0040: 00 05 C0 02 C0 0C C0 08   C0 12 00 0A C0 03 C0 0D  ................
0050: 00 16 00 13 00 04 00 FF   01 00 00 3E 00 0A 00 34  ...........>...4
0060: 00 32 00 17 00 01 00 03   00 13 00 15 00 06 00 07  .2..............
0070: 00 09 00 0A 00 18 00 0B   00 0C 00 19 00 0D 00 0E  ................
0080: 00 0F 00 10 00 11 00 02   00 12 00 04 00 05 00 14  ................
0090: 00 08 00 16 00 0B 00 02   01 00                    ..........
[Raw read]: length = 5
0000: 15 03 00 00 02                                     .....
[Raw read]: length = 2
0000: 02 28                                              .(
Thread-0, READ: SSLv3 Alert, length = 2
Thread-0, RECV TLSv1 ALERT:  fatal, handshake_failure
```

和提示信息描述的一样，是在SSL握手阶段就失败了，采用的协议是SSLv3。估计是这个协议被svn服务器禁用了。  
尝试配置成TLSv1，发现可以正常访问。

## 最终处理方式

由于原来的协议配置是在代码写死的，现在调整代码修改成在配置文件形式，见xxx.properties, 修改部分如下所示。  
和原来一样，如果需要修改，可以在xxx-user.properties进行覆盖。

```
# 系统属性,通过System.setProperty设置,不清楚具体含义请不要随意修改
# 调试时可以设置为all,否则设置为none
javax.net.debug=none
https.protocols=SSLv2Hello,SSLv3,TLSv1
svnkit.http.sslProtocols=SSLv2Hello,SSLv3,TLSv1
jsse.enableSNIExtension=false
```

或许以后用得着的内容:

* 调试网络信息，可以考虑javax.net.debug这个参数，它的值有很多选择，可以查阅相关材料
* svnkit的协议配置是通过svnkit.http.sslProtocols这个参数指定的
* 有空了解一下HTTPS/SSL的基本原理和过程