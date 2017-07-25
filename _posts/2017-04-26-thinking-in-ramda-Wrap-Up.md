---
layout: post
category: "thinking_in_ramda"
title:  "Ramda思维：开篇"
tags: [nodejs,functional programming]
summary: "这是一个关于函数式编程的系列文章，使用nodejs的ramda库来讲解如何在nodejs这门目前最流行的语言实现函数式编程的理念。"
---

>
这是一个关于函数式编程的系列文章，主要讨论如何用nodejs这门目前最流行的语言通过使用函数式编程的理念来写出高可读性、健壮而简洁的程序代码。

**\([英文原文链接在此](http://randycoulman.com/blog/2016/05/24/thinking-in-ramda-getting-started/ "英文原文链接在此")\)**

本文是这个系列文章的开篇，此系列讨论的是nodejs语言下的函数式编程(以下简称FP,即Functional Programming)，名为【Ramda编程思想】。

在这个系列文章里，我会使用[Ramda](http://ramdajs.com/ "Ramda")这个js库来讨论FP，[Ramda](http://ramdajs.com/ "Ramda")这个库与其它流行的js库\(例如[Underscore](http://underscorejs.org/ "Underscore")及[Lodash](https://lodash.com/ "lodash")\)或其它在语言层面就有FP特性的语言中的FP概念其实是相通的。

我会尽量只讨论轻量级、不那么学术性质那一面的FP，因为我希望这个系列能让大部分开发者都能轻易阅读，当然也有部分原因是我自己在FP这个领域也并未有太深的造诣。

## **RAMDA**

其实之前我已经在博客里多次介绍过[Ramda](http://ramdajs.com/ "Ramda")这个Javascript库了：

- [《在Redux应用中使用Ramda》](http://randycoulman.com/blog/2016/02/16/using-ramda-with-redux/)一文中，我介绍了在写Redux应用时如何在各种上下文环境中应用Ramda库。
- [《在Rails程序中使用Redux-api-middleware》](http://randycoulman.com/blog/2016/04/19/using-redux-api-middleware-with-rails/)一文中，我使用Ramda来处理请求(Request)与响应(Response)的数据包。

我发现Ramda是一个设计优秀的库，它提供了很多工具来帮助我们在Javascript这种语言环境下简洁、优雅地进行函数式编程。

如果你在阅读本系列文章时想体验Ramda，[Ramda官网提供了一个方便的浏览器沙盒试玩](http://ramdajs.com/repl/)。

## **关于函数**

函数式编程(Functional Programming)，顾名思义，是一种与函数这个概念密切相关的编程方法。为简单起见，我们先这样定义函数：

>
函数是一个可重复使用的代码片段，它能被传入0个或多个输入参数调用，然后返回一个结果给调用者。

下面是一个简单的JavaScript函数：

```JavaScript
function double(x) {
    return x * 2
}
```

若使用ES6的箭头函数语法，你甚至可以用更简洁的形式来写同一个函数。我特意在这里提到箭头函数，因为为了简洁及更高的可读性，我们后面会大量地使用这种箭头函数的语法。

```JavaScript
// 简单的ES6箭头函数(这里只有一行代码)
const double = x => x * 2
```

几乎每种语言都支持类型这样的函数定义方式(更接近数学语言符号)。

有些语言会更彻底，直接在语法层面就支持函数为第一等公民(First-class)。这里的第一等公民(First-class)，我的意思是可以把与函数当成其它数据类型一样的使用。例如：

- 将函数作为常量或变量来引用
- 将函数作为参数传递给其它函数
- 从其它函数中把函数作为结果返回给调用者

Javascript就是具备这种特性的其中一种语言，这是能运用FP的编程范式写Javascript的关键原因。

## **纯函数**

当我们在使用函数式风格来写程序时，传说中的【纯】函数至关重要。

纯函数是指没有副作用的函数。纯函数被调用时，不会更改外部变量的值，不会消耗标准输入的数据，不会产生数据到标准输出，不会读或写数据库，也不会更改传入参数的值...

纯函数遵从一个基本原则：无论你调用它多少次，只要你传入的参数值不变，它都会返回同一个结果。

当然，你可以照常地使用不纯的函数（而且你必然要用到非纯函数，如果你的程序要产生有用的价值），但你程序中的大部分代码都应该尽量使用纯函数。

## **赋值原是永恒**

另一个FP中的重要概念是"值不变性(Immutability)"。顾名思义，指的是变量一旦被赋值，它的值就永不可改变。

当我在使用赋值不改的风格时，只要我初始化完一个对象的值后，我就永远都不会再修改它。这意味着不会有任何数组的元素或对象的属性被改变。

如果我需要修改数组或对象内里的数据时，我会直接复制一份更新相关值再把新的数组直接返回。后面的文章会详细地讨论这一点。

值不变性与纯函数是相辅相成的。因为纯函数不允许有副作用，所以也就不会修改外部数据结构。所以其实两者是严格地以赋值不改的风格处理数据的。

## **从何入手？**

要以函数式的思维方式去写程序，最容易上手的技能就是从现在开始，用集合迭代函数(Collection-iteration Function)去改写你代码中的所有循环结构。

如果你有写过其它有类似的集合迭代函数的语言(Ruby及Smalltalk就是两个例子)，你应该对这个写法很有熟悉感。

Martin Fowler有几篇关于“集合管道”很这个概念的写得很好的文章，展示了如何使[使用这些函数](http://martinfowler.com/articles/collection-pipeline/)及[使用集合管道来重构旧代码](http://martinfowler.com/articles/refactoring-pipelines.html)。

提醒一下，这些函数其实都是在Array.prototype下(仅reject例外)可直接使用，所以其实你并非一定要引入Ramda才使用这些函数。但是，在这个系列的文章中，我依然会使用Ramda的版本以保持上下文的一致性。

## **FOREACH**

使用`forEach`可以替代原本我们习以为常的显式循环结构：

```javascript
// 抛弃这个写法
for (const value of myArray) {
  console.log(value)
}
 
// 用这个函数代替
forEach(value => console.log(value), myArray)
```

`forEach`所需的参数是一个函数及一个数组，它会逐个使用数组中的元素作为参数调用你传入的函数。

虽然`forEach`是最容易使用的集合迭代函数，但事实上我们在运用FP风格写程序时很少用到它。因为它并不返回值，所以我们只是在需要调用有副作用的函数时才会用到它。


## **MAP**

下一个最重要的需要学习的函数是`map`。与`forEach`类似，`map`也是把你传入的函数参数作用在数组的每个元素上。但与`forEach`不同的是，map会把每个元素应用的函数的返回值收集到一个新数组中，然后作为返回值返回给map的调用者。

举个例：

```javascript
map(x => x * 2, [1, 2, 3])  // --> [2, 4, 6]
```

这里用到了一个匿名函数，我们可以很容易地把匿名函数改成一个有命名的函数：

```javascript
const double = x => x * 2
 
map(double, [1, 2, 3])
```

## **FILTER/REJECT**

接下来，我们看看`filter`及`reject`。顾名思义，`filter`就是从一个数组中，按你传入的过滤函数的意义，返回被函数过滤之后的新数组。例如：

```javascript
const isEven = x => x % 2 === 0
 
filter(isEven, [1, 2, 3, 4])  // --> [2, 4]
```

`filter`应用你传入它的过滤函数(这里是`isEven`)到每个数组元素上。只要过滤函数返回的值为真，对应的数组元素就会加入到要返回的结果集中。反之，若过滤函数返回值为假，则其调用的数组元素就不会出现在返回集合里。

`reject`其实是做的相同的事，只是含义反过来。它只保留过滤函数返回假的数组元素，过滤函数返回值为真时对应的数组元素会被舍弃。

```javascript
reject(isEven, [1, 2, 3, 4]) // --> [1, 3]

```

## **FIND**

`find`应用你传入它的函数到每个数组元素上，并且当应用的函数返回值为真时，返回首次为真的函数对应的数组元素。

```javascript
find(isEven, [1, 2, 3, 4]) // --> 2
```

## **REDUCE**

`reduce`会比上面介绍的几个函数都更复杂些。这个函数值得深入去理解，如果你一开始看不懂也不紧要，记住不要因为它而放弃你的FP之路。你完全可以在没有`reduce`的情况下也写出很FP风格的代码。

`reduce`接受三个参数：

- 一个函数（这个函数接受两个参数）, 这个函数的第一个参数我们称之为"收集器"，第二个参数是要操作的数组中的元素。这个函数需要返回一个新的"收集器"的值。
- 一个初始值
- 你要操作的数组

我们来看看具体的例子然后推演一下reduce的处理过程：

```javascript
const add = (accum, value) => accum + value
 
reduce(add, 5, [1, 2, 3, 4]) // --> 15
```

1. `reduce`首先调用传入的函数(`add`)，把初始值(5)及数组的第一个元素(1)以参数形式传给`add`。`add`返回了收集器的值(` 5 + 1 = 6 `)。

2. `reduce`再次调用`add`，这次传给`add`的参数是新收集到的值(`6`)及数组的第二个元素(`2`)。`add`这时返回了`8`。

3. `reduce`再次调用`add`，这次传给`add`的参数是新收集到的值(`8`)及数组的第三个元素(`3`)。`add`这时返回了`11`。

4. `reduce`最后一次调用`add`，这次传给`add`的参数是新收集到的值(`11`)及数组的第四个元素(`4`)。`add`这时返回了`15`。

5. `reduce`返回最终收集到的值(`15`)。


## **总结**

通过学习使用这些集合迭代函数，你会习惯把函数作为参数传给另一个函数的概念。或者其实你已经在使用其它语言时早就习惯于这样做了，只是以前你并未了解到这就是函数式编程的风格。


## **接下来**

这个系列的下一篇，[组合函数的魔法](/thinking_in_ramda/thinking-in-ramda-combining-functions.html)，会介绍我们可以如何更进一步及开始用崭新及有趣的方法把多个函数组合成一个。



