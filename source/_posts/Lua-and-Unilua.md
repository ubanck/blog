---
title: Lua and Unilua
date: 2016-03-21 19:20:22
tags:
---
###LUA学习记录：

C API是一个C代码与Lua进行交互的函数集。他有以下部分组成：读写Lua全局变量的函数，调用Lua函数的函数，运行Lua代码片断的函数，注册C函数然后可以在Lua中被调用的函数，等等。

在C和Lua之间通信关键内容在于一个虚拟的栈。几乎所有的API调用都是对栈上的值进行操作，所有C与Lua之间的数据交换也都通过这个栈来完成。另外，你也可以使用栈来保存临时变量。

Lua库没有定义任何全局变量。它所有的状态保存在动态结构lua_State中，而且指向这个结构的指针作为所有Lua函数的一个参数。
<!--more-->
###运行步骤：

创建一个state并将标准库载入之后，就可以着手解释用户的输入了。对于用户输入的每一行，C程序首先调用luaL_loadbuffer编译这些Lua代码。如果没有错误，这个调用返回零并把编译之后的chunk压入栈。（记住，我们将在下一节中讨论魔法般的栈）之后，C程序调用lua_pcall，它将会把chunk从栈中弹出并在保护模式下运行它。和luaL_laodbuffer一样，lua_pcall在没有错误的情况下返回零。在有错误的情况下，这两个函数都将一条错误消息压入栈；我们可以用lua_tostring来得到这条信息、输出它，用lua_pop将它从栈中删除。

###堆栈：关于栈的操作请参见Programing in Lua一书中的186页的一个函数
压入元素，API有一系列压栈的函数，他将每种可以用C来描述的Lua类型压栈（pushnil,pushnumber,pushboolean,pushstring）等等，第一参数都是luaState!
使用checkStack来检测栈是否有足够的空间！
有检测元素类型的函数，比如 isnumber之类的！

###查询元素，tonumber,tostring之类的

堆栈的操作，gettop返回堆栈元素个数，也是栈顶元素的索引！settop设置栈顶（堆栈中元素个数）为一个指定的值！如果开始的栈顶高于指定的栈顶，顶部的值将会被丢弃！settop(L,0)清空堆栈！
使用负数会设置堆栈顶到指定的索引！
pushvalue压入堆栈上指定索引的一个拷贝到栈顶！insert移动栈顶的元素到指定索引的位置，并将这个索引位置上面的元素全部上移至栈顶被移动后留下的空隔！
replace从栈顶弹出元素将其设置到指定索引位置，没有任何移动操作

LUA中使用更多的使用对栈的操作来实现数据的交互！提供了一些对栈操作的API标准，在UniLua中，依照标准实现了这些函数，名字都只是小改动了一些，毕竟C#跟C语法规则还是不同！在对堆栈的操作中，是用索引来进行定位的，正数索引自底向上取值，负数索引自顶向下取值（以正负1开始）！在其中settop函数中使用了索引0用来清空堆栈
------
###扩展你的程序
做为配置语言是LUA的一个重要内容！

打开包并加载函数库，加载文件中的信息块 （参见188页 ）

引入table的概念
如 ；在Lua文件中进行这样的定义：BLUE={r=0,g=0,b=1} background=BLUE 在C程序里面引用getglobal(L,background); red=getfield("r") green=getfield("g")......
getfield函数在标准的LUA库里面没有提供，需要自己实现，在LuaAPI中提供了gettable函数，他接受table在栈中的位置为参数，将对应key值出栈，返回与key对应的value！
也就是说，可以使用这个方式，将对栈的操作进行封装，封装一个getfield函数来方便我们对table中值的取的。


###The Registry
registry 一直位于一个由LUA_REGISTRYINDEX定义的值所对应的假索引(pseudo-index)的位置。一个假索引除了他对应的值不在栈中之外，其他都类似于栈中的索引。
registry就是普通的Lua表，因此，你可以使用任何非nil的Lua值来访问她的元素。然而，由于所有的C库共享相同的registry ，你必须注意使用什么样的值作为key，否则会导致命名冲突。

registry实现了全局的值，upvalue机制实现了与C static等价的东西