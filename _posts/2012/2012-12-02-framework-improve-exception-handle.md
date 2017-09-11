---
layout: post
comments: true
title: "从某维护系统的架构改造谈起–异常处理(旧)"
description: ""
categories: ["总结"]
---

这是刚到公司时写的~

Java 提供了两类主要的异常：runtime exception和checked exception。
所有的checked exception是从java.lang.Exception类衍生出来的，
而runtime exception则是从java.lang.RuntimeException或java.lang.Error类衍生出来的。

如果你希望强制你的类调用者来处理异常，那么就用Checked Exception;
如果你不希望强制你的类调用者来处理异常，就用UnChecked。
那么究竟强制还是不强制，权衡的依据在于从业务系统的逻辑规则来考虑，
如果业务规则定义了调用者应该处理，那么就必须Checked，如果业务规则没有定义，就应该用UnChecked。

至于类调用者catch到NoSuchUserException和PasswordNotMatchException怎么处理，也要根据他自己具体的业务逻辑了。
或者他有能力也应该处理，就自己处理掉了；或者他不关心这个异常，也不希望上面的类调用者关心，就转化为RuntimeException；
或者他希望上面的类调用者处理，而不是自己处理，就转化为本层的异常继续往上抛出来。
根据上面的一些观点在现有的系统上建立基于三层架构的异常处理模型，主要有以下做法：

* Dao层次关注底层异常，当很多SQLException无法恢复，转换成统一的RuntimeException交给Service处理
* Service层关心某些有业务含义的业务处理，并转化成相应的业务异常(同样也是RuntimeException)交给Action处理
* Action层可能对某些特定的业务异常感兴趣，感兴趣就处理(或许尝试修复)，不感兴趣就交给统一业务异常处理器处理
* 统一业务异常处理器会记录exception log，并负责相应的错误展现页面

实现思路(待续)：
* exception util:提供一些异常处理的工具,例如转换异常
* exception wrapper:作为统一的异常类，可以用来保存原始的exception信息
* exception collector:可以保存多个exception信息的异常类，用于某些特殊场合
* exception handler:用于处理exception的统一处理，主要是日志与错误页面

