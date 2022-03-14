---
title: Flutter中的架构模式
date: 2022-03-14 09:09:08
categories:
- Architecture
tags:
- Flutter
- Patterns
---

{% asset_img 1.png 示意图 width="400" %}

Flutter 拥有一个不断增长的状态管理解决方案生态系统。 [Flutter文档](https://flutter.dev/docs/development/data-and-backend/state-mgmt/options)本身列出了 10 多个选项！弄清楚选择哪种解决方案是一项艰巨的任务。

在我们进入更多细节之前，让我们首先了解一个非常基本的概念。 
Flutter 是一个声明式 UI 框架。这意味着我们需要呈现在特定的状态下Flutter UI效果。

```
UI = *f(*state*)*
```

牢记这一点有助于编写更好的与框架配合使用的Flutter代码。

现在让我们看一下最广泛采用的两种状态管理模式。

 * 业务逻辑组件 (BLoC) 模式
 * 提供者模式

注意：本文假设您熟悉 Flutter 和 dart 编程语言。

## Business Logic Component (BLoC)

### 为什么要创建 BLoC？

谷歌在 2018 年谷歌 I/O 开发者大会上介绍了 BLoC 模式。这种模式最初是由 Google Ads 团队在他们的 Flutter 应用中使用的。它的创建是为了处理复杂的状态变化，而这在当时的解决方案中是很难做到的。

### 什么是 BLoC？

BLoC 是 UI = f(state) 概念的直接实现。它将这一概念与 dart 语言的基本部分 - Streams 结合在一起。

流是一系列异步事件。我们也可以将应用程序中的 UI 交互视为事件流。

BLoC 接收这些事件，对这些事件进行一些处理，并输出一个状态流。

flutter_bloc 就是这样一种状态管理包，它可以轻松实现 BLoC 模式。由于一切都建立在 dart 流之上，因此它对测试有很好的支持。

Flutter bloc 包还有一些配套包，可以帮助实现一些复杂的用例，例如恢复状态和添加 undo-redo 功能。

需要注意的一点是，BLoC 只是一个消耗事件并发出状态的组件。我们仍然需要使用一些东西在小部件树中提供这个 BLoC。通常，提供程序用于此目的。

### BLoC 是如何工作的？

让我们创建一个 BLoC 来搜索城市并显示搜索结果列表。

1. 首先，我们创建 BLoC 将消费的事件。

```
abstract class CitiesEvent {}

class SearchCities extends CitiesEvent {
  String searchTerm;

  SearchCities({
    required this.searchTerm,
  });
}

class SelectCity extends CitiesEvent {
  int cityId;
  
  SelectCity({
    required this.cityId,
  });
}
```

2. 创建 UI 将使用的状态

```
abstract class CitiesState {}

class CitiesLoading extends CitiesState {}

class CitiesLoaded extends CitiesState {
  List<City> cities;
  
  CitiesLoaded({
    required this.cities,
  });
}
```

3. 创建 BLoC。这里我们处理传入的事件并返回状态。

```
class CitiesBloc extends Bloc<CitiesEvent, CitiesState> {

	@override
	Stream<CitiesState> mapEventToState(
	  CitiesEvent event,
	) async* {
	  if (event is SearchCities) {
      
	    // Set state to Loading
	    yield CitiesLoading();
      
	    // Process the event
	    final citiesSearchResults = await fetchCities(event.searchTerm);
      
	    // Set the state to Loaded with the search results
	    yield CitiesLoaded(cities: citiesSearchResults);
	  }
	  // Handle other states
	}
}
```

4. 要访问小部件树中的块，我们提供了 BlocProvider

```
BlocProvider(
  create: (_) => CitiesBloc(),
  child: Column(
    children: const [
      CitiesSearchBar(),
      CitiesListPage(),
    ],
  ),
),
```

5. 我们检索上面提供的 bloc。从 UI 向 BLoC 添加新状态。 BLoC 将处理这些事件并返回状态。

```
@override
Widget build(BuildContext context) {
  final CitiesBloc citiesBloc = Provider.of<CitiesBloc>(context);
  return Padding(
    padding: const EdgeInsets.only(left: 10.0),
    child: TextFormField(
      onChanged: (searchTerm) =>
          citiesBloc.add(CitiesEvent.searchCities(searchTerm)),
    ),
  );
}
```

6. UI 将消费这些状态并更新视图

```
BlocBuilder<CitiesBloc, CitiesState>(
  builder: (context, state) {
    if (state is CitiesLoading) return CircularProgressIndicator()
    if (state is CitiesLoaded) return CitiesList(state.cities)
  },
),
```

首先，我们定义可以在应用程序中发生的事件。我们还定义了应用程序应该显示的状态。然后当用户与应用程序交互时，事件被添加到执行必要操作的 BLoC。 BLoC 然后发出由 UI 接收的状态。

简而言之，事件进入 BLoC，状态从 BLoC 出来。

## Provider

### 为什么要创建提供者模式？

BLoC 对许多人来说很难理解。在大多数用例中，总是使用数据流似乎不是一个优雅的解决方案。需要一个更简单的解决方案来处理Flutter中的状态。

第二个问题是使用 **InheritedWidget** 很难在小部件树下提供一些数据。

### 什么是提供者模式？

虽然，我们将这种方法称为提供者模式，但提供者包本身只是 **InheritedWidget** 的一个包装器，具有一些附加功能。它可以在小部件树下提供任何类型的对象。提供者本身不保持或改变状态。为此，我们可以使用 ChangeNotifier。 ChangeNotifier 是一个类，它为任何扩展它或将其用作 mixin 的类提供侦听功能。

### ChangeNotifier 的替代方案

将侦听器添加到更改通知器是 O(1) 操作。移除侦听器并向其侦听器发送更新是一个 O(n) 操作。

除了更改通知器，我们还可以使用状态通知器。 state_notifier 是由创建提供程序包的同一作者创建的包，因此二者的集成非常好。它比更改通知器更有效，并且使测试更容易。

### 提供者模式是如何工作的？

让我们实现相同的城市搜索用例

1.首先我们创建一个扩展 **ChangeNotifier** 的类。

```
class CitiesSearchModel extends ChangeNotifier {
	List<City> citiesList = [];

	void searchCities(String searchTerm) async {
		citiesList = await fetchCities(searchTerm);
		notifyListeners();
	}
}
```

2. 与 BLoC 类似，我们首先需要在小部件树中提供此类。一种称为 **ChangeNotifierProvider** 的特殊类型的提供程序用于此目的。

```
ChangeNotifierProvider<CitiesSearchModel>(
  builder: (context) => CitiesSearchModel(),
  child: CitiesListPage(),
);
```

3. 然后我们检索我们需要的模型类并调用 **searchCities** 函数。该模型将执行必要的数据获取，然后调用 **notifyListeners()**。

```
@override
Widget build(BuildContext context) {
  final CitiesBloc citiesBloc = Provider.of<CitiesSearchModel>(context);
  return Padding(
    padding: const EdgeInsets.only(left: 10.0),
    child: TextFormField(
      onChanged: (searchTerm) =>
          citiesBloc.searchCities(searchTerm),
    ),
  );
}
```

4. UI 将监听模型并更新视图。消费者小部件向模型添加了一个侦听器。每当调用 notifyListeners() 时，都会再次调用构建器并更新视图。

```
Consumer<CitiesSearchModel>(
  builder: (context, model, child) {
    return CitiesList(model.citiesList);
  },
)
```

## 什么时候用什么

任何好的状态管理解决方案都应该满足这些基本要求

 * 分离 UI 和业务逻辑
 * 让代码更容易测试
 * 使代码更具可读性

这两种方法都满足所有三个要求。但每个都有自己的缺点。

BLoC 模式具有陡峭的学习曲线。它为代码库添加了很多样板。

随着我们添加更多的侦听器，提供者模式变得更慢。理想的解决方案是两者的结合。我们需要为手头的工作选择正确的工具。

BLoC 非常强大，以流为骨干。但是，切记不可大材小用哟！

在需要 BLoC 提供的功能的情况下使用它。例如，一次有 2-3 个以上的侦听器处于活动状态，或者维护和浏览状态更改的历史记录，或者对传入事件进行去抖动和转换。

将提供者模式用于大多数（如果不是全部）状态管理需求。这是一个功能强大的解决方案，您可能永远不需要使用 BLoC。

## 近期发展

随着这两种方法在过去几年中变得更加成熟，对这些模式进行了一些更新，使其更易于使用。

BLoC 现在是 Cubit 类的子类。 Cubit 的工作方式与 BLoC 相同，但它试图通过用简单的函数替换事件类来减少我们需要用 BLoC 编写的一些样板。

提供程序包的创建者创建了一个名为riverpod 的新包。其网站上的标语为“提供者，但不同”。它的工作原理与提供程序相同，但试图修复原始包的一些缺陷。但原始的 provider 包仍然是 Flutter 官方文档中最受欢迎和推荐的解决方案。
