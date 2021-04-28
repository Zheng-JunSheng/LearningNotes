# `Lua` 元表与元方法

​	`Lua` 中的每个值都有一个元表。`table` 和 `userdata` 可以有各自独立的元表，而其他类型的值则共享其类型所属的单一元表。任何 `table` 都可以作为任何值的元表，而一组相关的 table 也可以共享一个通用的元表。一个 `table` 甚至可以作为它自己的元表。

​	在`Lua`代码中，只能设置`table`的元表。若要设置其他类型的值的元素，则必须通过C代码来完成的。

## 1 关系类与算术类的元方法

​	在元表中，每种算术操作符都有对应的字段名。(注意，`__`是两个下划线)

|    模式    |        描述        |
| :--------: | :----------------: |
|  `__add`   | 对应的运算符 '+'.  |
|  `__sub`   | 对应的运算符 '-'.  |
|  `__mul`   | 对应的运算符 '*'.  |
|  `__div`   | 对应的运算符 '/'.  |
|  `__mod`   | 对应的运算符 '%'.  |
|  `__unm`   | 对应的运算符 '-'.  |
| `__concat` | 对应的运算符 '..'. |
|   `__eq`   | 对应的运算符 '=='. |
|   `__lt`   | 对应的运算符 '<'.  |
|   `__le`   | 对应的运算符 '<='. |

`__eq`、`__lt` 和`__le`是关系类的元方法，其余是算术类的元方法。

现在我使用完整的实例代码来详细的说明算术类元方法的使用。

```lua
Set = {}
local mt = {} -- 集合的元表
 
-- 根据参数列表中的值创建一个新的集合
function Set.new(l)
    local set = {}
    for _, v in pairs(l) do set[v] = true end
     return set
end
 
-- 并集操作
function Set.union(a, b)
    local retSet = Set.new{} -- 此处相当于Set.new({})
    for v in pairs(a) do retSet[v] = true end
    for v in pairs(b) do retSet[v] = true end
    return retSet
end
 
-- 交集操作
function Set.intersection(a, b)
    local retSet = Set.new{}
    for v in pairs(a) do retSet[v] = b[v] end
    return retSet
end
 
-- 打印集合的操作
function Set.toString(set)
     local tb = {}
     for e in pairs(set) do
          tb[#tb + 1] = e
     end
     return "{" .. table.concat(tb, ", ") .. "}"
end
 
function Set.print(s)
     print(Set.toString(s))
end
```

现在定义了“+”来计算两个集合的并集，那么就需要让所有用于表示集合的`table`共享一个元表，并且在该元表中定义如何执行一个加法操作。首先创建一个常规的`table`，准备用作集合的元表，然后修改`Set.new`函数，在每次创建集合的时候，都为新的集合设置一个元表。代码如下：	

```lua
 -- 根据参数列表中的值创建一个新的集合
function Set.new(l)
    local set = {}
     setmetatable(set, mt)
    for _, v in pairs(l) do set[v] = true end
     return set
end
```

在此之后，所有由`Set.new`创建的集合都具有一个相同的元表，例如：

```lua
local set1 = Set.new({10, 20, 30})
local set2 = Set.new({1, 2})
print(getmetatable(set1))
print(getmetatable(set2))
assert(getmetatable(set1) == getmetatable(set2))
```

最后，我们需要把元方法加入元表中，代码如下：

```lua
mt.__add = Set.union
```

之后求两个集合的并集，`Lua`就会自动的调用`Set.union`函数，并将两个操作数作为参数传入。例如：

```lua
local set1 = Set.new({10, 20, 30})
local set2 = Set.new({1, 2})
local set3 = set1 + set2
Set.print(set3)
```

在上面列举的那些可以重定义的元方法都可以使用上面的方法进行重定义。现在就出现了一个新的问题，`set1`和`set2`都有元表，那我们要用谁的元表啊？虽然我们这里的示例代码使用的都是一个元表，但是实际中，会遇到这里说的问题，对于这种问题，`Lua`会按照以下步骤来查找元表：

1. 对于二元操作符，如果第一个操作数有元表，并且元表中有所需要的字段定义，比如我们这里的`__add`元方法定义，那么`Lua`就以这个字段为元方法，而与第二个值无关；
2. 对于二元操作符，如果第一个操作数有元表，但是元表中没有所需要的字段定义，比如我们这里的`__add`元方法定义，那么`Lua`就去查找第二个操作数的元表；
3. 如果两个操作数都没有元表，或者都没有对应的元方法定义，`Lua`就引发一个错误。

比如执行`set3 = set1 + 8`这样的代码，就会打印出以下的错误提示：

```lua
test.lua:16: bad argument #1 to 'for iterator' (table expected, got number)
```

但是，我们在实际编码中，可以按照以下方法，弹出我们定义的错误消息，代码如下：

```lua
function Set.union(a, b)
     if getmetatable(a) ~= mt or getmetatable(b) ~= mt then
          error("metatable error.")
     end
 
    local retSet = Set.new{} -- 此处相当于Set.new({})
    for v in pairs(a) do retSet[v] = true end
    for v in pairs(b) do retSet[v] = true end
    return retSet
end
```

