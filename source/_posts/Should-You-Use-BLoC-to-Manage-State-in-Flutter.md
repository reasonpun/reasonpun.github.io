---
title: 你应该使用BLoC来管理Flutter中的状态吗？
date: 2022-05-25 18:49:32
categories:
- Flutter
tags:
- Flutter
- BLoC
---

{% asset_img 1.jpeg 示意图 width="400" %}

我将对Flutter中最广泛使用的状态管理器进行并列比较。

现在有很多工具，但往往缺乏对它们的明确比较。这是一个遗憾，因为选择了错误的状态管理器，当你开始构建时就不容易挽回了。

<!--more-->

本系列全部文章：

Part 1 ：[How-To-Manage-State-in-Flutter-With-Provider.html](https://pangz.fun/How-To-Manage-State-in-Flutter-With-Provider.html)
Part 2 ：[Should-You-Use-BLoC-to-Manage-State-in-Flutter.html](https://pangz.fun/Should-You-Use-BLoC-to-Manage-State-in-Flutter.html)
Part 3 ：[How-to-manage-state-in-Flutter-with-Mobx.html](https://pangz.fun/How-to-manage-state-in-Flutter-with-Mobx.html)

因此，我为你提供一个简短的指导手册。在第一次使用前请仔细阅读! 
第二个最流行的状态管理器，称为Flutter bloc，是本博客的主题。第一部分的链接见下文。

[如何用Provider管理Flutter中的状态]()

#### 什么是Flutter bloc？

Flutter bloc 包的名字来自于 BLoC 设计模式。BLoC 是 "业务逻辑组件 "的缩写，与大多数状态管理器一样，旨在将业务逻辑与视图解耦。

Flutter bloc包为您提供了在您的应用程序中实现BLoC模式的所有工具。该包的核心是围绕 "Cubits "和 "Blocs "这两个主要概念，我将简要地解释。

#### 用Cubit管理状态

{% asset_img 2.png 示意图 width="400" %}

用Cubit管理一个状态，对于以前使用过状态管理器的人来说，可能会觉得很熟悉。

 * 状态被存储和处理在一个Cubit类中。此外，Cubit暴露了一个状态变化的流。
 * BlocProvider使Cubit能够被视图访问。
 * BlocBuilder监听变化的状态并相应地渲染用户界面。

因此，管理一个递增的计数器的状态，将看起来如下:

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
}

void main() => runApp(CounterApp());

class CounterApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: BlocProvider(
        create: (_) => CounterCubit(),
        child: CounterPage(),
      ),
    );
  }
}

class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Counter')),
      body: BlocBuilder<CounterCubit, int>(
        builder: (context, count) => Center(child: Text('$count')),
      ),
      floatingActionButton: Column(
        crossAxisAlignment: CrossAxisAlignment.end,
        mainAxisAlignment: MainAxisAlignment.end,
        children: <Widget>[
          FloatingActionButton(
            child: const Icon(Icons.add),
            onPressed: () => context.read<CounterCubit>().increment(),
          ),
        ],
      ),
    );
  }
}
```

这种模式与我在本系列第一部分描述的用Provider进行的状态管理非常相似。
因此，Flutter Bloc的真正价值不在于Cubit，而在于实际Bloc本身。

#### 用Bloc管理状态

{% asset_img 3.png 示意图 width="400" %}

Bloc的独特卖点是事件。一个Cubit只是发送状态更新，而不提供关于变化的根本原因的额外信息。

只要你的状态是简单的，比如在计数器的例子中，这不会是一个问题。但假设你的计数器可以以10种不同的方式递增。你将如何调试意外的状态更新或根据变化的原因区分你的应用程序的行为？一言以蔽之，缺乏可追溯性带来了一个真正的问题。

```dart
// A Bloc transition
Transition {
  currentState: Counter(0),
  event: Increment,
  nextState: Counter(1)
}
Transition {
  currentState: Counter(0),
  nextState: Counter(1)
}
```

Bloc通过引入事件解决了这个问题。事件提供了关于状态改变的原因的信息。
事件让你完全控制你的应用程序的状态流。对于那些不只做计数器增量的应用程序来说，这是很理想的。

但每个伟大的礼物都是有代价的。Blocs比Cubits更啰嗦。额外的模板--在下面的例子中显示--需要用于状态和事件。但是，在我看来，这是一个值得付出的代价。
从长远来看，可追溯性是你的应用程序可管理性的先决条件。

```dart
part of 'Counter_bloc.dart';

// Counter state

abstract class CounterState extends Equatable {
  final int duration;

  const CounterState(this.duration);

  @override
  List<Object> get props => [duration];
}

class CounterInitial extends CounterState {
  const CounterInitial(int count) : super(count);

  @override
  String toString() => 'CounterInitial { count: $count }';
}

class CounterIncremented extends CounterState {
  const CounterIncremented() : super(0);
}

// Counter events

part of 'Counter_bloc.dart';

abstract class CounterEvent extends Equatable {
  const CounterEvent();

  @override
  List<Object> get props => [];
}

class CounterIncremented extends CounterEvent {
  const CounterIncremented({required this.increment});
  final int increment;
}
```

谁应该使用Flutter BLoC
那么，现在您已经掌握了Flutter bloc软件包，您是否应该真正使用它？

#### 优点
 * 作为第二大最受欢迎的状态管理器，该包得到了很好的维护，并且已经证明了它的价值。
 * 提供非事件基础的Cubits和基于事件的Bloc，该包可以适应您的应用需求。
 * 有很多方法可以防止用户界面的冗余更新。例如，Bloc的状态就使用了equatable。Equatable将旧的状态与新的状态进行比较，并在状态未变时防止更新。此外，BlocListener有一个listenWhen属性，它就像一个看门人，只允许访问理想的更新。综上所述，该包提供了关注你的应用程序性能的工具。
 * 该文档非常出色，阐述的内容远远超过了核心的内部结构。有大量的例子、方法的优点和缺点、最佳和坏的做法、以及关于包的核心概念的背景信息。

#### 缺点
 * Flutter Bloc包的巨大规模使其难以学习。当面对一个庞大的高质量工具箱时，可能很难选择合适的工具来完成工作。对于不熟悉流和基于事件的状态管理的开发者来说，学习曲线尤其陡峭。
 * 在所有的状态管理中，该包有一个可疑的荣誉，即需要最多的模板。特别是基于事件的组块是相当冗长的。要设置前面提到的基本的计数器组，你需要三个独立的文件：一个用于计数器的状态，一个用于计数器的事件，一个用于实际的计数器组。虽然有很好的扩展来自动创建模板，但你的代码库仍然会像一个吃饱了的小孩一样成长。

最后，在我看来，Flutter bloc是大多数Flutter应用程序的出路。它克服了Provider缺乏基于事件的状态管理的问题。此外，它的工具集很广泛，使它可以用于更简单和更复杂的应用。
这些优势使其陡峭的学习曲线值得一试。

<!-- https://betterprogramming.pub/should-you-use-bloc-to-manage-state-in-flutter-4f504ebc8711 -->