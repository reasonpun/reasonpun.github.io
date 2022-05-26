---
title: 如何用Provider管理Flutter中的状态
date: 2022-05-25 18:52:50
categories:
- Flutter
- Provider
tags:
- Flutter
- Provider
---

{% asset_img 1.jpeg 示意图 width="400" %}

可以认为Flutter是一个正在茁壮成长的孩子。

<!--more-->

本系列全部文章：

Part 1 ：[How-To-Manage-State-in-Flutter-With-Provider.html](https://pangz.fun/How-To-Manage-State-in-Flutter-With-Provider.html)
Part 2 ：[Should-You-Use-BLoC-to-Manage-State-in-Flutter.html](https://pangz.fun/Should-You-Use-BLoC-to-Manage-State-in-Flutter.html)
Part 3 ：[How-to-manage-state-in-Flutter-with-Mobx.html](https://pangz.fun/How-to-manage-state-in-Flutter-with-Mobx.html)

谷歌的UI工具包使你能够从一个代码库中构建原生和高性能的移动和网络应用。
最重要的是，大量可用的小工具使你获得了快速和愉快的开发体验。

在摆弄新玩具的时候，一般来说，阅读说明书是个好主意。由于Flutter是在几年前才开始流行起来的，它的说明书还在不断完善中。

清晰而简明的指南会让我的Flutter项目变得更好。尤其是为状态管理选择合适的工具，这绝不是一件容易的事。选项很多，但往往缺乏对它们的清晰比较。

在我看来，状态管理器之间的比较是非常需要的。一旦你的应用程序的状态与一个状态管理器纠缠在一起，切换到另一个工具并不是一件容易的事。因此，为了帮助你第一次就能做好，我将在一个由三部分组成的系列中比较Flutter最受欢迎的状态管理器。

第一部分：你是否需要一个状态管理器和用Provider管理状态

#### 你需要一个状态管理人吗？
在你开始比较无数的状态管理器选项之前，你应该问自己。

> "我真的需要一个状态管理器吗？"

如果你的应用程序在设计上是非常扁平的，而且状态很少在widget树的上下一层移动，那么通过属性和回调来传递状态可能会给你节省很多模板。

在下面的例子中，子程序从父程序中传递了一个回调，以跟踪TextButton被点击的次数。

```dart
class ParentState extends State<Parent> {
  int clicked = 0;

  void updateClickCount() {
    setState(() {
      clicked += 1;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      child: Column(children: [
        Child(onClickedCallBack: updateClickCount,),
        Text('I was clicked $clicked times!')
      ],)
      
    );
  }
}

class Child extends StatelessWidget {
  const Child({ key, this.onClickedCallBack }) : super(key: key);

  final void Function() onClickedCallBack;

  @override
  Widget build(BuildContext context) {
    return TextButton(onPressed: onClickedCallBack, child: Text('Click me!'))
  }
}
```

然而，如果你发现自己通过部件树的多个层次传递回调，这种方式管理状态很快就会变得麻烦和混乱。在这种情况下，你可以肯定你需要一种更强大的方式来管理状态。
但是，什么是适合这项工作的工具呢？

#### 用Provider管理状态

在Flutter应用程序中，有很多选择来管理你的状态。Provider是最受欢迎的状态管理器之一。这个社区创建的工具依赖于三个核心概念:

 * ChangeNotifier：您的状态的存储，状态从这里被更新，消耗状态的部件被通知。
 * ChangeNotifierProvider：这个部件使ChangeNotifier可以被树中的底层部件访问。
 * Consumer：一个听从状态变化并相应地更新用户界面的部件。

下面是一个使用Provider模式更新点击数的例子:

 * 状态是由DataProvider类存储和处理的，它存储了_count。
 * 通过在点击按钮时调用IncrementCount，_count从子部件中被更新。
 * NotifyListeners()确保通过消费者访问状态的父小组件得到相应的更新。

```dart
class DataProvider extends ChangeNotifier {
  int _count = 0;
  int get count => _count;
 
  DataProvider() {}
 
  void incrementCount(){
    _count++;
    notifyListeners();
  }
}

class Parent extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (BuildContext context) { DataProvider();},
      child: Consumer<DataProvider>(
        builder: (context, dataProvider, _) => Container(
          child: Column(children: [
            Child(),
            Text('I was clicked ${dataProvider._count}. times!')
          ],)
          
        ),
      ),
    );
  }
}

class Child extends StatelessWidget {
  const Child({ key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    DataProvider _data = Provider.of<DataProvider>(context);
    return TextButton(onPressed: () => _data.incrementCount(), child: Text('Click me!'))
  }
}
```

那么，你认为Provider是状态管理的最佳工具吗？

### 优点

 * 维护良好，经得起考验的软件包。
 * 单向状态管理的完整工具箱。

### 缺点
 * 总共有九个不同的提供者，对提供者的正确理解并不容易获得。
 * 不必要的重建也不容易避免。默认情况下，Consumer会更新其所有的子部件，即使状态中不相关的部分已经被更新。
 * Provider依赖于Flutter SDK，这使得你的业务逻辑和框架不可分割。这在架构设计中被认为是一种不好的做法。
 * 状态更新不是基于事件的。所以在上面的例子中，我们知道_count已经被更新了，但我们只能猜测变化的来源。这使得跟踪和理解你的应用程序中的状态变化变得复杂。

综上所述，我认为Provider对于有经验的Flutter开发者来说是一个不错的选择，他们正在构建一个有简单状态需求的应用程序。Provider很容易学习，但很难掌握。
此外，它缺乏更复杂的应用程序所需的基于事件的状态跟踪。

<!-- https://betterprogramming.pub/how-to-manage-state-in-flutter-with-provider-661ff322dd22 -->