---
layout: post
comments: true
title: "tidb源码学习之auto_increment"
description: "对tidb中auto_increment实现的研究"
categories: ["tidb", "auto_increment", "源码"]
---

### 自增ID

TiDB 的自增 ID (Auto Increment ID) 只保证自增且唯一，并不保证连续分配。
TiDB 目前采用批量分配的方式，所以如果在多台TiDB上同时插入数据，分配的自增 ID 会不连续。

### 代码实现位置

https://github.com/pingcap/tidb/tree/master/meta/autoid/autoid.go
