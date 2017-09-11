---
layout: post
comments: true
title: "hello erlang, part 4"
description: "erlang的学习笔记"
categories: ["erlang", "列表", "高阶", "笔记", "测试", "eunit"]
---

### 练习重点
列表解析和高阶函数是erlang的基本用法之一，熟练掌握还是很有必要的。
作为一个阶段的学习总结，按照书上的一个练习题，尝试实现列表模块中的一些方法。

### 后续学习计划
再找时间总结一下位匹配用法，然后就开始客户端服务器编程模型，
网络和分布式方面的内容。

相关内容虽然都看过一下，但是没有敲过代码，感觉心里没底。还是那个硬道理：纸上得来终觉浅呀。

### 简单的测试用例
作为另外一个章节的内容，单元测试。我顺便实践一下erlang的单元测试框架eunit，
里边使用了宏来作为assert语句(使用eclipse插件可以有很好的提示)，
总的来说，xunit的基本风格都差不多，注意import eunit的包，方法名采用_test结束。
执行的时候调用模块的test方法就可以了。我简单写了一些测试用例:

{% highlight erlang %}
-module(myliststest).
-include_lib("eunit/include/eunit.hrl").
-import_all(mylists).

%% EUnit TestCase

%% all/2
all_empty_test() ->
  ?assert(mylists:all(fun(E) -> E > 10 end, [])).
all_test() ->
  ?assertNot(mylists:all(fun(E) -> E > 10 end, [11,12,10])).

%% any/2
any_empty_test() ->
  ?assertNot(mylists:any(fun(E) -> E > 10 end, [])).
any_test() ->
  ?assert(mylists:any(fun(E) -> E > 10 end, [11,12,10])).

%% append/1
append_1_test() ->
  ?assertEqual([1,2,3,a,b,4,5,6], mylists:append([[1, 2, 3], [a, b], [4, 5, 6]])).

%% append/2
append_2_test() ->
  ?assertEqual("abcdef", mylists:append("abc", "def")).

%% delete/2
delete_test() ->
  ?assertEqual([1,2,4], mylists:delete(3, [1,2,3,4])).

%% dropwhile/2
dropwhile_test() ->
  ?assertEqual([1,2], mylists:dropwhile(fun(E) -> E >= 3 end, [1,2,3,4])).

%% duplicate/2
duplicate_test() ->
  ?assertEqual([x, x], mylists:duplicate(2, x)).

%% filter/2
filter_test() ->
  ?assertEqual([3,4], mylists:filter(fun(E) -> E >= 3 end, [1,2,3,4])).

%% reverse/1
reverse_test() ->
  ?assertEqual([4,3,2,1], mylists:reverse([1,2,3,4])).

%% min/1
min_test() ->
  ?assertEqual(2, mylists:min([3,4,2,5])).

%% zip/2
zip_2_test() ->
  ?assertEqual([{1,3},{2,4}], mylists:zip([1,2], [3,4,5])).

%% zip/3
zip_3_test() ->
  ?assertEqual([{1,3,5},{2,4,7}], mylists:zip3([1,2], [3,4,5], [5,7])).

%% zipwith/3
zipwith_test() ->
  ?assertEqual([5,7,9], mylists:zipwith(fun(X, Y) -> X+Y end, [1,2,3], [4,5,6])).

%% zipwith/4
zipwith3_test() ->
  ?assertEqual([[a,x,1],[b,y,2],[c,z,3]], mylists:zipwith3(fun(X, Y, Z) -> [X,Y,Z] end, [a,b,c], [x,y,z], [1,2,3])).
{% endhighlight %}

### 示例代码
根据上面的测试用例，我写了实现代码，如下所示。注意的时候，我偷懒使用了export_all,
但是min/2好像默认有一个已经被引入了，所以需要去除这个模块。就export_all本身来说，
也不是推荐方式，还是应该严格区分public api和local api。
{% highlight erlang %}
-module(mylists).
-compile(export_all).
-compile({no_auto_import,[min/2]}).

%% all/2
all(_P, []) -> 
  true;
all(P, [H|T]) ->
  case P(H) of
	true -> all(P, T);
	false -> false
  end.

%% any/2
any(_P, []) ->
  false;
any(P, [H|T]) ->
  case P(H) of
	true -> true;
	false -> any(P, T)
  end.

%% append/1
append(LL) ->
  [X || L <- LL, X <- L].

%% append/2
append(LA, LB) ->
  LA ++ LB.

%% delete/2
delete(E, L) ->
  [X || X <- L, E /= X].

%% dropwhile/2
dropwhile(P, L) ->
  [X || X <- L, not P(X)].

%% duplicate/2
duplicate(N, E) ->
  duplicate(N, E, []).
duplicate(0, _E, L) ->
  L;
duplicate(N, E, L) when N > 0 ->
  duplicate(N-1, E, [E|L]).

%% filter/2
filter(P, L) ->
  [X || X <- L, P(X)].

%% reverse/1
reverse(L) ->
  reverse(L, []).
reverse([], L) ->
  L;
reverse([H|T], L) ->
  reverse(T, [H|L]).

%% min/1
min([H|T]) ->
  min(T, H).
min([], M) ->
  M;
min([H|T], M) ->
  if
    H < M -> min(T, H);
    true -> min(T, M)
  end.

%% zip/2
zip(LA, LB) ->
  reverse(zip(LA, LB, [])).
zip([], _LB, L) ->
  L;
zip(_LA, [], L) ->
  L;
zip([HA|TA], [HB|TB], L) ->
  zip(TA, TB, [{HA,HB}|L]).

%% zip/3
zip3(LA, LB, LC) ->
  reverse(zip3(LA, LB, LC, [])).
zip3([], _LB, _LC, L) ->
  L;
zip3(_LA, [], _LC, L) ->
  L;
zip3(_LA, _LB, [], L) ->
  L;
zip3([HA|TA], [HB|TB], [HC|TC], L) ->
  zip3(TA, TB, TC, [{HA,HB,HC}|L]).

%% zipwith/3
zipwith(P, LA, LB) ->
  [P(X,Y) || {X,Y} <- zip(LA, LB)].

%% zipwith/4
zipwith3(P, LA, LB, LC) ->
  [P(X,Y,Z) || {X,Y,Z} <- zip3(LA, LB, LC)].
{% endhighlight %}
