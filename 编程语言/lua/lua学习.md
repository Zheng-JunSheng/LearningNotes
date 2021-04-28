# `lua`学习

## 第一章 开始

### 1.1 程序块

​	`lua`执行的每段代码，都称为一个程序块。一个程序块也就是一连串的语句或命令。

​	几条连续的`lua`语句之间并不需要分隔符，但也可以使用分号来分隔语句。例如,以下四个程序块都是合法，并且完全等价：

```lua
a = 1
b = a * 2

a = 1;
b = a * 2;

a = 1; b = a * 2

a = 1 b = a * 2
```

​	除了将程序代码保存到一个文件中再执行，还可以在交互模式中运行解释器。当不使用任何参数而直接运行解释器时，就会看到这样的提示符：

```shell
zjs@ubuntu:~$ lua
Lua 5.3.3  Copyright (C) 1994-2016 Lua.org, PUC-Rio
> 
```

在这种模式中，输入的每条命令都将立即被执行。在交互模式下，运行程序块的方式是使用函数`dofile`，这个函数会立即执行一个文件。例如：

```shell
zjs@ubuntu:~$ lua main.lua
enter a number:
4
24
zjs@ubuntu:~$ lua
Lua 5.3.3  Copyright (C) 1994-2016 Lua.org, PUC-Rio
> dofile("main.lua")
enter a number:
4
24
> 
```

要退出交互模式和解释器，只需要输入一个end-of-file控制字符（在UNIX中为【`Ctrl`+ D】组合键，在DOS/Windows中为【`Ctrl`+ Z】组合键）。

### 1.2 词法规范

​	`Lua`中的标识符可以由任意字母、数字和下画线构成的字符串，但不能以数字开头，同时也应该避免使用以一个下画线开头并跟着一个或多个大写字母的标识符，`Lua`将这类标识符保留用作特殊用途。

