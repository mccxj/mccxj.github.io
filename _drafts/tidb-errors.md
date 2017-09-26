---
layout: post
comments: true
title: "tidb源码学习之错误管理"
description: "对tidb中错误管理部分的使用介绍"
categories: ["tidb", "error", "学习", "源码", "错误"]
---

### golang错误机制

golang没有内置所谓异常的方式，通常是通过返回一个error参数来标识是否出现错误。不过也提供了一个简单粗暴的panic,recover机制，它只是在应用的框架上使用，绝不是用来控制错误处理流程的。

error是一个简单的接口，只要实现Error方法，就是一个error。自定义实现error，就可以通过判断不同的error类型，来做一些区别对待，例如是忽略并显示警告，还是显示错误。

### tidb错误类型

```
type Error struct {
	class ErrClass
	code ErrCode
	message string
	args []interface{}
	file string
	line int
}
```

上面是tidb自定义错误的定义，包括了错误类型，错误码，错误消息，还有一些错误位置相关的信息。

它有一个方法ToSQLError，用来转换成对外的错误信息(主要是mysql错误类型和tidb专有的)

转换关系是这样的，可以看看types包中的errors.go的用法，了解实际的使用。
```
ErrClassToMySQLCodes map[ErrClass](map[ErrCode]uint16)
terrors, trace, sqlerror
```

### 错误链的使用

类似java那样带错误链的功能，可以看到更底层的错误信息。


介绍trace机制，errors的
