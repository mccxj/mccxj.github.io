---
layout: post
comments: true
title: "IOS14无法安装企业自签名App案例"
description: "前阵子苹果发布了IOS14，很多人就发现企业的自签名App就无法安装了"
categories: ["IOS", "证书"]
---

## 问题描述

前阵子苹果发布了IOS14，很多人就发现企业的自签名App就无法安装了。很不走运，我们有个App也遇到了。网上找了一圈，大多数文章都说要升级TLSv1.2，但也很不走运，这个办法对我们也没用。

## 解决方案

以ios14、self signed certificate等作为关键字在stackoverflow、apple developer forum找了一圈，发现不少人遇到这个问题，最终碰运气了一堆方案，总算能安装了。

下面简单记录一下配置的关键点。

| 配置项                     | 配置要求                                | 备注                                             |
|----------------------------|-----------------------------------------|--------------------------------------------------|
| TLS协议版本                | TLSv1.2                                 | 基本要求，1.2+                                   |
| 签名算法                   | SHA-256                                 | 基本要求，至少SHA1是不行的                       |
| 公钥算法                   | RSA2048                                 | 基本要求，RSA2048+                               |
| 证书时间                   | 按ios13的说法不能大于825天              | 测试发现，设置10年也没有问题                     |
| 扩展字段(DNS设置)          | 在Subject Alternative Name标识对应的DNS | 官方特殊要求，只在CommonName标识DNS是不够的      |
| 扩展字段(ExtendedKeyUsage) | 在ExtendedKeyUsage标识serverAuth        | 官方特殊要求                                     |
| 扩展字段(IP设置)           | 在Subject Alternative Name标识对应的IP  | 在苹果论坛看到的，没有找到官方出处，但试了有影响 |
| 扩展字段(basicConstraints) | 要标识CA:TRUE,pathlen:0                 | 设置为CA证书，没有特别验证真伪                   |

设置后，如果是公网的话，可以通过一些在线工具来验证，例如https://myssl.com/, 工具箱比较齐全，报告也比较详细。如果没有环境，那可以通过chrome来验证，众所周知，chrome的证书验证是很严格的，安装信任证书(自签名)，打开网站后通过看看是不是绿色的，有问题的话，也可以通过F12 + Security这个Tab页来具体查看。

## 步骤参考

- 证书生成命令

```
openssl req \
-newkey rsa:2048 \
-x509 \
-nodes \
-keyout myKey.key \
-new \
-out myCert.crt \
-subj /CN=hello.com \
-config ./myConfig.cnf \
-reqexts SAN \
-extensions SAN \
-sha256 \
-days 365
```

- 配置信息

```
cat myConfig.cnf
[ req ]
default_bits = 2048
distinguished_name = req_distinguished_name
req_extensions = SAN
extensions = SAN
[ req_distinguished_name ]
countryName = CN
stateOrProvinceName = AA
localityName = BB
organizationName = CC
[SAN]
subjectAltName = @alt_names
extendedKeyUsage = serverAuth
basicConstraints=CA:TRUE,pathlen:0
[alt_names]
DNS.1 = hello.com
IP.1 = 1.2.3.4
```

## 参考材料

- [Not able to enroll iPhone in MDM in iOS 14 with Self signed certificate](https://developer.apple.com/forums/thread/661259)
- [Requirements for trusted certificates in iOS 13 and macOS 10.15](https://support.apple.com/en-us/HT210176)
- [IOS 14 - Self signed certificate - not trustable](https://stackoverflow.com/questions/63600820/ios-14-self-signed-certificate-not-trustable)