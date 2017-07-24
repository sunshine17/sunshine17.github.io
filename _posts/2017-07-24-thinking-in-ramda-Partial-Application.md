---
layout: post
category: "thinking_in_ramda"
title:  "Ramda思维：部分应用函数"
tags: [nodejs,functional programming]
summary: "这是一个关于函数式编程的系列文章，使用nodejs的ramda库来讲解如何在nodejs这门目前最流行的语言实现函数式编程的理念。"
---

这是【Ramda思维】这个讨论函数式编程思想系列文章的第3篇。

**\([英文原文链接在此](http://randycoulman.com/blog/2016/06/07/thinking-in-ramda-partial-application/ "英文原文")\)**

在[上一篇(第2篇)](/thinking_in_ramda/thinking-in-ramda-combining-functions.html)中，我们讨论了组合函数的几种方式，最后着重谈到如何使用`pipe`及`compose`来实现以管道的方式顺序调用一组函数。
