---
title: 浅析setState()
date: 2022-04-01 09:00:24
categories:
- Flutter
tags:
- Flutter
- Function
---

{% asset_img 1.png 示意图 width="400" %}

大多数时候，初学者在编写setState()方法时都会感到困惑。

<!--more-->

> 大括号的顺序是什么，为什么有这么多的大括号，这些是什么？
> 这是一个方法吗？
> 这是一个函数中的一个函数吗？

此外，当人们这样写时：

{% asset_img 2.png 示意图 width="400" %}

那么这是不是一个函数的函数里面的函数调用？🤔

所以，让我们为你分解一下吧!
但在此之前，让我们先定义一些函数。

**你能说出这两个函数之间的区别吗？**

{% asset_img 3.jpg 示意图 width="400" %}

所以，第一个函数是Void Function()，基本上是一个零参数的函数，不返回任何东西。
第二个是String Function()，也是一个零参数的函数，但它返回String。

让我们在这里暂停一下，回到我们的setState()方法，打开它的文档。

{% asset_img 4.png 示意图 width="400" %}

{% asset_img 5.png 示意图 width="400" %}

所以setState是我们的State类中的一个类方法。这也是为什么你不能从任何无状态的Widget中得到setState的原因，因为他们没有一个相关的State类。
但也要注意，setState需要一个参数，这个参数是VoidCallback类型的。
现在，VoidCallback到底是什么？
如果你打开VoidCallback的文档，它不过是......

{% asset_img 6.png 示意图 width="400" %}

...只是void Function()的一个别名。
所以你可能会问，这是否意味着VoidCallback等同于我们之前写的displayMessage()函数，基本上等同于一个不需要参数、不返回数据的函数？

所以严格来说，setState需要一个void Function()作为参数。

{% asset_img 7.gif 示意图 width="400" %}

所以setState()基本上采取了一个匿名函数，一个没有名字的函数。
所以下一次，当你为setState()提供参数时，只要想象一下写一个没有函数名的无效函数体，你就不会对括号产生混淆了。

### 对函数的引用作为参数
有的时候，我也注意到人们这样做：
{% asset_img 8.png 示意图 width="400" %}

如果你也是其中之一，你是否意识到你只是在一个函数的函数里面做一个函数调用，没有任何额外的好处？
现在你理解了setState的方法，你是否认为提供一个对displayMessage()的引用就能简单地做到这一点？
观察一下下面的变化。

{% asset_img 9.gif 示意图 width="400" %}

在这里，我们只是为setState提供了一个对函数displayMessage()的引用，这就完成了任务，不需要嵌套函数，代码数量也少。


Dart称其为tear-offs，你也许可以在这里读到一些这方面的内容⤵️。
[https://dart.dev/guides/language/effective-dart/usage#dont-create-a-lambda-when-a-tear-off-will-do](https://dart.dev/guides/language/effective-dart/usage#dont-create-a-lambda-when-a-tear-off-will-do)

事实上，你也可以定义一个函数变量，就像你对任何原始类型如String、bool或int所做的那样。

{% asset_img 10.png 示意图 width="400" %}

💡 注意：然而，函数声明比函数变量更值得推荐。

但请记住，这种提供引用的方式只有在你想提供的函数也是void Function()的情况下才有效。
以下情况不会起作用 🔴

{% asset_img 11.png 示意图 width="400" %}

就这样了。
我希望你在读完这篇文章后对setState()不再感到困惑。🙏
如果你觉得自己有什么不明白的地方，请你尝试着去挖掘一下文档，当涉及到Flutter时，它是一个神奇的信息宝库。