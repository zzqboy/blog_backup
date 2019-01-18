---
title: lua中面向对象
date: 2018-06-17 23:24:20
tags: lua
---
# pil实现方法
要点：self语法糖、setmatetable元表、index方法
提到了如何用__index方法来实现类和继承，让lua也具备面向对象的特性

## Account.lua
```lua
module("Account", package.seeall)

function Account:new(o)
    o = o or {}
    setmetatable(o, self)
    self.__index = self
    return o
end

function Account:deposit()
	print("account deposit")
end
```
这个机制的意思就是在创建一个新table的时候，把元类设为
Account，在Account查找没有的方法
```lua
a = Account:new{balance = 0}
a:deposit(100) -- 等价于 getmetatable(a).__index.deposit(a, 100)
```
实际上我们编写类的时候习惯把每个类划分为文件，如何用module结合上面的做法呢？
这里写另外继承account的specialaccount类

## SpecialAccount.lua
```lua
module("SpecialAccount", package.seeall)
require("Account")

-- 继承自Account
_G.SpecialAccount = Account:new()

--定义基类没有的方法
function SpecialAccount:getLimit()
	print("SpecialAccount getLimit")
end
```
注意上面的第5行代码，为什么不是书中的SpecialAccount = Account:new()？
因为我们用module的时候已经默认引入了SpecialAccount，如果用这种做法，那么相当于增加了另外的变量，那么下面的代码就会报错

> attempt to call method ‘new’ (a nil value)

## test.lua
```lua
require "Account"
require "SpecialAccount"

a = Account:new()
a.deposit()

s = SpecialAccount:new()
s.deposit()
s.getLimit()
```
原因在于module(“SpecialAccount”, package.seeall)相当于下面的代码
```lua
local modname = "SpecialAccount"
local M = {}
_G[modname] = M
package.loaded[modname] = M
setfenv(1, M)
```
当SpecialAccount = Account:new()的时候，其实相当于
```lua
_G["SpecialAccount"].SpecialAccount = Account:new()
```
这样在require “SpecialAccount”自然就找不到内嵌的new
按照上面的写法就可以自然玩起面向对象了，附上代码
[https://github.com/zzqboy/misc/tree/master/lua_oo](https://github.com/zzqboy/misc/tree/master/lua_oo)