# Lua笔记

字符串前加#可以获得长度



## 函数

### 声明

```lua
function func_name(a,b,c)
    
end
```

调用细节：不传的参数就是nil





## Table

### 定义

```lua
arr = {...}
```

元表分为**正整数下标**（数组）和字符串下标（字典）



数组啥类型都可以存

#获取数组长度

访问元素方式：中括号[]



字典用法：

arr = {

​		a = 1

​		b = ...

}



### 常用API

table.insert(table_name，index, val）

table.remove(table_name, index)



## 全局表

_G表示Lua中的全局表，所有的全局变量都在 _G中

其中table也是一个全局变量（表），table.insert其实是table中的一个变量，它是一个函数



## 布尔类型

只有nil和false是false，其余为真

Lua中布尔运算不会只返回true和false，而是会返回使得其为真的内容例如下面这句

```lua
a > 10 and "yes" or "no"
```

当a>10为真，则返回yes，因为字符串本身就是true的，true和true与获得原本的yes，否则返回no



## 分支语句

语法

```lua
if condiction then
	--do something
else
	--do somthing
end
```



## 循环语句

for循环

```lua
for i = startVal,endVal,[stride] do
	--do thing
    --you can break in for!
end
```

注意，循环中修改i没用，会创建一个新的叫i的变量

while循环

```lua
while condiction do
	--do something
end
```



## 迭代器

ipairs(t) / pairs可以获得一个table的键值对

```lua
for i,j in ipairs(t) do
	-- do something
end
```

上代码可以把key赋值给i，val赋值给j

ipairs和pairs的区别在于，ipairs会关注下标的连续性

内部实际上是调用了next函数

```lua
next(t,index)
```

next会返回index后一个键值对



## 元表

元表可以用于描述原始值在一些操作下的行为，实现的方法是实现**元方法**

例如一般定义一个普通表t，将它和数值相加是没有意义的，但可以定义一个元表mt，mt中定义函数_ _add来定义加法行为，再把mt设置成t的元表（setmetatable(t,mt))，就能实现表t与数值相加的行为



## 面向对象

一个以字符串作为“下标”的table可以通过和元表配合实现面向对象的能力

