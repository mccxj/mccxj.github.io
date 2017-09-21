---
layout: post
comments: true
title: "tidb源码学习之测试框架"
description: "对tidb中测试框架部分的使用介绍"
categories: ["tidb", "gocheck", "学习", "源码", "测试"]
---

tidb项目对测试非常重视，采用了大量的测试用例来保证代码的质量，目前代码测试的整体覆盖率在70%左右。

golang自带了测试框架，但tidb实际上使用的是gocheck的测试框架，这个框架兼容官方测试框架的用法。

测试框架介绍路径 
gocheck有一些额外的优点，可以在使用过程中慢慢体会。

- 强大的assert功能
- 按test suite来组织测试用例，并支持setUp,tearDown
- 一些便捷功能，如临时文件

下面介绍tidb在测试中一些有趣的用法。在tidb的测试用例里边会用到testkit和testleak两个工具包。

### testkit

主要是提供一些便捷的工具，例如查询并检查错误，比较查询结果等，对照使用即可。

### testleak

主要是提供了一个检测goroutine泄露的工具。使用很方便，只需要在开头加一个defer语句就可以了。原理也不复杂，就是通过runtime.Stack()得到所有正在运行的goroutine的堆栈，排除一些已知的堆栈，如果还剩下一些未知的，那可能就是有问题了。详细的可以看看tidb的源代码。