​	可以在任何地方以两个连字符（--）开始一个行注释。`Lua`也提供块注释，以“--[[”开始，到"]]"结束，一个常见的技巧是将这些代码放入"--[["和"--]]"中间，这样当重新启用这段代码时，只需要在第一行行首添加一个连字符（-）即可。

### 1.3 全局变量

​	全局变量不需要声明，只需一个值赋予就可以创建。在`Lua`中，访问一个未初始化的变量不会引发错误，访问结果是一个特殊的值`nil`。
​	通常没有必要删除一个全局变量，如果一定要删除某个全局变量的话，只需要将其赋值为`nil`。

### 1.4 解释器程序

​	解释器是一个小型的程序，可以通过它来直接使用`Lua`。
​	解释器程序的用法如下：

```lua
lua [ options ] [ script [ args ] ]
```

常见参数如下：

| **-e start** | 执行start语句，如果start中含有空格，引号等特殊字符则需要使用引号括起来 |
| ------------ | ------------------------------------------------------------ |
| **-i**       | **在执行完script后进入交互模式**                             |
| **-l**       | **加载库文件**                                               |
| **-v**       | **显示版本信息**                                             |

在脚本代码中，可以通过全局变量`arg`来检索脚本的启动参数。解释器在运行脚本前，会用所有的命令行参数创建一个名为`arg`的table。

## 第二章 类型与值

在`Lua`中有8种基础类型：

| nil          | **空** ，将nil赋予一个全局变量等同于删除它                   |
| ------------ | ------------------------------------------------------------ |
| **boolean**  | **布尔**                                                     |
| **number**   | **数字，实数，Lua中没有整数类型**                            |
| **string**   | **字符串，是不可变的值，根据修改要求来创建新字符串，和table、函数一样都是自动内存管理机制所管理的对象，无须担心字符串分配和释放** |
| **userdata** | **自定义类型**                                               |
| **function** | **函数**                                                     |
| **thread**   | **线程**                                                     |
| **table**    | **表**                                                       |

函数`type`可以根据一个值返回其类型名称，`type`函数总是返回一个字符串。

长度操作符`#`用于返回一个数组或线性表的最后一个索引值。以下是几种长度操作符在`Lua`中的习惯写法：

| **print(a[#a])** | 打印列表a的最后一个值 |
| ---------------- | --------------------- |
| **a[#a] = nil**  | **删除最后一个值**    |
| **a[#a+1] = v**  | **将v添加到列表末尾** |

对于所有未初始化的元素的索引结果都是`nil`。`Lua`将`nil`作为界定数组结尾的标志。当一个数组中间有`nil`时，长度操作符会认为这些`nil`元素是结尾标记。我们可以使用函数`table.maxn`，来返回一个`table`的最大正索引数。

## 第三章 表达式

### 3.1 算术操作符

​	加减乘除，指数（^），取模（%），一元的负号（-）。

### 3.2 关系操作符

​		`< > <= >= == ~=`

所有关系操作符结果都是true或false。其中`~=`是不等性测试，如果两个值具有不同的类型，`Lua`就认为他们是不相等的。**`nil`只与其本身相等。**

### 3.3 逻辑操作符

​	逻辑操作符有`and`、`or`和`not`。所有的逻辑操作符将`false`和`nil`视为假，其他的任何东西视为真。
​	对于`and`，如果第一个操作数为假，就返回第一个操作数，否则返回第二个操作数。
​	对于`or`  ，如果第一个操作数为真，就返回第一个操作数，否则返回第二个操作数。

```lua
> print(4 and 5)
5
> print(nil and 5)
nil
> print(false and 13)
false
> print(4 or 5)
4
> print(false or 13)
13
```

操作符`not`只返回true或false。

```lua
> print(not nil)
true
> print(not false)
true
> print(not 0)
false
> print(not not nil)
false
```

### 3.4 字符串连接

在`Lua`中连接两个字符串，可以使用操作符`..`。如果其中任意一个操作数是数字的话，`Lua`会将这个数字转成一个字符串。

```lua
> print("hello " .. "world")
hello world
> print(0 .. 1)
01
```

### 3.5 优先级

![操作符优先级](C:\Users\ADMIN\Desktop\lua学习.assets\20170106090838088)

### 3.6 table构造式

​	最简单的构造式是一个空构造式{},用于创建一个空`table`。构造式还可以用于初始化数组。例如：

```lua
> days = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"}
> print(days[1])
Sunday
> print(days[2])
Monday
> print(days[4])
Wednesday
```

数组通常以1作为索引的起始值。如果真需要以0作为一个数组的起始索引的话，通过这种语法也可以轻松做到：

```lua
> days = {[0]="Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"}
> print(days[1])
Monday
> print(days[2])
Tuesday
> print(days[0])
Sunday
```

`Lua`还提供另一种特殊的语法用于初始化`table`：

```lua
a = {x = 10, y = 20}
```

等价于

```lua
a = {}; a.x = 10; a.y = 20;
```

无论使用哪种方式来创建`table`，都可以在`table`创建之后添加或删除其中的某些字段，因此所有`table`都可以构造成一样的，而构造式只是在`table`的初始化时发挥作用。

## 第四章 语句

### 4.1 赋值

​	赋值的基本含义是修改一个变量或一个`table`中字段的值：

```lua
a = "hello " .. "world"
t.n = t.n + 1
```

​	`Lua`允许多重赋值，每个值或每个变量之间以逗号分隔。例如：

```lua
a, b = 10, 2 * x
```

​	赋值后，变量a为10， 变量b为2 * x。

​	在多重赋值中，`Lua`先对等号右边的所有元素求值，然后才执行赋值。这样就可以使用多重赋值来交换两个变量，例如：

```lua
x, y = y, x
a[i], a[j] = a[j], a[i]
```

​	`Lua`赋值规则：若值的个数少于变量的个数，那多余的变量会被赋为`nil`；若值的个数更多的话，那么多余的值会被丢弃掉。

### 4.2 局部变量与块

​	相对于全局变量，`Lua`还提供了局部变量。通过`local`语句来创建局部变量：

```lua
local i = 1
```

​	与全局变量不同的是，局部变量的作用域仅限于声明他们的那个块。一个块可以是一个控制结构的执行体、或者是一个函数的执行体再或者是一个程序块。

​	`Lua`还可以显式地界定一个块，只需要将这些内容放入一对关键字**do-end**中即可。每当输入了`do`时，`Lua`不会单独地执行后面每行的内容，而是直至遇到一个相应的`end`时，才会执行整个块的内容。

​	在`Lua`中有一种写法是

```lua
local foo = foo
```

这句代码创建一个局部变量`foo`，并将用全局变量`foo`的值初始化它。这种方法可以加速在当前作用域中对`foo`的访问。

### 4.3 控制结构

#### 4.3.1 if then else

```lua
if op == "+" then
	r = a + b
elseif op == '-' then
	r = a - b
else 
	error("invalid operation")
end
```

​	注意，`Lua`不支持switch语句。

#### 4.3.2 while

```lua
local i = 1
while a[i] do
	print(a[i])
	i = i + 1
end
```

#### 4.3.3 repeat

​	一条**`repeat-until`**语句重复执行其循环体直到条件为真时结束。测试是在循环体之后做的，因此循环体至少会被执行一次。

```lua
--打印输入的第一行不为空的内容
repeat
	line = io.read
until line ~= " "
print(line)
```

#### 4.3.4 数字型for

​	`for`语句有两种形式：数字型`for`和泛型`for`。

数字型`for`语法如下：

```lua
for var = exp1， exp2， exp3 do
	<执行体>
end
```

`var`从`exp1`变化到`exp2`，每次变化都以`exp3`作为步长递增`var`，并执行一次执行体。`exp3`是可选的，若不指定的话，`Lua`会将步长默认为1.

如果不想设置循环上限的话，可以使用常量`math.huge`。

`for`循环的一些小细节。`for`的三个表达式是在循环开始前一次性求值的。其次，控制变量会被自动地声明为`for`语句的局部变量，并且仅在循环体内可见，因此，控制变量在循环结束后就不存在了。

#### 4.3.5 泛型for

​	泛型`for`循环通过一个迭代器函数来遍历所有值：

```lua
for i,v in ipairs(a) do print(v) end
```

`ipairs`是`Lua`基础库提供的，是一个用于遍历数组的迭代器函数。在每次循环中，`i`会被赋予一个索引值，`v`会被赋予一个对应于该索引的数组元素值。

与数字型`for`循环有两个相同点：

1. 循环变量是循环体的局部变量
2. 决不应该对循环变量作任何赋值

### 4.4 break与return

`break`和`return`语句用于跳出当前的块。

由于语法构造的原因，`break`和`return`应是程序块的最后一条语句，或者是`end`、`else`或`until`前的一条语句。

## 第五章 函数

一个函数定义具有一个名称、一系列的参数和一个函数体。调用函数时提供的实参数量可以与形参数量不同，若实参多余形参，则舍弃多余的实参；若实参不足，则多余的形参初始化为`nil`。形参为函数定义时参数表中的参数，实参为调用函数时传入的参数。

### 5.1 多重返回值

`Lua`允许函数返回多个结果，只需要在`return`关键字后列出所有的返回值即可。

```lua
function maximum (a)
    local mi = 1        
    local m = a[mi]       
    for i,val in ipairs(a) do
       if val > m then
           mi = i
           m = val
       end
    end
    return m, mi
end

print(maximum({8,10,23,12,5})) --->23	3
```

`Lua`会调整一个函数的返回值数量以适应不同的调用情况。如果一个函数调用不是一系列表达式的最后一个元素，那么只产生一个值。一系列表达式有4种情况：

- 多重赋值
- 函数调用时传入的实参列表
- table的构造式
- return语句

如果一个函数没有返回值或者没有返回足够多的返回值，那么`Lua`会用`nil`来补充缺失的值。

当函数调用作为另一个函数调用的最后一个（或仅有的）实参时，第一个函数的所有返回值都将作为实参传入第二个函数。

我们也可以将一个函数调用放入一对圆括号中，从而迫使它只返回一个结果。

这里还要介绍一个特殊函数`unpack`。它接受一个数组作为参数，并从下标1开始返回该数组的所有元素。

### 5.2 变长参数

参数表中的3个点(`...`)表示该函数可接受不同数量的实参。

表达式(`...`)的行为类似于一个具有多重返回值的函数，它返回的是当前函数的所有变长参数。

```lua
function add(...)  
local s = 0  
  for i, v in ipairs{...} do   --> {...} 表示一个由所有变长参数构成的数组  
    s = s + v  
  end  
  return s  
end  
print(add(3,4,5,6,7))  --->25
```

通常在遍历变长参数的时候只需要使用 **{…}**，然而变长参数可能会包含一些 **nil**，那么就可以用 **select** 函数来访问变长参数了：**select('#', …)** 或者 **select(n, …)**

- **select('#', …)** 返回可变参数的长度
- **select(n, …)**   返回 **n** 到 **select('#',…)** 的参数

调用 `select` 时，必须传入一个固定实参 `selector`(选择开关) 和一系列变长参数。如果 `selector` 为数字 n，那么 `select` 返回 n 后所有的参数，否则只能为字符串 **#**，这样 `select` 返回变长参数的总数。

### 5.3 具名实参

具名实参是指具有名称的实参。

```lua
function createPanel( opt )
    print(opt.x)
    print(opt.y)
    print(opt.width)
    print(opt.height)
    print(opt.background)
    print(opt.border)
end
createPanel({x=1,y=2,width=200,height=160,background="white",border=1}) -- 参数是匿名table
-- result:
-->1
-->2
-->200
-->160
-->white
-->1
dataTable = {x=1,y=2,width=200,height=160,background="white",border=1}
createPanel(dataTable)  --另一种写法
-- result:
-->1
-->2
-->200
-->160
-->white
-->1
```

对于参数很多的函数，有时很难记住参数的名字和参数的顺序以及哪些参数是可选的。通过table可以在调用这类函数时可以随意指定参数的顺序，并且可以只传递需要设定的参数。这就是具名实参的好处。

## 第六章 深入函数

函数是一种第一类值（First-Class Value），它们具有特定的词法域（Lexical Scoping）。

第一类值：这表示在`Lua`中函数与其他传统类型的值（例如数字、字符串）具有相同的权利。函数可以存储到变量中或`table`中，可以作为实参传递给其他函数，还可以作为其他函数的返回值。

词法域：一个函数可以嵌套在另一个函数中，内部的函数可以访问外部的函数中的变量。

函数是**值**：一个函数定义实际上就是一条语句（赋值语句），这条语句创建了一种类型为“函数”的值，并将这个值赋予给一个变量。 结果称为一个**匿名函数**

```lua
function foo(x)  return 2*x end 

foo = function(x) return 2*x end
```

### 6.1 closure（闭合函数）

在`Lua`中，一个`closure`是由一个函数和该函数会访问到的非局部变量组成的，其中非局部变量是指不是在局部作用范围内定义的一个变量，但同时又不是一个全局变量，主要应用在嵌套函数和匿名函数里。

闭合函数的主要应用：

- 作为高阶函数的参数，比如像`table.sort`函数的参数。
- 创建其他的函数的函数，即函数返回一个闭包。
- 闭包对于回调函数也非常有用。典型的例子就是界面上按钮的回调函数，这些函数代码逻辑可能是一模一样，只是回调函数参数不一样而已，即`upvalue`的值不一样而已。
- 创建一个安全的运行环境，即所谓的沙盒（sandbox）。

### 6.2 非全局的函数

函数不仅可以存储在全局变量中，还可以存储在table的字段中和局部变量中。

局部函数是将一个函数存储到一个局部变量中，该函数只能在某个特定的作用域中使用。因为`Lua`是将每个程序块作为一个函数来处理的，所以在一个程序块中声明的函数就是局部函数，这些函数只能在该程序块中可见。

对于局部函数定义

```lua
local function foo (<参数>) <函数体> end
```

`Lua`将其展开为

```lua
local foo
foo = function (<参数>) <函数体> end
```

在间接递归的情况中，必须使用一个明确的前向声明：

```lua
local f, g 

function g()
	<一些代码> f() <一些代码> 
end

function f()
	<一些代码> g() <一些代码> 
end
```

### 6.3 正确的尾调用

当一个函数调用是另一个函数的最后一个动作时，该调用才算是一条“尾调用”。

举例说明，以下是对g的一条尾调用：

```lua
function f (x) return g(x) end
```

当f调用完g之后就再无其它事情可做，程序就不需要返回那个尾调用所在的函数，也不需要保存任何关于该函数的栈信息。当g返回时，执行控制权可以直接返回到调用f的那个点上。

尾调用的判断准则：一个函数在调用完另一个函数之后，是否就无其他事情需要做。例如：

```lua
return <func>(<args>)
```

这样的调用形式才算是一条尾调用。

## 第七章 迭代器与泛型`for`

### 7.1 迭代器与closure

迭代器是一种可以遍历一种集合中所有元素的机制。在`Lua`中通常将迭代器表示为函数，每调用一次函数，即返回集合中的下一个元素。

一个`closure`是一种可以访问其外部嵌套环境中的局部变量的函数。对于`closure`而言，这些变量就可用于在成功调用之间保持状态值，从而使`closure`可以记住它在一次遍历中所在的位置，因此一个`closure`结构通常涉及到两个函数：`closure`本身和一个用于创建该`closure`的工厂（factory）函数。

```lua
function values(t) --工厂函数
local i = 0
	return function() i = i + 1 return t[i] end
end
t = {10, 20, 30}
iter = values(t)
while true do
	local element = iter()
	if nil == element then
		break
	end
	print(element)
end

for element in values(t) do
	print(element)
end
```

### 7.2 泛型`for`的语义

泛型`for`的语法如下：

```lua
for <var-list> in <exp-list> do
	<body>
end
```

`<var-list>`为一个或多个变量名的列表

`<exp-list>`为一个或多个表达式的列表，通常只有一个元素，即对迭代器工厂的调用

`for`做的第一件事就是对in后面的表达式求值。返回3个值供`for`保存：一个迭代 器函数、一个恒定状态、一个控制变量。（多则弃，少则nil补）。
在初始化步骤之后，`for`会以恒定状态和控制变量来调用迭代器函数，然后`for`将迭 代器函数的返回值赋予变量列表中的变量。如果第一个返回值为`nil`，那么循环结束，否则，`for`执行它的循环体，随后再次调用迭代器函数，并重复这个过程

### 7.3 无状态的迭代器

无状态的迭代器是一种自身不保存任何状态的迭代器。

在每次迭代中，`for`循环都会用恒定状态和控制变量来调用迭代器函数，同时迭代器可以根据这两个值来为下次迭代生成下一个元素。

`Lua`会自动将`for`循环中表达式列表的结果调整为3个值，迭代器函数，恒定状态和控制变量初始值。

### 7.4 具有复杂状态的迭代器

一个迭代器通过一个table就可以保存任意多的数据，此外在循环过程中可以改变这些数据（恒定状态总是同一个table，但是table的内容是可以改变的）。

由于这种迭代器可以在恒定状态中保存所有数据，所以它们通常可以忽略泛型for提供的第二个参数（控制变量）。

### 7.5 真正的迭代器

迭代器并没有做实际的迭代，真正做迭代的是`for`循环。迭代器只是为每次迭代提供一些成功后的返回值。

## 第八章 编译、执行与错误

`Lua`是一种解释型的语言，有能力（并且轻易地）执行动态生成的代码。

### 8.1 编译

前面提到了`dofile`函数，但实际上`dofile`是一个辅助函数，`loadfile`才做了真正核心的工作。`loadfile`会从一个文件加载`Lua`代码块，但它不会运行代码，只是编译代码，然后将编译结果作为一个函数返回。在发生错误时，`loadfile`会返回`nil`和错误消息。

函数`loadstring`与`loadfile`类似，不同之处在于它是从一个字符串中读取代码，而非从文件读取。`loadstring`总是在全局环境中编译他的字符串。
`loadstring`在`Lua`5.2中遗弃，可以使用`load`。

`load`接受一个读取器函数，并在内部调用它来获取程序块。读取器函数可以分几次返回一个程序块，`load`会反复调用它，直到他返回`nil`为止。

### 8.2 C代码

`Lua`提供的所有关于动态链接的功能都聚集在一个函数中，即`package.loadlib`。该函数有2个字符串参数：动态库的完整路径和正确的函数名称。典型的调用代码如下：

```lua
local path = "/usr/local/lib/lua/5.1/socket.so"
local f = package.loadlib(path, "luaopen_socket")
```

`loadlib` 函数加载指定的库，并将其链接入`Lua`，将C函数作为`Lua`函数的返回（并没有调用库中的任何函数）。

通常使用`require`来加载C程序库（这个函数会搜索指定的库），然后用`loadlib`来加载库，并返回初始化函数。

### 8.3 错误（error）

`Lua`所遇到的任何未预期条件都会引发一个错误，只要发生一个错误，`Lua`就会结束当前程序块并返回应用程序。

`assert`函数检查其第一个参数时候为true，若为true，则简单地返回该参数；否则（为false或nil）就引发一个错误。第二个参数是一个可选的信息字符串。

当一个函数遭遇了一种未预期的状况（异常），它可以采取2种基本的行为：返回错误代码（通常是`nil`）、引发一个错误（调用`error`）。

原则：易于避免的异常应引发一个错误，否则应返回错误代码。

一种典型的`Lua`技巧，检测文件是否打开。

```lua
file = assert(io.opean(FileName, "r"))
```

### 8.4 错误处理与异常

对于大多数应用程序而言，无须在`Lua`代码中作任何错误处理，应用程序本身会负责这类问题。如果执行中发生了错误，此调用会返回一个错误代码，这样应用程序就能采取适当的行动来处理错误。

如果需要在`Lua`中处理错误，则必须使用方式`pcall`来包装需要执行的代码。

`pcall`函数会以一种“保护模式”来调用它的第一个参数，因此`pcall`可以捕获函数执行中的任何错误。如果没有发生错误，`pcall`返回true和函数调用的返回值；否则，返回false和错误消息。

### 8.5 错误消息与追溯

在遇到一个内部错误时，`Lua`就会产生错误消息，错误消息就是传递给`error`函数的值。

`error`函数还有第二个附加参数`level`层，用于指出应由调用层级中的哪个（层）函数来报告当前的错误，也就是说明了谁应该为这错误负责。第二个附加参数默认为1，即为调用error函数的位置；2即为调用error函数的函数的位置；0则不打印出错位置信息。

## 第九章 协同程序

协同程序与线程差不多，也就是一条执行序列，拥有自己独立的栈、局部变量和指令指针，同时又与其他协同程序共享全局变量和其他大部分东西。

两者的主要区别在于，一个具有多个线程的程序可以同时运行几个线程，而协同程序需要彼此协作地运行。
也就是说，一个具有多个协同程序的程序在任意时刻只能运行一个协同程序，并正在运行的协同程序只会在其显式地要求挂起（suspend）时，它的执行才会暂停。

### 9.1 协同程序基础

`Lua`将所有有关于协同程序的函数放置在一个名为“coroutine”的`table`中。函数`create`用于创建新的协同程序，它只有一个参数（一个函数）。该函数的代码就是协同程序所需执行的内容。`create`会返回一个`thread`类型的值，用以表示新的协同程序。例如：

```lua
co=coroutine.create(function() print("hi") end)

print(co) -->thread: 0x563b7f819678
```

一个协同程序有四种状态：

1. 挂起状态
2. 运行状态
3. 死亡状态
4. 正常状态：当一个协同程序A唤醒另一个协同程序B时，A就处于一个特殊状态（正常状态normal），既不是挂起状态（无法继续A的执行），也不是运行状态（B在运行）。

当创建一个协同程序时，它处于挂起状态。可以通过函数`status`来检查协同程序的状态。

```lua
print( coroutine.status( co ))  -->suspended
```

函数`coroutine.resume`用于启动或者再次启动一个协同程序的执行，并将状态从挂起改为运行。

```lua
coroutine.resume(co) -->hi
```

在打印完hi后，协同程序就处于死亡状态。

```lua
print( coroutine.status( co ))  -->dead
```

`Lua`的协同程序可以通过一对`resume-yield`来交换数据。

在第一次调用`resume`时，并没有对应的`yield`在等待它，因此所有传递给`resume`的额外参数都将视为协同程序主函数的参数。例如：

```lua
co = coroutine.create(function(a, b, c)
　　		print("co", a, b, c)
　　		end)
coroutine.resume(co, 1, 2， 3)	-->co 1 2 3
```

在`resume`调用返回的内容中，第一个值为true 表示没有错误，而后面所有的值都是对应`yield`传入的参数。例如：

```lua
co = coroutine.create(function(a, b)
		coroutine.yield(a + b, a - b)
		end)
print("co", coroutine.resume(co, 10, 20)) 	-->co  true  30  -10
```

`yield`返回的额外值就是对应`resume`传入的参数。例如：

```lua
coc = coroutine.create(function()
		print("co", coroutine.yield())
		end)
coroutine.resume(co)
coroutine.resume(co, 4, 5)	-->co 4 5
```

当一个协同程序结束时，它的主函数所返回的值都将作为对应的`resume`的返回值。例如：

```lua
co = coroutine.create(function()
		return 6, 7
		end)
print(coroutine.resume(co)) -->true 6 7
```

非对称的协同程序（asymmetric coroutine）：`Lua`提供了两个函数来控制协同程序的执行，一个用于挂起执行，另一个用于恢复执行。

对称的协同程序(symmetric coroutine) ：只有一个函数用于转让协同程序之间的执行权。

### 9.2 管道与过滤器

一个关于协同程序的经典示例就是“生产者-消费者”的问题。

```lua
function producer()
　　while true do 
　　local x = io.read()  --产生新的值
　　send(x)			--发送给消费者
　　end
end
function consumer()
　　while true do 
　　local x = receive()  --从生产者接收值
　　io.write(x, "\n")	 --消费新的值
　　end
end
```

协同程序被称为是一种匹配生产者和消费者的理想工具，一对`resume-yield`完全一改典型的调用者与被调用者之间的关系。当一个协同程序调用`yield`时，它不是进入一个新的函数，而是从一个悬而未决的`resume`调用中返回；同样对于`resume`的调用也不会启动一个新函数，而是从一次`yield`调用中返回。

消费者驱动（consumer-driven）模式：当消费者需要一个新值时，它唤醒生产者。生产者返回一个新值后停止运行，并等待消费者的再次唤醒。

```lua
function receive()
　　local statue, value = coroutine.resume(producer)
　　return value
end
function send(x)
　　coroutine.yield(x)
end
```

过滤器（filter）：是一种位于生产者和消费者之间的处理功能，可用于对数据的一些交换。过滤器既是一个消费者也是一个生产者，它唤醒一个生产者促使其产生新值，然后又将变换后的值传递给消费者。

### 9.3 以协同程序实现迭代器

遍历某个数组的所有排列组合的形式：

```lua
function permgen(a, n)
	n = n or #a
	if n <= 1 then
		coroutine.yield(a)		
	else
		for i = 1, n do
			a[n], a[i]=a[i], a[n]  --将第i个元素放到数组末尾
			permgen(a, n-1)	       --递归调用
			a[n], a[i]=a[i], a[n]  --恢复第i个元素
		end
	end
end

function printResult(a)
	for i = 1, #a do
		io.write(a[i], " ")
	end
	io.write("\n")
end

function permutations(a)
	local co = coroutine.create(function() permgen(a) end)
	return function()			                            --迭代器
		local code, res = coroutine.resume(co)               --开启
		return res		                                    --返回的res就是传入到函数permgen中的参数a
	end
end

for p in permutations({1, 2, 3}) do
	printResult(p)	--调用打印函数
end
```

上述代码运行结果：

```
2 3 1 
3 2 1 
3 1 2 
1 3 2 
2 1 3 
1 2 3 
```

定义一个工厂函数`permutations`，用于将生成函数放到一个协同程序中运行，并出具迭代器函数（只是简单地唤醒协同程序）。函数`permutations`是将一条唤醒协同程序的调用包装在一个函数中。`Lua`专门提供了一个函数`coroutine.wrap`来完成这个功能。类似于`create`，`wrap`创建了一个新的协同程序，不同的是，`wrap`并不是返回协同程序本身，而是返回一个函数。每当调用这个函数时，唤醒一次协同程序。若使用`wrap`，上述代码中的`permutations`可以这么写：

```lua
function permutations(a)
	return coroutine.wrap(function() permgen(a) end)
end
```

### 9.4 非抢占式的多线程

协同程序提供了一种协作式的多线程，每个协同程序都等于是一个线程。一对`yield-resume`可以将执行权在不同线程之间切换。协同程序与常规的多线程的不同之处：当一个协同程序运行时，是无法从外部停止它的，只有当协同程序显式地要求挂起时（调用`yield`），它才会停止。

对于非抢占式的多线程来说，只要一个线程调用一个阻塞的（blocking）操作，整个程序在该操作完成前都会停止下来。但是这种行为对于大多数应用程序来说的无法接受的。

典型的多线程使用情况：通过HTTP下载几个远程的文件。

```lua
　function download(host, file)
　　	local c = assert(socket.connect(host, 80))
　　	local count = 0
　　	c:send("GET "..file.." HTTP/1.0\r\n\r\n")
　　	while true do
　　		local s, status, partial = receive(c)
　　		count = count + #(s or partial)
　　		if status == "closed" then
　　			break
　　		end
　　	end
　　	c:close()
　　	print(file, count)
　　end
```

## 

