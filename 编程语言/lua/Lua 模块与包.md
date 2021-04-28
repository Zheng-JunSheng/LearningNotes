# `Lua` 模块与包

从`Lua5.1`版本开始，就对模块和包添加了新的支持，可是使用`require`和`module`来定义和使用模块和包。`require`用于使用模块，`module`用于创建模块。简单的说，一个模块就是一个程序库，可以通过`require`来加载。然后便得到了一个全局变量，表示一个`table`。这个`table`就像是一个命名空间，其内容就是模块中导出的所有东西，比如函数和常量。

## 1 require函数

`require`函数的调用形式为`require "<模块名>"`。该调用会返回一个由模块函数组成的`table`，并且还会定义一个包含该`table`的全局变量。在使用`Lua`中的标准库时可以不用显示的调用require，因为`Lua`已经预先加载了他们。

require函数在搜素加载模块时，有一套自定义的模式，如：

```lua
?;?.lua;c:/windows/?;/usr/local/lua/?/?.lua
```

在上面的模式中，只有问号(?)和分号(;)是模式字符，分别表示`require`函数的参数(模块名)和模式间的分隔符。如：调用`require "sql"`，将会打开以下的文件：

```lua
sql
sql.lua
c:/windows/sql
/usr/local/lua/sql/sql.lua
```

`Lua`将`require`搜索的模式字符串放在变量**`package.path`**中。当`Lua`启动后，便以环境变量`LUA_PATH`的值来初始化这个变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。如果`require`无法找到与模块名相符的`Lua`文件，就会找C程序库。**C程序库的搜索模式存放在变量`package.cpath`中。**而这个变量则是通过环境变量`LUA_CPATH`来初始化的。

## 2 编写模块的基本方法

新建一个文件，命名为`game.lua`，代码如下：

```lua
local M = {};
local modelName = ...;
_G[modelName] = M;
function M.play()
    print("start");
end
function M.quit()
    print("end");
end
return M;
```

加载`game.lua`,使用如下

```
Lua 5.3.3  Copyright (C) 1994-2016 Lua.org, PUC-Rio
> game = require "game"
> game.play()
start
```

## 3 使用环境

仔细阅读上例中的代码，我们可以发现一些细节上问题。比如模块内函数之间的调用仍然要保留模块名的限定符，**如果是私有变量还需要加local关键字，同时不能加模块名限定符。如果需要将私有改为公有，或者反之，都需要一定的修改。**那又该如何规避这些问题呢？我们可以通过`Lua`的函数“全局环境”来有效的解决这些问题。

我们把`game.lua`这个模块里的全局环境设置为`M`，于是，我们直接定义函数的时候，不需要再带`M`前缀。
因为此时的全局环境就是`M`，不带前缀去定义变量，就是全局变量，这时的全局变量是保存在`M`里。所以，实际上，`play`和`quit`函数仍然是在`M`这个`table`里。

```lua
local M = {};
local modelName = ...;
_G[modelName] = M;
package.loaded[modname] = M
setfenv(1, M);
function M.play()
    print("start");
end
function M.quit()
    print("end");
end
return M;
```

## 4 module函数

前面的几个实例中，它们都以相同的模式开始：

```lua
local modname = ...
local M = {}
_G[modname] = M
package.loaded[modname] = M
     --[[
     和普通Lua程序块一样声明外部函数。
     --]]
setfenv(1,M)
```

我们可以用module(...)函数来代替以上代码，如：

```lua
module(..., package.seeall);

function M.play()
    print("start");
end
function M.quit()
    print("end");
end
return M;
```

默认情况下，`module`不提供外部访问，必须在调用它之前，为需要访问的外部函数或模块声明适当的局部变量。然后`Lua`提供了一种更为方便的实现方式，即在调用module函数时，多传入一个`package.seeall`的参数，相当于 `setmetatable(M, {__index = _G}) `。例如：`module(..., package.seeall)`

## 5 子模块与包

`Lua`支持具有层级性的模块名，可以用一个点来分隔名称中的层级。假设一个模块名为`mod.sub`，那么它就是mod的一个子模块。因此，可以认为模块`mod.sub`会将其所有值都定义在`table mod.sub`中，也就是一个存储在`table mod`中，且`key`为`sub`的`table`。就好比下述的定义：

```lua
local mod = {sub = {}}
```

当`require`一个模块`mod.sub`时，`require`会用原始的模块名`mod.sub`作为key来查询`table package.loaded`和`package.preload`，其中，模块名中的点在搜索时没有任何意义。但是，当搜索一个定义子模块的文件时，`require`会将点转换成另一个字符，通常就是系统的目录分隔符，转换之后`require`就像搜索其他名称一样来搜索这个名称。假如路径为以下字符串：

```lua
?;?.lua;/usr/local/lua/?.lua;/usr/local/lua/?/init.lua
```

并且目录分隔符为“/”，那调用`require "a.b"`会尝试打开以下文件：

```
./a/b.lua
/usr/local/lua/a/b.lua
/usr/local/lua/a/b/init.lua
```

