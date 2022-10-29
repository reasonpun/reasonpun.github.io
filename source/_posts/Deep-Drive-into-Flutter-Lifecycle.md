---
title: 深入了解 Flutter 生命周期
date: 2022-03-26 09:07:22
categories:
- Flutter
tags:
- Flutter
- Lifecycle
---

{% asset_img 1.png 示意图 width="400" %}

在这篇文章中，我们将讨论flutter的生命周期，每个人都知道生命周期在我们生活中是一个非常重要的存在，对于编码来说也是如此。

<!--more-->

在移动开发中，每一个移动应用都有一些事情，他们正在一个一个地做......
就像我们打开我们的应用程序，所以首先他们正在创建应用程序的设计，一步一步地根据情况来处理。
就像你看到的任何应用程序Instagram或Twitter，如果你打开这些，首先他们从服务器上加载数据，然后根据从服务器上加载的数据建立UI，然后根据你的使用情况和应用程序想要显示的数据，我想你明白我想要描述什么。

So!

让我们进入正题，在每一个flutter应用程序中都有一些生命周期的方法，这些方法根据情况调用。
在flutter中，所有的东西都是一个widget，主要是flutter应用从两个状态开始，StatelessWidget和StatefullWidget。

{% asset_img 2.png 示意图 width="400" %}

如果你的类扩展自**StatelessWidget**，意味着这个页面只是在里面建立了widget的设计，而不是更新这个。
如果你的类扩展自**StatefulWidget**，意味着这个页面将根据你的要求进行更新。
这是一个首次构建，然后通过你的操作来更新。
让我们现在开始讨论StatefulWidget的生命周期。

{% asset_img 3.jpeg 示意图 width="400" %}

 * createState()。当框架被指示建立一个有状态的Widget时，它会立即调用createState()。

它在树中给定的位置为这个部件创建可变的状态。
子类应该覆盖这个方法，以返回其相关的State子类的一个新创建的实例。

``` dart
_HomePageState createState() => _HomePageState();
```

框架可以在StatefulWidget的生命周期内多次调用这个方法。例如，如果该部件被插入到树的多个位置，框架将为每个位置创建一个单独的State对象。
同样的，如果widget从树上被移除，后来又被插入树中，框架将再次调用createState来创建一个新的State对象，从而简化State对象的生命周期。

``` dart
@override
  void initState() {
    super.initState();
  }
```

 * initState()。
 这是创建widget时调用的第一个方法（在类的构造函数之后）initState被调用一次，而且只有一次。它必须调用super.initState()。

当这个对象被插入到树中时，它就会被调用。
框架将为其创建的每个State对象精确地调用此方法一次。
覆盖此方法以执行初始化，这取决于此对象被插入树中的位置（即上下文）或用于配置此对象的部件（即部件）。

``` dart
@override
  void didChangeDependencies() {
    super.didChangeDependencies();
  }
```

 * didChangeDependencies()。这个方法在小部件第一次被建立时，在initState之后立即被调用。

当这个State对象的依赖关系发生变化时，它就会被调用。
例如，如果之前的构建调用引用了一个后来发生变化的InheritedWidget，框架会调用这个方法来通知这个对象的变化。
这个方法也会在initState之后立即被调用。从这个方法调用BuildContext.dependOnInheritedWidgetOfExactType是安全的。
子类很少覆盖这个方法，因为框架总是在依赖性变化后调用构建。一些子类确实覆盖了这个方法，因为他们需要在依赖关系发生变化时做一些昂贵的工作（例如，网络获取），而这些工作如果在每次构建时都做的话，就太昂贵了。

``` dart
@override
  Widget build(BuildContext context) {
    return Scaffold();
  }
```

 * build()。这个方法经常被调用。它是必须的，它必须返回一个Widget。

它描述的是这个Widget所代表的用户界面的部分。
框架在许多不同的情况下调用这个方法。比如说。
 * 在调用initState之后。
 * 在调用didUpdateWidget之后。
 * 在收到对setState的调用之后。
 * 在这个状态对象的依赖关系发生变化后（例如，由之前的构建所引用的InheritedWidget发生变化）。
 * 在调用deactivate之后，然后在另一个位置将状态对象重新插入树中。

这个方法有可能在每一帧中被调用，除了构建一个widget之外，应该不会有任何副作用。
框架用这个方法返回的widget替换这个widget下面的子树，要么更新现有的子树，要么移除子树并创建一个新的子树，这取决于这个方法返回的widget是否可以更新现有子树的根，这由调用Widget.canUpdate决定。
典型的实现是返回一个新创建的小部件，这些小部件是用这个小部件的构造函数、给定的BuildContext和这个State对象的内部状态的信息配置的。
给定的BuildContext包含关于该部件在树中的位置的信息，该部件正在被构建。例如，该上下文提供了树中这个位置的继承部件的集合。BuildContext参数总是与这个State对象的context属性相同，并且在这个对象的生命周期内保持不变。BuildContext参数在这里是多余的，以便这个方法与WidgetBuilder的签名相匹配。

``` dart
@override
void didUpdateWidget(ProductSquare oldWidget) {
    super.didUpdateWidget(oldWidget);
}
```

 * didUpdateWidget(): 如果父部件发生了变化，不得不重建这个部件（因为它需要给它不同的数据），但它被重建时的运行时类型是一样的，那么这个方法就会被调用。
 这是因为Flutter正在重新使用状态，这是长期的。在这种情况下，你可能想再次初始化一些数据，就像你在initState中一样。

