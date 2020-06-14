---
title: lua中的元表
categories: 
 - 技术
tags:
 - Java
 - lua
---

lua中元表是一个很好用的功能,它允许我们定义table或者userdata的一些操作.

首先元表并不是表,它只是我们对一些基本操作的定义,也不是table特有的,在调用table或者userdata的操作的时候,会判断元表有没有定义过该操作.这些操作实际上是lua提供的元方法,主要有一下这些元方法:
```
__tostring, __pairs, __name, __index, __newindex, __call, __add, __sub, __mul, __div, __mod, __pow, __unm, __idiv, __band, __bor, __bxor, __bnot, __shl, __shr, __concat, __len, __eq, __lt, __le, __gc, __mode,
```
比如`_add`方法,在调用`+`的时候调用
```
local t1 = {1}
local t2 = {2}
local t3 = t1 + t2
```
我们直接对两个table执行＋运算，会报错
`lua: /usercode/file.lua:3: attempt to perform arithmetic on local 't1' (a table value)`
因为lua是不允许两个table相加的,这时候我们就可以用元表,定义元方法`_add`来实现table相加.
```
local mt = {}
--定义mt.__add元方法（其实就是元表中一个特殊的索引值）为将两个表的元素合并后返回一个新表
mt.__add = function(t1,t2)
	local temp = {}
	for _,v in pairs(t1) do
		table.insert(temp,v)
	end
	for _,v in pairs(t2) do
		table.insert(temp,v)
	end
	return temp
end

local t1 = {1,2,3}
local t2 = {2}

--设置t1的元表为mt  setmetatable是lua自带的方法
setmetatable(t1,mt)

-- 运行`+`时就会调用我们定义元表中的元方法
local t3 = t1 + t2
--输出t3
local st = "{"
for _,v in pairs(t3) do
	st = st..v..", "
end
st = st.."}"
print(st)
```
结果为:
```
{1, 2, 3, 2, }
```
因为程序在执行t1+t2的时候，会去调用t1的元表mt的__add元方法进行计算。
具体的过程是：
1.查看t1是否有元表，若有，则查看t1的元表是否有__add元方法，若有则调用。
2.查看t2是否有元表，若有，则查看t2的元表是否有__add元方法，若有则调用。
3.若都没有则会报错。
所以说，我们通过定义了t1元表的__add元方法，达到了让两个表通过+号来相加的效果.

##### __call
__call可以让table当做一个函数来使用。
```
local mt = {}
--__call的第一参数是表自己
mt.__call = function(mytable,...)
    --输出所有参数
    for _,v in ipairs{...} do
        print(v)
    end
end

t = {}
setmetatable(t,mt)
--将t当作一个函数调用
t(1,2,3)
```
结果：
```
1
2
3
```
##### __tostring
__tostring可以修改table转化为字符串的行为
```
local mt = {}
--参数是表自己
mt.__tostring = function(t)
    local s = "{"
    for i,v in ipairs(t) do
        if i > 1 then
            s = s..", "
        end
        s = s..v
    end
    s = s .."}"
    return s
end

t = {1,2,3}
--直接输出t
print(t)
--将t的元表设为mt
setmetatable(t,mt)
--输出t
print(t)
```
结果：
```
table: 0x14e2050
{1, 2, 3}
```
##### __index
调用table的一个不存在的索引时，会使用到元表的__index元方法，和前几个元方法不同，__index可以是一个函数也可是一个table。
作为函数：
将表和索引作为参数传入__index元方法，return一个返回值
```
local mt = {}
--第一个参数是表自己，第二个参数是调用的索引
mt.__index = function(t,key)
    return "it is "..key
end

t = {1,2,3}
--输出未定义的key索引，输出为nil
print(t.key)
setmetatable(t,mt)
--设置元表后输出未定义的key索引，调用元表的__index函数，返回"it is key"输出
print(t.key)
```
结果：
```
nil
it is key
```
作为table：
查找__index元方法表，若有该索引，则返回该索引对应的值，否则返回nil
```
local mt = {}
mt.__index = {key = "it is key"}

t = {1,2,3}
--输出未定义的key索引，输出为nil
print(t.key)
setmetatable(t,mt)
--输出表中未定义，但元表的__index中定义的key索引时，输出__index中的key索引值"it is key"
print(t.key)
--输出表中未定义，但元表的__index中也未定义的值时，输出为nil
print(t.key2)
```
结果：
```
nil
it is key
nil
```
