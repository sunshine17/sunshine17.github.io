---
layout: post
category: "thinking_in_ramda"
title:  "[TIR-04]Ramda思维：声明式编程思想"
tags: [nodejs,functional programming]
summary: "这是一个关于函数式编程的系列文章，使用nodejs的ramda库来讲解如何在nodejs这门目前最流行的语言实现函数式编程的理念。"
---

这是[【Ramda思维】](/posts/thinking_in_ramda.html)这个讨论函数式编程思想系列文章的第**04**篇。

**\([英文原文链接在此](http://randycoulman.com/blog/2016/06/14/thinking-in-ramda-declarative-programming/ "英文原文")\)**

在[上一篇(第3篇)](thinking_in_ramda/thinking-in-ramda-Partial-Application.html)中，我们讨论了如何使用Currying及部分应用函数（partial application）的技术来组合那些不只一个参数的函数。

在我们开始写一些短小的函数式的程序块并将其组合时，你会发现我们需要写很多函数来封装JavaScript语言的基本运算操作，例如算术运算、比较、逻辑及流程控制。写这些函数很枯燥乏味，但幸好Ramda这个FP库会帮我们解决这个烦恼。

但是，我们需要了解一些背景知识。


# **声明式编程 VS 指令式编程**

有很多方法来分类各种编程语言/风格。例如静态类型vs动态类型、解释型语言vs编译型　语言、底层vs高级语言...等等。

另一个类似的划分方式为声明式vs指令式编程范式。

本文并不深入探讨所有的编程范式，简单地说，指令式编程范式是这样一种编程风格：由程序员告诉电脑要做什么，并且明确写清楚如何去做。这样就导致了我们日常有一大堆构建元素：控制流(`if`-`then`-`else`语句及循环结构)，算术运算符(`+`,`-`,`*`,`/`)，比较运算符(`===`,`>`,`<`,等等)，还有逻辑运算符(`&&`,`||`,`!`)。

而声明式编程范式则是：程序员告诉电脑要做什么，但只告诉电脑人类想要的是什么（不像指令式连处理步骤都要写清楚）。然后电脑需要自己去思考如何计算出结果。

其中一个经典的声明式编程语言叫Prolog。在Prolog语言里，一个程序由一组事实及推理规则组成。通过提问题来开始Prolog的程序，然后Prolog的推理引擎使用你定义的事实及推理法则来推理出你问题的答案。

函数式编程也是声明式编程范式的其中一个子集。在一个函数式程序中，我们定义一系列的函数，然后通过组合这些函数来告诉电脑我们想要什么。

当然，即使在声明式的程序里，也需要做类似指令式程序里的简单任务。控制流程、算术运算、比较、逻辑分支等仍然是声明式编程里的基本构建元素。但我们会用声明式的风格来表达以上构建元素。



# **声明式的构建元素**

由于我们使用的Javascript是一种指令式的编程语言，所以当我们在写“常规”的Javascript代码时，使用标准的指令构建元素也是可以的。

但当我们在使用`pipeline`之类的“管道”或类似的构造元素来写函数式的变换时，指令式的构造元素就不太适用了。


下面我们看看Ramda提供了什么基本构建元素给我们来解决这个问题。



## **数学运算**

回到[第２章](/thinking_in_ramda/thinking-in-ramda-combining-functions.html)，我们实现了一系列的数学运算变换来演示何为“管道”(pipeline)：


```javascript
// 数学运算的管道

const multiply = (a, b) => a * b
const addOne = x => x + 1
const square = x => x * x
 
const operate = pipe(
  multiply,
  addOne,
  square
)
 
operate(3, 4) // => ((3 * 4) + 1)^2 => (12 + 1)^2 => 13^2 => 169
```

在上面的示例里，特别要留意的是我们是如何实现那些基本数学运算函数的。

其实Ramda为数学运算提供了`add`,`subtract`,`multiply`及`divide`函数。所以我们可以直接使用Ramda提供的`multiply`来替换上面自定义的对应函数，还能利用Ramda提供的Curry化的`add`函数来代替上面的`addOne`，而`multiply`其实也能使用`square`来替代之：


```javascript
// 使用Ramda的数学运算函数

const square = x => multiply(x, x)
 
const operate = pipe(
  multiply,
  add(1),
  square
)
```

`add(1)`与我们熟知的自增运算符(`++`)很相似，不同之处在于`++`会修改被自增的变量的值，所以它是会变异的。我们在[第0１篇](thinking_in_ramda/thinking-in-ramda-Wrap-Up.html)提到，”不变异“是函数式编程的一个核心原则，所以我们一般不使用`++`或类似的`--`。

我们一般使用`add(1)`及`subtract(1)`来做递增及递减的操作，而这两个操作很常要用到，所以Ramda分别提供了`inc`及`dec`给我们使用。

因此，现在可以把我们的管道再进一步简化如下：

```javascript
// 使用Ramda的inc函数

const square = x => multiply(x, x)
 
const operate = pipe(
  multiply,
  inc,
  square
)
```

`subtract`是二元运算符`-`的对应函数，但同时我们知道还有个取负数的一元运算符`-`。当然也可以使用`multiply(-1)`来实现取负数，但Ramda也提供了`negate`函数来实现此操作。


## **比较**

同样地，在[第２篇](/thinking_in_ramda/thinking-in-ramda-combining-functions.html)时，我们写了一些函数来计算一个人是否有投票权。当时最终版的代码如下：


```javascript
// 是否有投票权

const wasBornInCountry = person => person.birthCountry === OUR_COUNTRY
const wasNaturalized = person => Boolean(person.naturalizationDate)
const isOver18 = person => person.age >= 18
 
const isCitizen = either(wasBornInCountry, wasNaturalized)
 
const isEligibleToVote = both(isOver18, isCitizen)

```

上面的代码里部分函数还是使用标准的比较运算符（`===`及`>=`）。你可能已经猜到，Ramda同样也提供了对应的函数来替代它们。

下面我们使用`equals`及`gte`函数来分别替换上面的`===`及`>=`。

```javascript
// 使用Ramda的比较函数

const wasBornInCountry = person => equals(person.birthCountry, OUR_COUNTRY)
const wasNaturalized = person => Boolean(person.naturalizationDate)
const isOver18 = person => gte(person.age, 18)
 
const isCitizen = either(wasBornInCountry, wasNaturalized)
 
const isEligibleToVote = both(isOver18, isCitizen)

```

另外Ramda也提供了`gt`替代`>`、`lt`替代`<`，还有`lte`替代`<=`。

这些函数的参数顺序似乎与我们熟悉的一样（第１个参数大于第２个参数？）。单独使用的时候当然没有问题，但当需要做函数组合的时候就有点问题了。这些函数的设计似乎违反了Ramda的“数据放在最后一个参数”的原则，所以当我们在管道或类似场景使用这些函数时就需要多加注意。这个时候，`flip`及点位符函数（`__`）就能派上用场了。

同样地，除了`equals`外，Ramda还提供了一个`identical`函数来判断两个值是否在同一个内存引用。


`===`有很多常见的使用场景：例如检测一个字符串或数组是否为空（` str === '' ` 或 `arr.length === 0 `），再例如检测一个变量是否为`null`或`undefined`。Ramda同样地提供了相应的函数：`isEmpty`及`isNil`。


## **逻辑运算**


在[第２篇](/thinking_in_ramda/thinking-in-ramda-combining-functions.html)（或就在上面），我们使用`both`及`either`函数来替代`&&`及`||`运算符。也提及到使用`complement`来替代`!`。


当我们要组合处理的是同一个值时，这些组合起来的函数运作得很好。在上例中，`wasBornInCountry`、`wasNaturalized`及`isOver18`都是对一个人进行操作运算。


但有时我们需要分别应用`&&`、`||`及`!`运算符到不同的值。为了处理这种情况，Ramda提供了`and`、`or`及`not`函数。我用这个方式来区分这些函数：`and`、`or`及`not`操作数值，而`both`、`either`及`complement`操作的是函数。


`||`的常见用法是提供默认值。例如，我们经常会这样写：


```javascript

const lineWidth = settings.lineWidth || 80

```


这是一个常见的写法，而且可用，但这种写法依赖于JavaScript的"逻辑假"定义。如果`0`是一个有效的设定呢？由于`0`在JavaScript的逻辑运算里会被视作“假”，所以上例的结果是得到一个lineWidth值为80。

当然我们可以使用刚刚提到的`isNil`函数来解决这个问题，但其实Ramda提供了另一个更优雅的选择：`defaultTo`。

```javascript
// 使用defaultTo函数

const lineWidth = defaultTo(80, settings.lineWidth)
```

`defaultTo`会使用`isNil`来检测第２个参数是否为空，如果非空，则返回这个参数的值，否则返回第１个参数的值（默认值）。



# **条件分支**


其实在函数式的程序里，流程控制并不太必要，但偶尔还是会用到。我们在[第１篇博文](http://4377.myds.me:8085/thinking_in_ramda/thinking-in-ramda-Wrap-Up.html)里讨论到的集合迭代函数会处理大部分的循环场景，但条件分支还是相当重要的。


## IFELSE

下面我们写一个函数，`forever21`，这个函数只有一个参数"age"，返回下一年的岁数。但是正如函数名所暗示的，如果参数值大于或等于21，则永远只返回21.

```javascript
// 永远21岁

const forever21 = age => age >= 21 ? 21 : age + 1

```

上面我们的条件(` age >= 21 `)及第２个分支(` age + 1 `)其实可以写成` age `的函数。可以把第一个分支(`21`)重写成一个常量函数(` () => 21 `)。现在，我们有３个函数都接受（或忽略）一个`age`参数。


这时，我们就可以使用Ramda的`ifElse`函数，这个函数与我们习惯的`if...then...else`结构的作用一样，或类似三元操作符（`?:`）。


```javascript
// 使用 ifElse

const forever21 = age => ifElse(gte(__, 21), () => 21, inc)(age)

```

上面也提到，当使用比较函数来组合函数时，有时结果并非如我们所想，所以这里需要引入占位符(`___`)。另外我们也可以使用`lte`来实现：

```javascript
// 使用 lt 来替代 gte

const forever21 = age => ifElse(lte(21), () => 21, inc)(age)
```

上例代码可这样理解："21小于或等于age"。后面我们会坚持使用占位符的版本，因为该版本能提高代码可读性及减少困惑。


## **常量函数**


在上面的场景里，常量函数就很有用了。Ramda提供了一个简短的常量函数：`always`。

```javascript
// 使用always常量函数

const forever21 = age => ifElse(gte(__, 21), always(21), inc)(age)
```

Ramda也提供了另外两个快捷函数`T`及`F`来替代`always(true)`及`always(false)`。



## **INDENTITY函数**


现在看看另一个函数，`alwaysDrivingAge`。这个函数只有一个age参数，当age大于等于16时，函数返回值为输入参数。但当age小于16时，函数返回16。这个函数其实就相当于让人可以假装自己到了能考驾照的年龄（即使实际上未到）。

```javascript
// 驾照年龄

const alwaysDrivingAge = age => ifElse(lt(__, 16), always(16), a => a)(age)
```

这个条件的第２个分支(` a => a `)是另一个在函数式编程范式中常见的模式。这叫“身份函数”。即这个函数永远原样返回输入的参数值。


你应该能猜到了，Ramda同样提供了一个`identity`函数。

```javascript
// 使用indentity函数

const alwaysDrivingAge = age => ifElse(lt(__, 16), always(16), identity)(age)
```

`identity`能接受多个参数，但只返回第一个参数的值。如果我们想返回的是其它第n个参数的值，可以使用`nthArg`函数。但`nthArg`没有`identity`常用。



## **WHEN 及 UNLESS**

在一个`ifElse`语句里有一个分支是`identity`的情况其实很常见，所以Ramda为此场景也提供了一个快捷函数。


在我们的例子中，第２个分支是`identity`时，可以使用`when`来替代`ifElse`：

```javascript
// 使用when

const alwaysDrivingAge = age => when(lt(__, 16), always(16))(age)
```

如果第一个分支是`identity`，则可使用`unless`。如果把我们的条件分支反过来写，可以使用`unless`来替代`gte(__, 16)`：

```javascript
// 使用 unless

const alwaysDrivingAge = age => unless(gte(__, 16), always(16))(age)
```


## **COND函数（替代switch）**

Ramda提供了`cond`函数来替代`switch`语句（或`if...then...else`链式语句）。

下面使用Ramda文档里的例子来介绍这个`cond`的使用方法：

```javascript
// 使用cond

const water = temperature => cond([
  [equals(0),   always('water freezes at 0°C')],
  [equals(100), always('water boils at 100°C')],
  [T,           temp => `nothing special happens at ${temp}°C`]
])(temperature)
```

对我而言，在我写的Ramda代码里还没正式用过这个函数，如果你写过多年Common Lisp程序的话，这个`cond`对你而言应该就很熟悉了。



# **总结**

本文介绍的一系列Ramda函数其实都是为了使我们原来指令式的代码变成声明式的函数式风格。


# **下一篇**

最后举的几个自定义函数的例子（`forever21`、`drivingAge`及`water`）都是接受一个参数，构建出一个新的函数，然后应用这个函数到一个参数。


这就是一个常见的模式，而且Ramda提供了相关的工具函数来提高代码可读性。在下一篇，无指针风格将会讲解如何用这个模式去进一步地简化我们的函数。






