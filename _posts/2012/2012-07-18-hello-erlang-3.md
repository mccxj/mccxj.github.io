---
layout: post
comments: true
title: "hello erlang, part 3"
description: "erlang的学习笔记"
categories: ["erlang", "消息", "进程", "笔记"]
---

### 练习重点
并发编程是erlang的重要话题，所以这次先来热身，熟悉一下erlang中进程的基本用法。

### 练习1:进程环
编写一个程序，它生成N个进程并相连形成一个环，一旦启动，这些进程会环绕发送M个消息。
然后当收到消息的时候正常终止。调用ring:start(M, N, Message)来启动环。

### 思路及编程示例
首先，为了形成环状，就是让最后一个进程能够给第一个进程发送消息，
调用spawn生成进程的时候，需要把第一个pid一直传递到最后一个。所以需要有个start的方法，
但需要多加个第一个进程id的参数，关注参数N=1作为临界点。

还有，需要等待消息，然后给下一个进程发送消息，之后继续等待消息，直到M=0。
所以需要一个等待消息的方法，同时需要关注参数M=0作为临界点。

最后，整个环，所有的进程一开始都在等待，需要有个消息来启动环，
可以在第一个进程启动下一个进程之前，往自身邮箱里边发送一条消息，
这样第一个进程进入等待消息的时候，就能接收到消息，从而启动环。

我的代码示例如下：
{% highlight erlang %}
-module(ring).

%% Exported Functions
-export([start/3]).
-export([start/4]).

%% API Functions
start(M, N, Message) ->
  self() ! {self(), start},
  start(M, N, Message, self()).

start(M, 1, Message, Rid) ->
  waitmessage(M, 1, Message, Rid);

start(M, N, Message, Rid) ->
  Pid = spawn(ring, start, [M, N-1, Message, Rid]),
  waitmessage(M, N, Message, Pid).

waitmessage(0, _N, _Message, _Pid) ->
  receive
    {Pd, Msg} ->
      io:format("pid ~p receive message ~p from ~p~n", [self(), Msg, Pd])
      ok
  end;

waitmessage(M, N, Message, Pid) ->
  receive
    {Pd, Msg} ->
      io:format("pid ~p receive message ~p from ~p~n", [self(), Msg, Pd]),
      Pid ! {self(), Message},
      waitmessage(M-1, N, Message, Pid)
  end.
{% endhighlight %}

通过简单的测试，如果去掉io:format的话，效率会提高不少。IO操作消耗的时间还是不可小视呀。
另外，测试大数量进程的时候，内存也用得比较厉害，毕竟这个练习更像是一个同步操作。
