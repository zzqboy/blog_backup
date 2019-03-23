---
title: erlang gb_tree平衡树做排行
date: 2018-03-14 23:30:53
tags: algorithm
categories: 技术
---
# 介绍
平衡树简称AVL，出名的有红黑树，这里介绍一下erlang中gb_tree的实现
gb_tree的原理比红黑树简单，没有过多的旋转跳跃闭着眼，是一种叫AA树的结构（Arne Andersson’s General Balanced Trees），有兴趣看这篇论文：[传送门](http://user.it.uu.se/~arnea/ps/simp.pdf)
<!-- more -->

# 结构
{Size, Tree}是整个结构体，Tree的定义又是{Key, Value, Smaller, Bigger} | nil
初始化直接返回{0, nil}

# 插入
```erlang
insert(Key, Val, {S, T}) when is_integer(S) ->  
    S1 = S+1,
    {S1, insert_1(Key, Val, T, ?pow(S1, ?p))}.   % 给size+1，insert_1返回新的结构
```
insert_1又是如何找到要插入的位置，且做平衡的?
```erlang
% 由于对称性，这里讲插入左子树的情况就行
insert_1(Key, Value, {Key1, V, Smaller, Bigger}, S) when Key < Key1 ->  % 要插入的key比目前节点的key小
    case insert_1(Key, Value, Smaller, ?div2(S)) of
        % 递归，在目前节点的左子树继续查找，当Smaller为nil的时候返回下面两种情况
        % T1 就是已经更新好的左子树
	{T1, H1, S1} ->
	    T = {Key1, V, T1, Bigger},
	    {H2, S2} = count(Bigger),
	    H = ?mul2(erlang:max(H1, H2)),  %% 每层都会被调用一次
	    SS = S1 + S2 + 1,
	    P = ?pow(SS, ?p),
	    if
		H > P ->  % 满足这个条件就重新平衡
		    balance(T, SS);
		true ->
		    {T, H, SS}
	    end;
	T1 ->
	    {Key1, V, T1, Bigger}  % 结果--节点和右子树均没改变，T1改变
    end;
```
# 平衡
也就是上面的balance(T, SS),这里什么时候会被执行呢？看一下下面代码
```erlang
%% 是的insert_1的{T1，H1， S1}分支被执行
insert_1(Key, Value, nil, S) when S =:= 0 ->
    {{Key, Value, nil, nil}, 1, 1};
```
看看官方的说明
![gb_tree](/img/gb_tree.jpg)
也就是说 13行的H>P就是重新进行平衡的时候了，而平衡的操作也很简单，看下代码，就是按顺序填满一棵树
```erlang
balance_list_1(L, S) when S > 1 ->
    Sm = S - 1,
    S2 = Sm div 2,
    S1 = Sm - S2,
    {T1, [{K, V} | L1]} = balance_list_1(L, S1),
    {T2, L2} = balance_list_1(L1, S2),
    T = {K, V, T1, T2},
    {T, L2};
balance_list_1([{Key, Val} | L], 1) ->
    {{Key, Val, nil, nil}, L};
balance_list_1(L, 0) ->
    {nil, L}.
```
# 删除
删除比插入是更简单了，找到对应的结点，然后从结点的右子树里找到一个最小的代替当前的点
```erlang
delete_1(Key, {Key1, Value, Smaller, Larger}) when Key < Key1 ->
    Smaller1 = delete_1(Key, Smaller),
    {Key1, Value, Smaller1, Larger};
delete_1(Key, {Key1, Value, Smaller, Bigger}) when Key > Key1 ->
    Bigger1 = delete_1(Key, Bigger),
    {Key1, Value, Smaller, Bigger1};
delete_1(_, {_, _, Smaller, Larger}) ->
    merge(Smaller, Larger).

merge(Smaller, nil) ->
    Smaller;
merge(nil, Larger) ->
    Larger;
merge(Smaller, Larger) ->
    {Key, Value, Larger1} = take_smallest1(Larger),
    {Key, Value, Smaller, Larger1}.
```
可以看到整棵树没有旋转等复杂操作，但是仍是一个效率比lists高的二叉树

# 排行榜
如果是上面实现的数据结构，那么排行榜是实时的，且各个操作效率比内置的List还快，我之前写的版本就是这个改的。
只是有的地方需要改变一下，大概有下面几点

因为无法知道以后的需求，可以加入一个cmp函数自定义排序顺序
查找前几名的时候，因为查找一个可以二分法，前几名同样也可以用二分法获取
当某个榜上的玩家数据更新的时候，需要从树里删除再重新插入，这样才能保持有序
其他的可以看看 [https://cloud.tencent.com/developer/article/1006610](https://cloud.tencent.com/developer/article/1006610)