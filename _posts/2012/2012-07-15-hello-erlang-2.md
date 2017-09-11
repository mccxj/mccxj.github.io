---
layout: post
comments: true
title: "hello erlang, part 2"
description: "erlang的学习笔记"
categories: ["erlang", "笔记", "排序", "函数式", "尾递归"]
---

### 练习重点
1. 依然是熟悉erlang匹配模式的编程方式，对于用习惯了判断，循环操作的人来说，这个变化很大
2. 熟悉erlang列表操作
3. 为后面学习高阶函数fun，还有更多高级数据类型打好基础

### 练习1:压缩有序列表
类似搜索里边的倒排索引，需要对词对应的文档列表进行存放。
在这里，我们将一个有序列表转换成另外一种形式，如
{% highlight erlang %}
%% 原始形式
[1,1,2,4,5,6,6,98,100,101,102,102].
%% 最终形式
[{1,2},{4,6},{98,98},{100,102}].
{% endhighlight %}

### 示例代码
{% highlight erlang %}
multi(L) -> lists:reverse(multi_local(L, [])).
multi_local([], R) -> R;
multi_local([H|L], R) ->
	{LL, E} = multi_r(L, H),
	multi_local(LL, [{H, E}|R]).

%% 获取L里边里边和H同属一组的最后一个数,同时返回剩余的列表
%% 如果L以H开头，表示出现重复数
%% 如果L以H+1开头，表示出现序列数
multi_r(L, H) ->
	N = H + 1,
	case L of
		[H|LL] -> multi_r(LL, H);
		[N|LL] -> multi_r(LL, N);
		[] -> {[], H};
		_ -> {L, H}
	end.
{% endhighlight %}

### 效率问题
这里和上次不一样的是，这里没有使用lists:append方法，而是在最后使用了reverse.

{% highlight erlang %}
[1] ++ [2, 3].
lists:append([1], [2, 3]).
lists:reverse([3|[2, 1]]).
{% endhighlight %}
上面三种方式得到的结果是一样的。前面两种，每增加一个元素，都会遍历左边的列表，所以在递归里边处理的话，
效率并不好。把元素添加到表头，在递归完成后进行反转，效果就会好些。

### 练习2:马踏棋盘
作为一个阶段的学习总结，对[马踏棋盘的算法思考][1]用erlang实现。

代码比较悲剧，我选择了array，用了一些fun函数，从这么个例子就可以看出，自己还没掌握好erlang惯用法。
感觉数据结构没有选好，并且用了命令语言的一些编程思路，有些东西或许可以用列表解析来弄，
等再掌握多一些东西，再回头来体会一把!
{% highlight erlang %}
-module(horse).

%% Exported Functions
-export([walkboard/0]).

%% API Functions
walkboard() -> 
  %% lists:foreach(fun(I) -> lists:foreach(fun(J) -> walkboard({I, J}) end, lists:seq(0,7)) end, lists:seq(0,7)).
  walkboard({2,3}).

walkboard({I,J}) ->
  {ARR, PATH, POSS} = init(),
  catch walk(0, setvalue({I, J}, 1, ARR), array:set(0, {I,J}, PATH), POSS).

walk(63, _, PATH, _) -> throw(PATH);
walk(MARK, ARR, PATH, POSS) ->
  {IP, JP} = array:get(MARK, PATH),
  WC = fun({I,J,_}) -> 
         walk(MARK+1, setvalue({I+IP,J+JP}, 1, ARR), array:set(MARK+1, {I+IP,J+JP}, PATH), POSS)
       end,
  lists:foreach(WC, filter(MARK, PATH, POSS, ARR)).

%%
%% Local Functions
%%
init() ->
  ARR = array:new(8, {default,array:new(8, {default,0})}),
  PATH = array:new(64),
  POSS = [{-2, 1}, {2, 1}, {1, 2}, {-1, 2}, {2, -1}, {-2, -1}, {-1, -2}, {1, -2}],
  {ARR, PATH, POSS}.

filter(MARK, PATH, POSS, ARR) ->
  {I, J} = array:get(MARK, PATH),
  Fun1 = fun({IP, JP}) ->
           {IXP, JXP} = {I+IP, J+JP},
           case valid({IXP, JXP}, ARR) of
             false -> {IP, JP, -1};
             true -> 
             %% 计算空点数
             CAL = fun({IP2, JP2}, Sum) -> 
                     case valid({IXP+IP2, JXP+JP2}, ARR) of
                       true -> 1 + Sum;
                       false -> Sum
                     end
                   end,
                   {IP, JP, lists:foldl(CAL, 0, POSS)}
           end
         end,
  %% 最后进行排序
  T = lists:map(Fun1, POSS),
  S = lists:filter(fun({_, _, Z}) -> Z == 0 end, T),
  LEN = length(S),
  if
    LEN > 1 -> [];
    LEN > 0 -> S;
    true -> SS = lists:filter(fun({_, _, Z}) -> Z >= 0 end, T),
        lists:sort(fun({_, _, Z1}, {_, _, Z2}) -> Z1 < Z2 end, SS)
  end.
  

valid({I,J}, ARR) ->
  I >= 0 andalso I<8 andalso J>=0 andalso J<8 andalso getvalue({I,J}, ARR) == 0.

getvalue({I,J}, ARR) ->
  array:get(J, array:get(I, ARR)).

setvalue({I,J}, VAL, ARR) ->
  array:set(I, array:set(J, VAL, array:get(I, ARR)), ARR).
{% endhighlight %}
 [1]: 20120712_horse-riding-chessboard.html
