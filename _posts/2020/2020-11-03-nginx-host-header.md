---
layout: post
comments: true
title: "Nginx中Host Header相关问题"
description: "最近新上的一个webservice的应用在k8s上不能访问，最后发现是pod里边的axis2框架生成的wsdl端口不对，一直显示是1089。最后修改了nginx的配置，并重启pod才修复"
categories: ["Nginx", "Host"]
---

## 问题描述

```
nginx<ip1:1293>  ----> ingress<ip2:1089> ---->service --> pod<:8080>
```

最近新上的一个webservice的应用在k8s上不能访问，最后发现是pod里边的axis2框架生成的wsdl端口不对，一直显示是1089。最后修改了nginx的配置，并重启pod才修复，下面进行总结。

## nginx的Host基础知识

nginx和Host头相关的变量有：

- $host 表示请求过来的Host头(如果有的话)，或者请求的域名(不带端口)
- $http_host 表示请求过来的Host头(如果有的话)，或者空
- $proxy_host 表示upstream的请求地址

默认情况下，通过proxy_pass之后Host会被改写，指向$proxy_host，也就是
```
proxy_set_header Host $proxy_host
```

这样upstream server就不能获取到原始的地址，所以做法是：
```
proxy_set_header Host $host:$server_port
```
或者
```
proxy_set_header Host $http_host
```

第一个是当前外层nginx的写法，第二个是nginx ingress controller的默认配置(实际是$best_http_host,但他们是同一个变量)。

两者区别在于请求有没有带Host头的情况，但这个只针对Http/1.0比较特殊，1.1之后都要求带Host头，否则会返回400。

## 验证结果

测试结果如下，结果都是根据request的getServername,getServerPort获取的：

- 直接ingress访问，没有Host的情况，应用看到ip2:80
- GET http://ip2:1089/health.jsp HTTP/1.0
- 直接ingress访问，按Host信息来，应用看到ip2:1089
- GET http://ip2:1089/health.jsp HTTP/1.1
- Host: ip2:1089
- 直接ingress访问，按Host信息来，应用看到ip1:1293，通过外层nginx之后的Host就是这个结果
- GET http://ip2:1089/health.jsp HTTP/1.1
- Host: ip1:1293
- 通过外层nginx访问，没有Host头，应用能够看到IP,但端口是看不到了
- GET http://ip1:1293/health.jsp HTTP/1.0
- 通过外层nginx访问，正常
- GET http://ip1:1293/health.jsp HTTP/1.1
- Host: ip1:1293
- Http1.1不带头，返回400
- GET http://ip1:1293/health.jsp HTTP/1.1

## 总结

- Http/1.0已经绝迹，所以不需要考虑Host缺少的情况，这种情况下nginx和ingress的配置方式没有区别
- 对于axis2，如果开启modifyUserWSDLPortAddress参数，那么wsdl的soap location会重新生成，但ip部分每次请求都变化，port部分在第一次访问后就固定。所以如果涉及Host端口调整，应用需要重启才能生效。
- 基于第二点，由于端口第一次会固定，所以不应该把访问wsdl作为pod的探针，以免端口被错误固定。