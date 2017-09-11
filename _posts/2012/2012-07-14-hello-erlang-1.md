---
layout: post
comments: true
title: "hello erlang, part 1"
description: "erlang的学习笔记"
categories: ["erlang", "笔记", "排序", "函数式", "尾递归"]
---

### 练习重点
1. erlang是单赋值的，就好像都经过java里边的final修饰过
2. erlang没有内置的循环结构，这个和大多数语言是不一样的
3. erlang是函数式语言，需要理解模式匹配编程方式
4. erlang经常会遇到递归, 并且支持尾递归函数调用

### 练习1:数据库demo
作为学习erlang里边的模式匹配，熟悉语法。
编写一个数据库模块db.erl，它能够存储、检索、删除元素。我们需要实现的接口有:
{% highlight erlang %}
db:new()                   %% Db.
db:destroy(Db)             %% ok.
db:write(Key, Element, Db) %% NewDb.
db:delete(Key, Db)         %% NewDb.
db:read(Key, Db)           %% {ok, Element} | {error, instance}.
db:match(Element, Db)      %% [Key1, ..., KeyN].
{% endhighlight %}

### 实现代码
{% highlight erlang %}
-module(db).

%% Exported Functions
-export([new/0, write/3, read/2, match/2, delete/2, destroy/1]).

%% API Functions
new() -> [].
destroy(_) -> ok.
write(Key, Element, DB) -> [{Key,Element}|DB].
delete(Key, DB) -> delete_local(Key, DB, []).
read(Key, DB) ->
	case DB of
		[{Key, Element}|_] -> {oKey, Element};
		[_|ODB] -> read(Key, ODB);
	    _ -> {error, instance}
	end.
match(Element, DB) -> match_local(Element, DB, []).

%% Local Functions
match_local(Element, DB, LIST) ->
	case DB of
		[{Key, Element}|ODB] -> match_local(Element, ODB, [Key|LIST]);
		[_|ODB] -> match_local(Element, ODB, LIST);
	    _ -> LIST
	end.

delete_local(Key, DB, NDB) ->
	case DB of
		[{Key, _}|ODB] -> delete_local(Key, ODB, NDB);
		[H|ODB] -> delete_local(Key, ODB, [H|NDB]);
	    _ -> NDB
	end.
{% endhighlight %}

当然，case语句同样可以用如下代码替代：其中第三行的Key因为没有使用就换成_,
不然会提示警告variable 'Key' is unused。
{% highlight erlang %}
delete_local(Key, [{key, _}|ODB], NDB) -> delete_local(Key, ODB, NDB);
delete_local(Key, [H|ODB], NDB) -> delete_local(Key, ODB, [H|NDB]);
delete_local(_, _, NDB) -> NDB;
{% endhighlight %}

### 练习2:排序算法
下面练习一下快速排序和归并排序，再熟悉一下模式匹配和递归的写法。
{% highlight erlang %}
-module(sort).
-export([quicksort/1, mergesort/1]).

%% 快速排序
quicksort([H|Tail]) ->
	lists:append([quicksort(lists:filter(fun(X) -> X < H end, Tail)), 
				  [H], 
				  quicksort(lists:filter(fun(X) -> X >= H end, Tail))]);
quicksort(L) -> L.

%% 归并排序
mergesort(A) ->
	case length(A) of
		0 -> [];
		1 -> A;
		LEN -> M = length(A) div 2,
			   merge(mergesort(lists:sublist(A, 1, M)), 
					 mergesort(lists:sublist(A, M + 1, LEN - M)), [])
	end.
merge(A, [], L) -> lists:append([L, A]);
merge([], B, L) -> lists:append([L, B]);
merge([HA|TA], [HB|TB], L) -> 
	if
		HA > HB -> merge([HA|TA], TB, lists:append([L, [HB]]));
		true -> merge(TA, [HB|TB], lists:append([L, [HA]]))
	end.
{% endhighlight %}

