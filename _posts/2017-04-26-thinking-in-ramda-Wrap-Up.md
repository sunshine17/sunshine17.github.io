---
layout: post
category: "thinking_in_ramda"
title:  "[翻译]Ramda程序思想：开篇"
tags: [nodejs,functional programming]
summary: "这是一个关于函数式编程的系列文章，使用nodejs的ramda库来讲解如何在nodejs这门目前最流行的语言实现函数式编程的理念。"
---

>
译者注：这是一个关于函数式编程的系列文章，主要讨论如何用nodejs这门目前最流行的语言通过使用函数式编程的理念来写出高可读性、健壮而简洁的程序代码。

**\([英文原文链接在此](http://randycoulman.com/blog/2016/05/24/thinking-in-ramda-getting-started/ "英文原文链接在此")\)**

本文是这个系列文章的开篇，此系列讨论的是nodejs语言下的函数式编程(以下简称FP,即Functional Programming)，名为【Ramda编程思想】。

在这个系列文章里，我会使用[Ramda](http://ramdajs.com/ "Ramda")这个js库来讨论FP，[Ramda](http://ramdajs.com/ "Ramda")这个库与其它流行的js库\(例如[Underscore](http://underscorejs.org/ "Underscore")及[Lodash](https://lodash.com/ "lodash")\)或其它在语言层面就有FP特性的语言中的FP概念其实是相通的。

我会尽量只讨论轻量级、不那么学术性质那一面的FP，因为我希望这个系列能让大部分开发者都能轻易阅读，当然也有部分原因是我自己在FP这个领域也并未有太深的造诣。

## **RAMDA**

其实之前我已经在这个博客里多次介绍过[Ramda](http://ramdajs.com/ "Ramda")这个Javascript库了：

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

(未完待续...)

