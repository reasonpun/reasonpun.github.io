---
title: Riverpod + Flutter Hooks：状态管理的最佳组合
date: 2022-06-02 09:18:46
categories:
- Flutter
tags:
- Flutter
- Riverpod
- Hooks
---

{% asset_img 1.png 示意图 width="400" %}

我编写flutter的代码已经有些年头了。
当我发现riverpod与flutter钩子的简单性相结合时的强大，我爱上了它，爱了，爱了！
现在几乎所有的项目中，我都使用它来进行状态管理。
我将向你解释为什么，并告诉你为什么你可以用这两个天使轻松地赋予你的项目。

<!--more-->

1. Flutter 钩子的简单性
每个Flutter开发者都是从使用StatefulWidget开始的，这是一个基础，我们必须通过它。
但是想象一下，你必须创建一个大的项目，重复使用StatefulWidget会变得很痛苦。
这就是flutter钩子出现的地方，flutter钩子帮助flutter开发者通过创建钩子来重用代码，然后，我们就可以快速推进项目，这里有一些例子。

 * 减少代码量

{% asset_img 2.png 示意图 width="400" %}

这个类，我们都是为StatefulWidget创建的，但大多数时候，我们从来没有使用过它，有了flutter钩子，一个widget的所有状态都在一个单一的类中处理，性能和StatefulWidget一样。

{% asset_img 3.png 示意图 width="400" %}

在一个StatefulWidget中，对于许多控制器，如AnimationController或TextEditingController，我们需要处理所有的生命周期，初始化，处置，didUpdateWidget，甚至添加TickerProviderStateMixin到状态类中，在每个StatefulWidget中我们都要使用这些控制器。

在flutter钩子中，我们只需要创建一个钩子来处理控制器的生命周期，然后我们在每个HookWidget中重复使用同一个钩子。
然后，代码就可以从这个样子：
{% asset_img 4.png 示意图 width="400" %}
变成这个样子：
{% asset_img 5.png 示意图 width="400" %}

所以，现在，我们知道用flutter钩子编码是多么容易，以及它对特别大的项目是多么有用。我不会在一个我将创建少于十个部件的应用程序中使用它，但在一个我需要创建更多部件的应用程序中，我不会错过使用flutter挂钩。

2. 钩子：Riverpod

状态管理的简单重要性在于：在所有的应用程序或多个widget之间共享一个或多个变量，每一个变量的变化都必须触发监听该变量的widget的重建。

如果你试图创建一个全局变量并改变它，这将不会触发小部件的重建。
为了解决这个问题，riverpod提供了创建一个全局提供者的可能性，它主要携带一个变量，但提供者可以携带一个stream，一个future，一个ChangeNotifier，...。

提供者可以很容易地在应用程序的每个部件中被访问，我们只需要把MyApp()部件包装成ProviderScrope。

{% asset_img 6.png 示意图 width="400" %}


在hooks_riverpod的新版本中，有一个叫做HookConsumerWidget的新部件，我们必须在我们的项目部件中扩展它。
HookConsumerWidget在构建时带有其他参数，WidgetRef（注意ref是简称），ref让我们有机会读取和观察提供者。

 * 读取提供者：一次性读取值，在值改变时不会触发重建。
 * 观察提供者：每一个变化都会触发小部件或小部件的一部分的重建，该小部件正在收听提供者。

 {% asset_img 7.png 示意图 width="400" %}

 在整个应用程序中观看相同的Firebase流

  {% asset_img 8.png 示意图 width="400" %}

纵观整个应用程序，我们可以跟踪并且修改provider。

  {% asset_img 9.png 示意图 width="400" %}

以上就是设置flutter_hooks和hooks_riverpod是整个过程。
也许有复杂的概念和许多方法来做同样的事情，但在你的应用程序中的大部分状态管理，你可能只使用这些概念。

关于riverpod的更多细节可以在这里找到[https://riverpod.dev/docs/getting_started](https://riverpod.dev/docs/getting_started)。

我希望我可以说服你使用riverpod和flutter_hooks，它是如此简单，如此容易使用和实现!

<!-- https://omasuaku.medium.com/riverpod-flutter-hooks-the-best-duo-for-state-management-9429728d632b -->