每当widget的配置发生变化时，它就会被调用。
如果父 widget 重建并请求更新树中的这个位置以显示具有相同 runtimeType 和 Widget.key 的新 widget，框架将更新这个 State 对象的 widget 属性以引用新的 widget，然后用以前的 widget 作为参数调用此方法。
覆盖这个方法，以便在widget发生变化时做出反应（例如，启动隐式动画）。
框架总是在调用didUpdateWidget后调用build，这意味着在didUpdateWidget中对setState的任何调用都是多余的。
如果一个状态的构建方法依赖于一个本身可以改变状态的对象，例如ChangeNotifier或Stream，或者其他可以订阅接收通知的对象，那么一定要在initState、didUpdateWidget和dispose中正确订阅和取消订阅。
在initState中，订阅该对象。
在didUpdateWidget中，如果更新的widget配置需要更换对象，则取消订阅旧的对象并订阅新的对象。
在dispose中，取消对该对象的订阅。
这个方法的实现应该以对继承方法的调用开始，如super.didUpdateWidget(oldWidget)。

``` dart
@override
void deactivate() {
    super.deactivate();
}
```

 * deactivate()。当State被从树中移除时，Deactivate被调用，但在当前帧变化完成之前，它可能被重新插入。这个方法的存在基本上是因为State对象可以从树上的一个点移动到另一个点。

当这个对象从树上被移除时被调用。

每当框架将这个状态对象从树中移除时，就会调用这个方法。在某些情况下，框架会将State对象重新插入到树的另一部分（例如，如果由于使用了GlobalKey，包含这个State对象的子树从树的一个位置嫁接到另一个位置）。
如果发生这种情况，框架将调用激活，让状态对象有机会重新获得它在停用时释放的任何资源。然后它还会调用build，让State对象有机会适应它在树中的新位置。
如果框架确实重新插入了这个子树，它将在子树被从树中移除的动画帧结束前这样做。

由于这个原因，State对象可以推迟释放大部分资源，直到框架调用它们的处置方法。
子类应该覆盖这个方法来清理这个对象和树中其他元素之间的任何链接（例如，如果你为祖先提供了一个指向子孙的RenderObject的指针）。
这个方法的实现应该以对继承方法的调用结束，如super.deactivate()。

```dart
@override
void dispose() {
    super.dispose();
}
```

 * dispose(): dispose()在State对象被移除时被调用，这是永久性的。这个方法是你应该取消订阅和取消所有的动画、流等等。

当这个对象被永久地从树上删除时，它被调用。
当这个状态对象永远不会再建立时，框架会调用这个方法。在框架调用dispose后，State对象被认为是未被挂载的，挂载的属性为false。
在这个时候调用setState是一个错误。生命周期的这一阶段是终结的：没有办法重新安装一个已经被处置的状态对象。
子类应该覆盖这个方法来释放这个对象所保留的任何资源（例如，停止任何活动的动画）。

如果一个State的构建方法依赖于一个本身可以改变状态的对象，例如ChangeNotifier或Stream，或者其他可以订阅接收通知的对象，那么一定要在initState、didUpdateWidget和dispose中正确订阅和取消订阅。

在initState中，订阅该对象。
在didUpdateWidget中，如果更新的widget配置需要更换对象，则取消订阅旧的对象并订阅新的对象。

在dispose中，取消对该对象的订阅。
这个方法的实现应该以对继承方法的调用结束，如super.dispose()。

所以，我们讨论了关于有状态生命周期方法的所有要点。

但是，应用程序的生命周期呢?
意味着如果你打开任何页面而不做任何动作，或者当屏幕对用户可见时，哪个函数会被调用，对于安卓或IOS来说，让我们看看。

**AppLifecycleState（应用程序是可见的，并对用户的输入作出反应）。**

```dart
@override
void didChangeAppLifecycleState(AppLifecycleState state) {
  if (state == AppLifecycleState.inactive) {
  }else   if (state == AppLifecycleState.paused) {
  }else   if (state == AppLifecycleState.resumed) {
  }else   if (state == AppLifecycleState.detached) {
  }
}
```

{% asset_img 4.jpeg 示意图 width="400" %}

 * inactive - 应用程序处于非活动状态，不接受用户输入。它只适用于iOS。应用程序处于非活动状态，不接受用户输入。

在iOS上，该状态对应于应用程序或Flutter主视图在前台运行的处于前台非活动状态。
应用程序在以下情况下会过渡到这种状态电话，进入应用切换器或控制中心时响应TouchID请求，或
切换器或控制中心时，或当UIViewController托管的Flutter应用程序正在过渡。
在安卓系统中，这相当于一个应用程序或Flutter主机视图运行在处于前台非活动状态。应用程序在以下情况下会过渡到这种状态另一个活动被关注，如分屏应用、电话。

一个画中画应用程序，一个系统对话框，或其他窗口。处于这种状态的应用程序应该认为它们可能在任何时候被暂停。

 * paused - 应用程序当前对用户不可见，对用户输入没有反应，并且在后台运行。
当应用程序处于这种状态时，引擎将不调用PlatformDispatcher.onBeginFrame和PlatformDispatcher.onDrawFrame的回调。
 * resumed - 应用程序是可见的，并对用户的输入做出响应。
 * suspending - 应用程序将被暂时中止。 仅适用于Android detached
 * detached - 应用程序仍然托管在flutter引擎上，但与任何主机视图分离。当应用程序处于这种状态时，引擎是在没有视图的情况下运行的。它可以是在引擎首次初始化时正在附加一个视图，也可以是在视图因导航器弹出而被销毁后。