关系类的元方法和算术类的元方法的定义是类似的，与算术类元方法不同的是，关系类的元方法不能应用于混合的类型。对于混合类型而言，关系类元方法的行为就模拟这些操作符在`Lua`中的普通行为。如果将一个字符串与数字比较，`Lua`就会引发一个错误。

等于比较永远不会引发错误。但是如果两个对象拥有不同的元方法，那么等于操作不会调用任何一个元方法，而是直接返回`false`。

## 2 库定义的元方法

各种程序库在元表中定义它们自己的字段是很普通的方法。函数`tostring`就是一个典型的实例，它能将各种类型的值表示为一种简单的文本格式。

函数`print`总是调用`tostring`来进行格式化输出，当格式化任意值时，`tostring`会检查该值是否有一个`__tostring`的元方法，如果有这个元方法，`tostring`就用该值作为参数来调用这个元方法，剩下实际的格式化操作就由`__tostring`元方法引用的函数去完成，该函数最终返回一个格式化完成的字符串。例如以下代码：

```lua
mt.__tostring = Set.toString
```

之后，调用`print`打印集合，`print`就会调用`tostring`函数，进而调用`Set.tostring`。

## 3 保护元素

我们会发现，使用`getmetatable`就可以很轻易的得到元表，使用`setmetatable`就可以很容易的修改元表，那这样做的风险是不是太大了，那么如何保护我们的元表不被篡改呢？在`Lua`中，函数`setmetatable`和`getmetatable`函数会用到元表中的一个字段，用于保护元表，该字段是`__metatable`。当我们想要保护集合的元表，是用户既不能看也不能修改集合的元表，那么就需要使用__`metatable`字段了。当设置了该字段时，`getmetatable`就会返回这个字段的值，而`setmetatable`则会引发一个错误，如以下演示代码：

```lua
mt.__metatable = "You cannot get the metatable"

set1 = Set.new{}
print(getmetatable(set1)) -->You cannot get the metatable
setmetatable(s1, {})      -->会引发一个错误
```

## 4 table的元方法

### 4.1 `__index`

默认情况下，当我们访问一个`table`中不存在的字段时，得到的结果是`nil`。但是这种状况很容易被改变；`Lua`是按照以下的步骤决定是返回`nil`还是其它值得：

1. 当访问一个`table`的字段时，如果`table`有这个字段，则直接返回对应的值；
2. 当`table`没有这个字段，则会促使解释器去查找一个叫`__index`的元方法，接下来就就会调用对应的元方法，返回元方法返回的值；
3. 如果没有这个元方法，那么就返回`nil`结果。

下面通过一个实际的例子来说明`__index`的使用。假设要创建一些描述窗口，每个`table`中都必须描述一些窗口参数，例如颜色，位置和大小等，这些参数都是有默认值得，因此，我们在创建窗口对象时可以指定那些不同于默认值得参数。

```lua
-- 创建一个命名空间
Windows = {} 
 
-- 创建默认值表
Windows.default = {x = 0, y = 0, width = 100, height = 100, color = {r = 255, g = 255, b = 255}}

-- 创建元表
Windows.mt = {} 
 
-- 声明构造函数
function Windows.new(o)
     setmetatable(o, Windows.mt)
     return o
end
 
-- 定义__index元方法
Windows.mt.__index = function (table, key)
     return Windows.default[key]
end
 
local win = Windows.new({x = 10, y = 10})
print(win.x)              -->10  访问自身已经拥有的值
print(win.width)          -->100 访问default表中的值
print(win.color.r)        -->255 访问default表中的值
```

根据上面代码的输出，结合上面说的那三步，我们再来看看，`print(win.x)`时，由于`win`变量本身就拥有`x`字段，所以就直接打印了其自身拥有的字段的值；`print(win.width)`，由于win变量本身没有`width`字段，那么就去查找是否拥有元表，元表中是否有`__index`对应的元方法，由于存在`__index`元方法，返回了default表中的width字段的值，`print(win.color.r)`也是同样的道理。

在实际编程中，`__index`元方法不必一定是一个函数，它还可以是一个`table`。当它是一个函数时，`Lua`以`table`和不存在`key`作为参数来调用该函数，这就和上面的代码一样；当它是一个`table`时，`Lua`就以相同的方式来重新访问这个`table`，所以上面的代码也可以是这样的：

```
Windows.mt.__index = Windows.default
```

如果不想在访问一个`table`时，涉及到它的`__index`元方法，可以使用函数`rawget`。

### 4.2 `__newindex`

`__newindex`元方法与`__index`类似，__`newindex`用于更新`table`中的数据，而`__index`用于查询`table`中的数据。当对一个`table`中不存在的索引赋值时，在`Lua`中是按照以下步骤进行的：

`Lua`解释器先判断这个table是否有元表：

1. 如果有了元表，就查找元表中是否有`__newindex`元方法；如果没有元表，就直接添加这个索引，然后对应的赋值；
2. 如果有这个`__newindex`元方法，`Lua`解释器就执行它，而不是执行赋值；
3. 如果这个`__newindex`对应的不是一个函数，而是一个`table`时，`Lua`解释器就在这个table中执行赋值，而不是对原来的`table`。