---
layout: post
category: "thinking_in_ramda"
title:  "[翻译]Ramda思维：组合函数的魔法"
tags: [nodejs,functional programming]
summary: "这是一个关于函数式编程的系列文章，使用nodejs的ramda库来讲解如何在nodejs这门目前最流行的语言实现函数式编程的理念。"
---

这是【Ramda思维】这个讨论函数式编程思想系列文章的第2篇。

**\([英文原文链接在此](http://randycoulman.com/blog/2016/05/31/thinking-in-ramda-combining-functions/ "英文原文")\)**

在[上一篇](/thinking_in_ramda/thinking-in-ramda-Wrap-Up.html)中，我介绍了Ramda及一些FP的基本概念，例如函数、纯函数、以及赋值永恒性。然后建议最好的快速入门法就是学会使用集合迭代函数，例如`forEach`, `map`,`select`, 等等。


## **简单的组合**

当你熟悉了把函数作为参数传参到其它函数时，你就会发现很多时候你会想把多个函数组合起来（或者可以叫函数串联）。

Ramda提供了好几个函数来实现简单的函数组合操作。看看其中的一部分：



### **COMPLEMENT(取补集)**

上一篇中，我们用`find`来找到列表中的第一个偶数：

```javascript
const isEven = x => x % 2 === 0
 
find(isEven, [1, 2, 3, 4]) // --> 2
```

如果现在我们想要的是找到第一个奇数呢？当然我们可以马上写一个`isOdd`的函数来实现。但其实，我们知道任何一个数字，如果不是偶数就肯定是奇数。这在数学上叫互补律(奇数+偶数=自然数)，我们可以重用之前的`isEven`函数。

Ramda提供了一个高阶函数：`complement`，它接受另一个函数参数然后返回一个新的函数，新函数返回与原函数相反的值。例如原函数返回`true`的话新函数就返回`false`，反之亦然。

```javascript
// 用complement函数的版本
const isEven = x => x % 2 === 0
 
find(complement(isEven), [1, 2, 3, 4]) // --> 1

```

可读性可以更高一点的是：可以为`complement`化后的函数命名，使之能被重复使用：

```javascript
// 命名的isOdd函数
const isEven = x => x % 2 === 0
const isOdd = complement(isEven)
 
find(isOdd, [1, 2, 3, 4]) // --> 1

```

需要指出的是，`complement`在函数运算上实现了数值运算中的`!`(取反)操作的概念。



### **BOTH/EITHER(取与/取或)**

假设我们要实现一个美国的选举投票系统。一个人要投票，我们需要先确定他是否有合法的选举投票权。根据美国法律，一个人只有年满18岁并且是我国公民才能选举投票。另外，只要他在美国境内出生或通过合法的移民手续后就被认为是美国的公民。

```javascript
// 合法的选举人
const wasBornInCountry = person => person.birthCountry === OUR_COUNTRY
const wasNaturalized = person => Boolean(person.naturalizationDate)
const isOver18 = person => person.age >= 18
 
const isCitizen = person => wasBornInCountry(person) || wasNaturalized(person)
 
const isEligibleToVote = person => isOver18(person) && isCitizen(person)

```

上面的代码是可行的，但Ramda提供了一些方便快捷的函数使我们可以把上述代码简化一下：

- `both`：实现逻辑与(且)，接受两个函数参数, 返回一个新的函数。新函数只有在两个函数参数都返回`true`时才返回`true`，否则返回`false`。

- `either`：实现逻辑或，接受两个函数参数，返回一个新的函数。只要两个函数参数任意一个返回`true`，新函数就会返回`true`，否则返回`false`。

借助这两个函数，我们可以简化一下`isCitizen`及`isEligibleToVote`：

```javascript

// 使用both及either简洁代码

const isCitizen = either(wasBornInCountry, wasNaturalized)
const isEligibleToVote = both(isOver18, isCitizen)

```

可以看到，`both`也同`complement`一樣，在函數上實現了數值運算的邏輯操作，這裡`both`實現的是`&&`（邏輯與）操作。同理，`either`也是，实现了`||`（逻辑或）操作。

Ramda同时也提供了`allPass`及`anyPass`函数，接受一个任意长度的函数数组。顾名思义地，`allPass`类似`both`，`anyPass`类似`either`。



## **PIPELINES(管道)**

**这是相当重要的一个概念，可以说是最有趣的魔法棒。**

很多时候，我们需要用一个管道（也可以说是工厂里的流水线）的方式，对数据进行若干个环节（函数）的处理。例如，我们可能拿到两个数字，先相乘，再加1，然后取平方。我们可以这样写：

```javascript

// 直观但粗糙的管道（流水线）

const multiply = (a, b) => a * b
const addOne = x => x + 1
const square = x => x * x
 
const operate = (x, y) => {
  const product = multiply(x, y)
  const incremented = addOne(product)
  const squared = square(incremented)
 
  return squared
}
 
operate(3, 4) // => ((3 * 4) + 1)^2 => (12 + 1)^2 => 13^2 => 169

```

请仔细观察上述代码中每个环节的操作是如果作用到上一操作的输出的。



### **PIPE(管道)**

Ramda提供了`pipe`函数，接受一个或多个函数作为参数，然后返回一个新的函数。

新的函数的参数个数与传入`pipe`的第一个函数的参数数量一样。然后输入数据（参数）就从第一个函数一直往下流，流经传入`pipe`的每个函数参数去处理。`pipe`接受的函数参数就好样管道的每个处理环节一样，依次传递处理结果给下一级函数处理。直到最后一个函数处理完，其返回的结果就是你调用`pipe`生成的函数所取得的结果。

请注意，传入`pipe`的函数中，除了第一个之外，其余所有函数都只能是单一参数函数（只接受一个参数）。

了解完`pipe`的特性后，我们就可以用`pipe`来简化上面粗糙的`operate`函数了：

```javascript
const operate = pipe(multiply, addOne, square)
```

处理过程：

1. 当我们调用`operate(3, 4)`时，`pipe`把`3`及`4`这两个参数传给`multiply`函数，`multiply`返回处理结果`12`
2. `multiply`的结果以参数形式传入`addOne`，`addOne`返回`13`
3. `addOne`返回的`13`流入下一个处理函数`square`，计算平方值，得到`169`
4. `square`返回的`169`就是整个`operate`的输出（结果）



### **COMPOSE(组合)**

另一个改写`operate`的方式是使用一行写完的风格来去除所有临时变量：

```javascript
const operate = (x, y) => square(addOne(multiply(x, y)))

```

这样代码会更为紧凑，但也变得没有那么容易读懂。这个写法可以用Ramda的`compose`函数来改写。

`compose`其实与`pipe`的处理方式一样，只是它会从右向左（与`pipe`相反）执行你传入的函数参数。看看用`compose`改写后的`operate`：

```javascript
const operate = compose(square, addOne, multiply)

```

这样得到的效果与上面介绍的`pipe`一样，只是传入的函数参数次序刚好相反。其实，Ramda底层中`compose`函数是用`pipe`实现的(你应该能猜到如何用`pipe`实现`compose`)。

我通常这样看`compose`：`compose(f, g)(value)`等同于`f(g(value))`。

与`pipe`同理，需要注意除了最后一个传入函数之外，其余函数都是单一参数参数。



### **用COMPOSE还是PIPE ?**

也许`pipe`应该是对于从命令式编程语言过来的人来说最容易理解的，因为命令式编程习惯了从左到右看函数。但是`compose`会更容易变形成为嵌套函数的形式。

我至今尚未归纳出一个很好的原则来界定何时用`compose`何时用`pipe`，因为两者在Ramda的思维里是一致的，也许选用哪个其实并不太重要，你看哪个更容易理解就用那个就行了。



## **总结**

通过以具体的特定逻辑组合多个函数，我们可以写出更为有趣的函数。



## **下一篇**

也许你已经注意到，我们在组合函数时会忽略被组合函数的参数。只有当最终调用组合后的函数时，我们才会传入所需的参数。

这在函数式编程中是很常见的事，我在下一篇：[部分应用函数](/thinking_in_ramda/thinking-in-ramda-partial-application.html)中会详细介绍这一点。

