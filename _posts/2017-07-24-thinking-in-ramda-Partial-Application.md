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

之前我們讨论的函数管道都是针对只有一个参数的函数而言，但如果需要管道化的函数是有多个参数呢？

例如，我们有一个集合，里面全是书本的对象，现在需要找出某一年出版的所有书的标题。下面看看如何用Ramda的集合迭代函数来实现这个需求：

```javascript
const publishedInYear = (book, year) => book.year === year
 
const titlesForYear = (books, year) => {
  const selected = filter(book => publishedInYear(book, year), books)
 
  return map(book => book.title, selected)
}
```

上例中，其实可以看出，如果能把`filter`及`map`两个函数组合成一个管道会更直观及优雅，但直到目前为止，我们还不知道该如何组合这类有不只一个参数的函数。

如果可以连`filter`里的箭头函数都省略不用，可读性还能进一步提高呢。现在我们先尝试实现这一点，因为省略箭头函数的过程或许可以给我们一些启发，从而实现多参函数的管道化。

## **高阶函数**

在[这个系列的开篇](/thinking_in_ramda/thinking-in-ramda-Wrap-Up.html)，我们讨论了函数作为第一类构造器。作为第一类对象的函数可以被以实参的形式传入其它函数使用，然后同理也能被其它函数当作结果返回给调用者。前者我们已经屡试不爽，但把函数作为结果返回这样的情况我们还没遇到过。

接受函数作为参数或返回函数作为结果的函数，就是所谓的"高阶函数"。

上例中，我们传入了一个箭头函数给`filter`: `book => publishedInYear(book, year)`, 然后现在我们想要把箭头也省略掉。为此，我们需要一个函数接受一个书本为入参，当这本书是在某个年份出版时，这个函数就返回true。同时，我们需要传入一个出版年份来使这个函数足够灵活。

实现这个的办法就是把`publishedInYear`改成能返回另一个函数的函数。我会用完整的函数语法来实现，这样大家能看得更清楚，然后再展示一个用箭头函数实现的简化版本。

```javascript
// Full function version:
function publishedInYear(year) {
  return function(book) {
    return book.year === year
  }
}
 
// Arrow function version:
const publishedInYear = year => book => book.year === year
```

有了这个新版本的`publishedInYear`函数，我们可以重写`filter`的函数调用，从而直接去掉原来的箭头函数。

```javascript
const publishedInYear = year => book => book.year === year
 
const titlesForYear = (books, year) => {
  const selected = filter(publishedInYear(year), books)
 
  return map(book => book.title, selected)
}
```

现在，当我们调用`filter`时，`publishedInYear(year)`会被立即求值，返回一个函数，该函数只需要一个参数：`book`，这个函数就是`filter`需要的函数形态。


## **部分应用函数**

我们可以用这种方式去重写任意的多参函数，使之变为只有一个参数的函数，但实际场景中并非所有要用到的函数都是我们自己控制的。另外，有时我们也需要用到普通的多参函数。

例如，假设我们有另一部分代码只是想检测一本书是否某个年份出版的，通常我们会这样写`publishedInYear(book, 2012)`，但部分应用函数的风格不会这样写，会写成`publishedInYear(2012)(book)`。这样只会降低可读性，同时也更啰嗦。

好在，Ramda提供了两个函数来避免这个问题：`partial`及`partialRight`。

这两个函数使我们可以用少于定义时的参数个数去调用函数，它们都会返回一个新函数，新函数的入参是剩下的参数，当剩下的所有参数都被传入时，新函数会调用原来你定义的函数。简单地说，就是会对你原本定义的函数wrap一层。

`partial`及`partialRight`的区别仅在于被调用时传入的参数与被包装的函数参数之间究竟是向左对齐还是向右对齐。

现在让我们回到原来的例子，使用这两个函数来实现相同的效果，从而无需重写`publishedInYear`。由于我们只想提供年份，而年份是最右侧的参数，所以这里使用`partialRight`。

```javascript
const publishedInYear = (book, year) => book.year === year
 
const titlesForYear = (books, year) => {
  const selected = filter(partialRight(publishedInYear, [year]), books)
 
  return map(book => book.title, selected)
}
```

如果一开始我们的`publishedInYear`函数的参数是`(year, book)`而非`(book, year)`，我们可以使用`partial`而非`partialRight`。

需要注意的是传递给`partial`或`partialRight`的参数都需要放在数组里，即使只有一个参数。否则有可能出错了半天你都未能反应过来。

```shell
# Confusing Error Message
First argument to _arity must be a non-negative integer no greater than ten
```

## **CURRY**




