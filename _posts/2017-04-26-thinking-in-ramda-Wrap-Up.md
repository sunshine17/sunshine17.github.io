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

- 在

