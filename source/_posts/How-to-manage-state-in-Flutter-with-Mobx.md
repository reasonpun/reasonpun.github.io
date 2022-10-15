---
title: 如何用Mobx在Flutter中管理状态
date: 2022-05-25 20:15:02
categories:
- Flutter
tags:
- Flutter
- Mobx
---

{% asset_img 1.jpeg 示意图 width="400" %}

在这个由三部分组成的系列中，我将对Flutter中最广泛使用的状态管理器进行并列比较。
现在有很多工具，但往往缺乏对它们的明确比较。
这是一个遗憾，因为选择了错误的状态管理器，当你开始构建时就不容易恢复了。

<!--more-->

本系列全部文章：

Part 1 ：[How-To-Manage-State-in-Flutter-With-Provider.html](https://pangz.fun/How-To-Manage-State-in-Flutter-With-Provider.html)
Part 2 ：[Should-You-Use-BLoC-to-Manage-State-in-Flutter.html](https://pangz.fun/Should-You-Use-BLoC-to-Manage-State-in-Flutter.html)
Part 3 ：[How-to-manage-state-in-Flutter-with-Mobx.html](https://pangz.fun/How-to-manage-state-in-Flutter-with-Mobx.html)

因此，我为你提供一个简短的指导手册。
在第一次使用前请仔细阅读! 
在这个系列的前两部分--下面的链接--我讨论了状态管理的强大功能：Provider和Flutter Bloc。
Mobx在规模上要小得多，但拥有强大的工具，Mobx是其更大的兄弟的一个重要竞争者。
Mobx到底是什么？你应该在你的项目中实施它吗？

#### 什么是Mobx？

Mobx的基础是由三个核心概念形成的。

 * Observables：状态在观察物中被存储和变异。
 * Actions：行动导致可观察的状态改变。
 * Reactions：当其相关的可观察对象的状态发生变化时，反应会被触发。这个触发器是你的应用程序中想要的副作用的开始，例如一个UI变化。

整个流程被敏锐地显示在下面的三角形中，它是如此简单，你不会迷失在其中！

{% asset_img 2.png 示意图 width="400" %}

#### 臭名昭著的计数器
那么实际情况是怎样的呢？Mobx中臭名昭著的递增式计数器看起来如下:

```dart
import 'package:mobx/mobx.dart';

part 'counter.g.dart';

class Counter = CounterBase with _$Counter;

abstract class CounterBase with Store {
  @observable
  int value = 0;

  @action
  void increment() {
    value++;
  }
}
```

在上面的要点中，观察者和行动可以通过它们的装饰器清楚地识别出来--前缀为"@"。这些装饰器不仅仅是为了展示。它们作为指令被一个叫做 "Mobx_codegen "的包读取，随后在相关的文件 counter.g.dart 中为你生成完整的功能代码。因此，装饰器允许你的代码保持简单和可读，而繁重的工作在后台为你完成。

对状态变化的反应可以通过三种不同的方式进行，在下面的例子中显示:

 * 自动运行：这个反应里面的函数在初始化时自动运行。之后，在它所追踪的可观察对象的每一个状态变化之后，它都会运行。因此，在我们的例子中，计数器的初始值0将被打印出来，同时还将打印任何后续的增量。
 * 常规反应：这种类型的反应需要两个函数作为参数。第一个函数返回一个从要跟踪的观察物中得到的值。第二个函数是在第一个值的返回值发生变化时需要运行的效果。所以在这个例子中，只要counter.value发生变化，就会打印出counterValue。
 * 有条件的反应：这个反应，就像它的名字所暗示的那样，只在满足条件的情况下运行。条件在函数的第一个参数中提供，反应在第二个参数中。此外，该反应只运行一次，之后就会自动处理掉。因此，当我们的计数器值达到 "6 "时，将打印出一个一次性的信息。

```dart
// init the counter class
final _counter = Counter();

// Auto run
final dispose = autorun((_){
  print(_counter.value);
});

// Regular reaction
final dispose = reaction(
  (_) => counter.value,
  (counterValue) => print(counterValue)
);

// Conditional reaction
final dispose = when(
  (_) => counter.value > 5,
  () => print('Bigger than 5!')
);
```

在上述三种反应的基础上，Mobx还提供了一个观察者小组件。这个小组件跟踪一个观察者，并在其值发生变化时重建用户界面。

```dart
Observer(builder: (_) => Text('${_counter.value}', style: const TextStyle(fontSize: 20),)),
```

有了观察者小组件作为蛋糕上的糖衣，Mobx烤出了一个优秀的反应糕点。对状态变化的反应是通过简单的技术完成的，而不会牺牲灵活性。

#### 你应该使用Mobx吗？
不再多说，您是否应该在您的flutter项目中使用Mobx？

#### 优点
 * Mobx在简单性方面表现出色。该软件包易于学习，易于掌握。通过装饰器将模板最小化，对状态的反应在简洁的函数和部件中完成。
 * 该软件包被广泛使用。起源于Javascript的状态管理器，后来加入了Flutter，文档和最佳实践都已经到位了。

#### 缺点
 * Mobx缺乏基于事件的状态管理。这意味着你只知道状态发生了变化，但不知道引起变化的确切事件。在更复杂的应用程序中，这种可追溯性的缺乏可能会造成问题，因为它使调试和管理状态变化更加模糊。
 * Mobx生成器在增加简单性方面是很好的，但同时也增加了另一个抽象的层次。它们使人们更难真正看到和理解引擎盖下发生了什么。

总之，我认为Mobx是所有可以不使用基于事件的状态管理的应用程序的明智选择。
它的独特卖点是简单，Mobx在你的开发团队中的适应性将是非常顺利的。
出于这个原因，在我看来，Mobx胜过其直接竞争对手。
不过，对于基于事件的状态管理，Flutter Bloc仍然是最合适的方式。

<!-- https://medium.com/better-programming/how-to-manage-state-in-flutter-with-mobx-cf5be8e8e50e -->