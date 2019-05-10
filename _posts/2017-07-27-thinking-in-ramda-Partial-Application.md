---
layout: post
category: "thinking_in_ramda"
title:  "[TIR-03]Ramda思维：部分应用函数"
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

到处使用`partial`或`partialRight`会令出现大量重复又无聊的代码。但同样把多参函数拆成一个系列的单参函数再调用也极为笨拙。

幸好，Ramda提供了一个解决方案：`Curry`。

[Currying](https://en.wikipedia.org/wiki/Currying)是函数式编程领域里的一个核心概念。技术上讲，一个Curry化后的函数永远都是一组单参函数，这就是刚才我说的笨拙的地方。在纯粹的函数式语言中，语法上看Curry化函数与调用多参函数其实并无区别。

但Ramda是一个JavaScript的库，而JavaScript这种语言并没有很优雅的语法去描述如何调用一组单参函数，Ramda的作者们也没有按传统的currying定义去实现。

在Ramda里，一个Curry化的函数在被调用时，传参只能传它所需参数的一个子集，它会返回一个能接受剩余未传参数的函数。当你以全部参数调用Curry化的函数时，它会直接调用底层的函数。

你可以认为Curry化的函数集两个优点于一身：你能把它当普通函数一样用全部参数直接调用，结果就与普通函数完全一样，返回函数的值；又或者你调用它时只传一部分参数，这样它返回的就是一个部分应用的函数，就如上面的`partial`。

需要注意的是这种灵活性会牺牲一点性能，因为`curry`需要计算出底层函数是如何被调用的，然后决定如何实现底层函数的调用。我通常的做法是只curry化那些需要多个地方使用`partial`的函数。

现在我们回头看看使用`curry`后之前的`publishedInYear`会变成怎样。注意使用`curry`的效果就如同你调用`partial`，但没有`partialRight`的`curry`版本。以后我会更深入地探讨这个，现在，我们需要反转`publishedInYear`的参数顺序来让年份成为第一个参数。

```javascript
const publishedInYear = curry((year, book) => book.year === year)
 
const titlesForYear = (books, year) => {
  const selected = filter(publishedInYear(year), books)
 
  return map(book => book.title, selected)
}
```

这次我们可以只用年份参数去调用`publishedInYear`，从而得到一个单参函数，这个新函数只有一个参数book，当你用book去调用它时，它会执行原来的底层函数并返回结果。当时，我们依然可以直接按原来的方式`publishedInYear(2012, book)`调用它，而无需用之前那个`)(`的难看语法，这样就兼容了两种情况。

## **参数顺序**

注意，为了让`curry`符合我们的调用方式，我们需要反转参数顺序。这在函数式编程里是很常见的做法，所以，几乎所有的Ramda函数都是把要操作的数据放在最后一个参数。

你可以把函数签名中靠前面的参数看成是这个函数操作的配置数据。例如`publishedInYear`函数，`year`参数就是这个操作的"配置"（那`year`参数是什么呢？），然后`book`参数就是要操作的数据(问题是这个数据去哪了？)。

我们已见识过使用集合迭代函数来实现curry化的例子，集合迭代函数都把要操作的集合放在最后一个参数里，因为这样可以使实现函数式编程风格变得更容易。

## **错误的参数顺序**

如果我们没有反转`publishedInYear`函数的参数顺序，那会怎样呢？那如何继续利用curry化的优势？

Ramda提供了一些选择。

### **FLIP**

第一个选择是`flip`函数。`flip`接受一个有2个参数以上的函数，然后返回一个新函数，新函数接受与原函数同样多的参数，只是参数的顺序相反了。

使用`flip`，我們可以像下面这样反转`publishedInYear`的参数顺序：

```Javascript
const publishedInYear = curry((book, year) => book.year === year)
 
const titlesForYear = (books, year) => {
  const selected = filter(flip(publishedInYear)(year), books)
 
  return map(book => book.title, selected)
}
```

大多数情况下，我更偏向于在定义函数时就把参数顺序按我的需要来设计，但是当你需要使用一个不是自己写的函数时，`flip`就很能派上用场了。


### **PLACEHOLDER(占位符)**

更通常用于的方法就是"占位符"参数了：`__`。

假设有一个3个参数的函数，我们想要只提供第一个及第三个参数，中间那个参数留着以后再提供（当作被操作的数据类似的角色）。这时就可以使用占位符来充当中间的参数了：

```Javascript

// 中间参数最后才提供

const threeArgs = curry((a, b, c) => { /* ... */ })
 
const middleArgumentLater = threeArgs('value for a', __, 'value for c')

```

你也可以在一次函数调用中使用多个占位符。例如上述代码中，假设你只想提供中间那个参数：

```Javascript
// 只提供中间一个参数

const threeArgs = curry((a, b, c) => { /* ... */ })
 
const middleArgumentOnly = threeArgs(__, 'value for b', __)

```

这样，我们也可以用占位符去实现`flip`函数的效果：

```Javascript

// 使用占位符实现flip的效果

const publishedInYear = curry((book, year) => book.year === year)
 
const titlesForYear = (books, year) => {
  const selected = filter(publishedInYear(__, year), books)
 
  return map(book => book.title, selected)
}
```

这个版本可读性更高，但是如果我需要多次使用`flipped`的版本时，我会用`flip`定义一个帮助函数，然后直接使用这个帮助函数，而不是每次要用的时候才调用`flip`。在这个系列文章的后面我们会看到更多这样的例子。

需要注意的是`__`只适用于Curry化的函数，而`partial`, `partialRight`及`flip`则适用于所有函数。如果你需要把`__`用于普通函数，你可以先用`curry`函数包装一下你的函数。

## **组装成管道**

现在我们试试能否把`filter`及`map`的调用移到管道方式里。下面是目前的代码，`publishedInYear`的参数顺序按最直观的逻辑设计：

```Javascript
// 目前的代码

const publishedInYear = curry((year, book) => book.year === year)
 
const titlesForYear = (books, year) => {
  const selected = filter(publishedInYear(year), books)
 
  return map(book => book.title, selected)
}
```

我们已经在上一篇了解到`pipe`及`compose`的用法，但目前仍需要多一点信息才能把它们利用起来。

缺失的一点信息就是：所有的Ramda函数都是默认curry化的。这当然包括`filter`及`map`也是curry化的。所以`filter(publishedInYear(year))`这样写完全正确，而且会返回一个新的函数，新函数只有一个参数，就是要操作的数据，正如`map(book => book.title)`。

现在，组合出的管道如下：

```Javascript
const publishedInYear = curry((year, book) => book.year === year)
 
const titlesForYear = (books, year) =>
  pipe(
    filter(publishedInYear(year)),
    map(book => book.title)
  )(books)
```

我们干脆更进一步，把`titlesForYear`的参数顺序也反转过来，使其更符合Ramda"数据最后"的参数习惯。同时把它curry化，以便日后能用在其它管道里。

```
const publishedInYear = curry((year, book) => book.year === year)
 
const titlesForYear = curry((year, books) =>
  pipe(
    filter(publishedInYear(year)),
    map(book => book.title)
  )(books)
)
```

## **总结**

到目前为止，这一篇应该是这个系列里技术难度最大的一篇文章了。部分应用函数与Curry化也许需要花费一点时间与精力来掌握，但当你掌握了这些思维方式后，你会发觉这些思维方式能让你用非常强大的方法（数学方法）来处理数据变形的问题。

在这种思维方式下，你会慢慢习惯用一些小而美的函数来构建处理管道，从而巧妙地实现数据变形。

## **下一篇**

想要更函数式地写程序，我们需要抛弃早已熟习的"命令式"写法，转投到"定义式"的阵营。为此，我们需要一些能表达出以往"命令式"写法的内容的函数式方法。下一篇[用定义来写程序](/thinking_in_ramda/thinking-in-ramda-declarative-programming.html)会深入地讨论这个Topic。






