---
title: 适合初学者的Flutter Bloc模式
date: 2022-03-15 15:06:52
categories:
- Architecture
tags:
- Flutter
- Patterns
- Beginners
---

{% asset_img 1.png 示意图 width="400" %}

### 什么是Flutter 的Bocl模式

Flutter bloc 是 Fl​​utter 应用程序的状态管理之一。您可以使用它以 **简单** 的方式处理应用程序的所有可能状态。
Flutter bloc 易于使用，因为您和您的团队将很快理解这个概念，无论您的级别是什么，这个库都有非常好的文档和大量的示例，也是 Flutter 社区中使用最多的库之一，因此，如果您有任何疑问或问题，您可能会通过互联网上的简单搜索找到解决方案。

**功能强大**是因为它可以帮助您创建各种应用程序，例如您可以创建用于学习目的的应用程序，也可以在生产环境中创建复杂的应用程序，flutter bloc 在这两种情况下都有效。
这个库的另一个重要方面是您可以轻松地**测试**您的块逻辑。

更多信息，您可以访问官方网站：[https://bloclibrary.dev/#/gettingstarted](https://bloclibrary.dev/#/gettingstarted)

## 那么……我该如何从 Flutter bloc 开始呢？

我给你的第一个建议是你应该阅读文档并阅读基础知识，在本文中我将尝试解释它，但如果你需要更深入，我强烈建议你访问文档。

### 它是如何工作的？

当您使用Flutter bloc 时，您将创建事件来触发与应用程序的交互，然后负责的 bloc 将发出带有状态的请求数据，在一个真实的示例中，它将是这样的：

1- 用户单击按钮以获取游戏列表。
2- 事件被触发，并通知阻止用户想要一个游戏列表。
3- 该bloc将请求此数据（例如，从负责连接到 API 以获取数据的存储库）。
4-当块有数据时，它将确定数据是成功还是错误，然后它会发出一个状态。
5- 视图将听取欧盟可能发出的所有可能状态以对其作出反应。例如，如果 bloc 发出 Success 作为状态，则视图将使用游戏列表重建它，但如果状态为 Error，则视图将使用错误消息或您想要显示的任何内容进行重建。

{% asset_img 2.png 示意图 width="400" %}

完美，现在您知道了Flutter bloc 如何工作的主要概念！现在，是时候知道如何使用它了。
想象一下，你想创建一个与游戏相关的块逻辑，你将需要这三个类：

 * games_bloc.dart
 * games_state.dart
 * games_event.dart

如您所见，您将需要一个块、状态和事件类。
在每个类中，您将管理所需的信息，不要着急，接下里的内容里我们会详细讲解一下这些知识。

### Bloc Widgets

这些是库为您提供的用于管理所有可能情况的小部件，例如，添加事件、侦听状态、发出状态、根据状态重建视图等。

*BlocProvider/MultiBlocProvider*

BlocProvider 负责为其子代提供bloc。是在使用之前“初始化” bloc 的方式。

{% asset_img 3.png 示意图 width="400" %}

如果您需要提供多个块，您可以使用 MultiBlocProvider 来获取不同的提供者。

{% asset_img 4.png 示意图 width="400" %}

### RepositoryProvider/MultiRepositoryProvider

RepositoryProvider 用于为其子项提供存储库。通常，当您需要创建存储库类的实例时，您将使用它，然后使用 BlocProvider，您将在 

```
context.read<YourRepository>(); 
```

的帮助下访问该存储库；

这是一个例子。

{% asset_img 5.png 示意图 width="400" %}

如果您需要多个存储库提供程序，您可以使用 MultiRepositoryProvider

{% asset_img 6.png 示意图 width="400" %}

### BlocListener

使用这个小部件，您将能够“监听”从您的bloc发出的不同状态，然后对它们做出反应，例如，显示一个快餐栏、对话框或导航到另一个页面......这个小部件不重建视图，它只是在听。

{% asset_img 7.png 示意图 width="400" %}

### BlocBuilder

有了这个，你将能够根据它们的状态重建你的小部件。

{% asset_img 8.png 示意图 width="400" %}

### BlocConsumer

当您需要控制块的状态以重建小部件以及导航或显示对话框等时，此小部件非常有用。此小部件具有侦听器和构建器功能，因此您可以一起使用它。

{% asset_img 9.png 示意图 width="400" %}

### BlocSelector

此小部件允许开发人员通过根据当前区块状态选择新值来过滤更新。

{% asset_img 10.png 示意图 width="400" %}

有关更多信息，您可以查看文档。
之后，我们可以从例子开始🙌

## 在实际项目中使用 Flutter Bloc

在这个项目中，我们将使用来自游戏 API 的数据来获取有关游戏的信息并在我们的视图中显示它们。
我选择的 API 是 [RAWG](https://rawg.io/apidocs)。要使用它，您需要创建一个 API 密钥。

做好的程序大概张这个样子

{% asset_img 11.gif 示意图 width="400" %}

主页有不同的部分，我们来看看。

### 标题

这是一个简单的小部件，显示两个文本和一个圆形头像。

{% asset_img 12.png 示意图 width="400" %}

### 类别小部件

显示 API 返回调用 getGenres 的不同类型。这个小部件有四种可能的状态：

 * 成功：显示类别（类型）列表。
 * 错误：显示错误信息。
 * 正在加载：显示一个 CircularProgressIndicator。
 * 已选：更改所选类别的大小和颜色。

{% asset_img 13.png 示意图 width="400" %}

### 分类筛选游戏小部件

显示 API 在使用额外的类型参数调用 getGames 时返回的按类型过滤的不同游戏。它具有三种可能的状态：

 * 成功：按类别显示游戏列表。
 * 错误：显示错误信息。
 * 正在加载：显示一个 CircularProgressIndicator。

 ### 全部游戏

显示没有过滤器的游戏列表。此小部件仅在其 bloc 发出 Success 状态并具有三种可能状态时才会显示：

 * 成功：显示游戏列表。
 * 错误：显示错误信息。
 * 正在加载：显示一个 CircularProgressIndicator。

{% asset_img 15.png 示意图 width="400" %}

### 项目结构

对于这个例子，我创建了这个结构：

{% asset_img 16.png 示意图 width="400" %}

正如您在 home 小部件文件夹中看到的那样，我添加了我之前提到的所有小部件。每个小部件都有自己的bloc，因此更干净和可维护。

### Home页面

这个页面非常重要，因为我在这里使用了两个 Bloc Widget：MultiBlocProvider 和 RepositoryProvider。
这个页面的想法是在初始化这个页面时让所有的块都准备好使用，所以，要做到这一点，我需要用 RepositoryProvider 包装我的子类，为所有块提供存储库，而且，我需要初始化所有带有 MultiBlocProvider 的块。

```
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:infogames/repository/game_repository.dart';
import 'package:infogames/repository/service/game_service.dart';
import 'package:infogames/ui/home/pages/home_layout.dart';
import 'package:infogames/ui/home/widgets/category/category_barrel.dart';
import 'package:infogames/ui/home/widgets/all_games_list_widget/all_games_barrel.dart';
import 'package:infogames/ui/home/widgets/games_by_category/games_by_category.dart';

class HomePage extends StatelessWidget {
  const HomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.deepOrangeAccent,
      body: RepositoryProvider(
        create: (context) => GameRepository(service: GameService()),
        child: MultiBlocProvider(
          providers: [
            BlocProvider<AllGamesBloc>(
              create: (context) => AllGamesBloc(
                gameRepository: context.read<GameRepository>(),
              )..add(
                  GetGames(),
                ),
            ),
            BlocProvider<CategoryBloc>(
              create: (context) => CategoryBloc(
                gameRepository: context.read<GameRepository>(),
              )..add(
                  GetCategories(),
                ),
            ),
            BlocProvider<GamesByCategoryBloc>(
              create: (context) => GamesByCategoryBloc(
                gameRepository: context.read<GameRepository>(),
              ),
            ),
          ],
          child: HomeLayout(),
        ),
      ),
    );
  }
}
```

事实上，正如您在这段代码中看到的那样，有两个bloc在开头添加了两个事件：
 * 获取游戏
 * 获取类别
这是添加新事件以通知其我们需要一些数据的方法之一。所以此时，主页有三个bloc，触发了两个事件。现在来看看 HomeLayout。

### HomeLayout

此类具有我上面提到的三个主要小部件，还包含视图的骨架。

```
import 'package:flutter/material.dart';
import 'package:infogames/ui/home/widgets/all_games_widget/all_games_widget.dart';
import 'package:infogames/ui/home/widgets/category_widget/categories_widget.dart';
import 'package:infogames/ui/home/widgets/games_by_category_widget/games_by_category_widget.dart';
import 'package:infogames/ui/home/widgets/header_title/header_title.dart';
import 'package:infogames/ui/widgets/container_body.dart';

class HomeLayout extends StatelessWidget {
  const HomeLayout({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(top: 80.0),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          HeaderTitle(),
          const SizedBox(height: 40.0),
          ContainerBody(
            children: [
              CategoriesWidget(),
              GamesByCategoryWidget(),
              AllGamesWidget(title: 'All games'),
            ],
          )
        ],
      ),
    );
  }
}
```

下一步是查看这些小部件中的每一个，所以让我们开始使用 CategoriesWidget。

## CategoriesWidget

### Category event

这是我添加此小部件所需的所有事件的地方。
 * GetCategories：获取类别的事件。
 * SelectCategories：知道何时选择类别的事件。

 ```
 part of 'category_bloc.dart';

class CategoryEvent extends Equatable {
  @override
  List<Object?> get props => [];
}

class GetCategories extends CategoryEvent {}

class SelectCategory extends CategoryEvent {
  SelectCategory({
    required this.idSelected,
  });
  final int idSelected;

  @override
  List<Object?> get props => [idSelected];
}
```

### Category state

在这个类里边，bloc可以发出不同的状态。我创建了一个扩展来以简洁明了的方式处理视图中所有可能的状态。
我使用 Equatable 库和 Dart 中的对象作比较，如果你不是很清楚的话，强烈建议您查看[文档](https://pub.dev/packages/equatable)。

```
part of 'category_bloc.dart';

enum CategoryStatus { initial, success, error, loading, selected }

extension CategoryStatusX on CategoryStatus {
  bool get isInitial => this == CategoryStatus.initial;
  bool get isSuccess => this == CategoryStatus.success;
  bool get isError => this == CategoryStatus.error;
  bool get isLoading => this == CategoryStatus.loading;
  bool get isSelected => this == CategoryStatus.selected;
}

class CategoryState extends Equatable {
  const CategoryState({
    this.status = CategoryStatus.initial,
    List<Genre>? categories,
    int idSelected = 0,
  })  : categories = categories ?? const [],
        idSelected = idSelected;

  final List<Genre> categories;
  final CategoryStatus status;
  final int idSelected;

  @override
  List<Object?> get props => [status, categories, idSelected];

  CategoryState copyWith({
    List<Genre>? categories,
    CategoryStatus? status,
    int? idSelected,
  }) {
    return CategoryState(
      categories: categories ?? this.categories,
      status: status ?? this.status,
      idSelected: idSelected ?? this.idSelected,
    );
  }
}
```

### Category bloc

在这里，您需要处理您拥有的所有事件。如您所见，在第 13 行和第 14 行，我正在检查事件是否是一个或另一个以创建它的方法。
mapGetCategoriesEventToState：此方法调用存储库以从 API 获取数据。当存储库返回数据 o 抛出错误时，该块会发出相应的状态。
mapSelectCategoryEventToState：此方法将发出“选定”这样的状态。

```
import 'package:equatable/equatable.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:infogames/repository/game_repository.dart';
import 'package:infogames/repository/models/model_barrel.dart';

part 'category_event.dart';
part 'category_state.dart';

class CategoryBloc extends Bloc<CategoryEvent, CategoryState> {
  CategoryBloc({
    required this.gameRepository,
  }) : super(const CategoryState()) {
    on<GetCategories>(_mapGetCategoriesEventToState);
    on<SelectCategory>(_mapSelectCategoryEventToState);
  }
  final GameRepository gameRepository;

  void _mapGetCategoriesEventToState(
      GetCategories event, Emitter<CategoryState> emit) async {
    emit(state.copyWith(status: CategoryStatus.loading));
    try {
      final genres = await gameRepository.getGenres();
      emit(
        state.copyWith(
          status: CategoryStatus.success,
          categories: genres,
        ),
      );
    } catch (error, stacktrace) {
      print(stacktrace);
      emit(state.copyWith(status: CategoryStatus.error));
    }
  }

  void _mapSelectCategoryEventToState(
      SelectCategory event, Emitter<CategoryState> emit) async {
    emit(
      state.copyWith(
        status: CategoryStatus.selected,
        idSelected: event.idSelected,
      ),
    );
  }
}
```

现在……我如何检查视图中的状态？
好吧，当发出一个状态时，我想用相应的数据重建视图。为此，在我看来，我有一个 BlocBuilder。
在这种情况下，我只想在当前状态成功时重建视图，因此我使用 buildWhen() 来实现（第 11 行）。

```
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:infogames/ui/home/widgets/category_widget/category_barrel.dart';

class CategoriesWidget extends StatelessWidget {
  const CategoriesWidget({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<CategoryBloc, CategoryState>(
      buildWhen: (previous, current) => current.status.isSuccess,
      builder: (context, state) {
        return CategoriesSuccessWidget();
      },
    );
  }
}
```
很酷，当显示此小部件时，用户将能够单击其中一个类别，当这些发生时，我将添加两个事件：
GetGamesByCategory ：按类型过滤游戏。这将由另一个块处理：GamesByCategoryBloc（第 26 行）。我很快就会谈到这个bloc。
SelectCategory：在视图中更改所选项目的颜色和大小。这将使用相同的块处理：CategoryBloc（第 32 行）。

来看下完整的类。

```
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:infogames/repository/models/genre.dart';
import 'package:infogames/ui/home/widgets/category_widget/category_barrel.dart';
import 'package:infogames/ui/home/widgets/games_by_category_widget/games_by_category_barrel.dart';

class CategoriesSuccessWidget extends StatelessWidget {
  const CategoriesSuccessWidget({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<CategoryBloc, CategoryState>(
      builder: (context, state) {
        return SizedBox(
          height: MediaQuery.of(context).size.height * .15,
          child: ListView.separated(
            padding: const EdgeInsets.symmetric(horizontal: 16.0),
            shrinkWrap: true,
            itemBuilder: (context, index) {
              return CategoryItem(
                key: ValueKey('${state.categories[index].name}$index'),
                category: state.categories[index],
                callback: (Genre categorySelected) {
                  context.read<GamesByCategoryBloc>().add(
                        GetGamesByCategory(
                          idSelected: categorySelected.id,
                          categoryName: categorySelected.name ?? '',
                        ),
                      );
                  context.read<CategoryBloc>().add(
                        SelectCategory(
                          idSelected: categorySelected.id,
                        ),
                      );
                },
              );
            },
            scrollDirection: Axis.horizontal,
            separatorBuilder: (_, __) => SizedBox(
              width: 16.0,
            ),
            itemCount: state.categories.length,
          ),
        );
      },
    );
  }
}
```

让我们看一下 state.isSelected 时的视图（选择类别时）。
我使用 BlocSelector 来控制这种情况，当用户单击其中一个类别时，将触发事件并且该 bloc 将发出带有所选类别 id 的状态 isSelected，因此在 bloc选择器，我必须检查这些条件是否为真（第 24 行）以使用新的大小和颜色重建视图（第 35、36 和 39 行）。

```
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:infogames/repository/models/genre.dart';
import 'package:infogames/ui/home/widgets/category_widget/category_barrel.dart';

typedef CategoryCLicked = Function(Genre categorySelected);

class CategoryItem extends StatelessWidget {
  const CategoryItem({
    Key? key,
    required this.category,
    required this.callback,
  }) : super(key: key);

  final Genre category;
  final CategoryCLicked callback;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => callback(category),
      child: BlocSelector<CategoryBloc, CategoryState, bool>(
        selector: (state) =>
            (state.status.isSelected && state.idSelected == category.id)
                ? true
                : false,
        builder: (context, state) {
          return Column(
            children: [
              AnimatedContainer(
                duration: const Duration(milliseconds: 400),
                curve: Curves.easeInOutCirc,
                padding: const EdgeInsets.symmetric(horizontal: 2.0),
                alignment: Alignment.center,
                height: state ? 70.0 : 60.0,
                width: state ? 70.0 : 60.0,
                decoration: BoxDecoration(
                  shape: BoxShape.circle,
                  color: state ? Colors.deepOrangeAccent : Colors.amberAccent,
                ),
                child: Icon(
                  Icons.gamepad_outlined,
                ),
              ),
              SizedBox(height: 4.0),
              Container(
                width: 60,
                child: Text(
                  category.name ?? '',
                  style: TextStyle(
                      fontSize: 10.0,
                      fontWeight: FontWeight.bold,
                      color: Colors.black87),
                  textAlign: TextAlign.center,
                  maxLines: 2,
                  overflow: TextOverflow.ellipsis,
                ),
              )
            ],
          );
        },
      ),
    );
  }
}
```

## GameByCategoryWidget
### GameByCategoryEvent

在这里，我创建了一个事件来按类别过滤所有游戏，并添加了类别名称以将其显示为列表的标题。

```
part of 'games_by_category_bloc.dart';

class GamesByCategoryEvent extends Equatable {
  @override
  List<Object?> get props => [];
}

class GetGamesByCategory extends GamesByCategoryEvent {
  GetGamesByCategory({
    required this.idSelected,
    required this.categoryName,
  });
  final int idSelected;
  final String categoryName;

  @override
  List<Object?> get props => [idSelected, categoryName];
}
```

### GameByCategoryState
和之前的状态类一样，这里我有一个扩展来检查不同的状态，还有一个 copyWith 方法来创建游戏列表和类别名称的新副本。

```
part of 'games_by_category_bloc.dart';

enum GamesByCategoryStatus { initial, success, error, loading }

extension GamesByCategoryStatusX on GamesByCategoryStatus {
  bool get isInitial => this == GamesByCategoryStatus.initial;
  bool get isSuccess => this == GamesByCategoryStatus.success;
  bool get isError => this == GamesByCategoryStatus.error;
  bool get isLoading => this == GamesByCategoryStatus.loading;
}

class GamesByCategoryState extends Equatable {
  const GamesByCategoryState({
    this.status = GamesByCategoryStatus.initial,
    List<Result>? games,
    String? categoryName,
  })  : games = games ?? const [],
        categoryName = categoryName ?? '';

  final List<Result> games;
  final GamesByCategoryStatus status;
  final String categoryName;

  @override
  List<Object?> get props => [status, games, categoryName];

  GamesByCategoryState copyWith({
    List<Result>? games,
    GamesByCategoryStatus? status,
    String? categoryName,
  }) {
    return GamesByCategoryState(
      games: games ?? this.games,
      status: status ?? this.status,
      categoryName: categoryName ?? this.categoryName,
    );
  }
}
```

### GameByCategoryBloc

在这个块中，我将处理调用存储库的事件 GetGamesByCategory 以获取与该类型 ID 匹配的所有游戏。当存储库返回有效数据时，bloc将发出成功的消息，例如状态和列表的新副本以及类别名称，相反，如果结果无效，则bloc将发出错误状态。

```
import 'package:equatable/equatable.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:infogames/repository/game_repository.dart';
import 'package:infogames/repository/models/model_barrel.dart';

part 'games_by_category_event.dart';
part 'games_by_category_state.dart';

class GamesByCategoryBloc
    extends Bloc<GamesByCategoryEvent, GamesByCategoryState> {
  GamesByCategoryBloc({
    required this.gameRepository,
  }) : super(const GamesByCategoryState()) {
    on<GetGamesByCategory>(_mapGetGamesByCategoryEventToState);
  }
  final GameRepository gameRepository;

  void _mapGetGamesByCategoryEventToState(
      GetGamesByCategory event, Emitter<GamesByCategoryState> emit) async {
    try {
      emit(state.copyWith(status: GamesByCategoryStatus.loading));

      final gamesByCategory =
          await gameRepository.getGamesByCategory(event.idSelected);
      emit(
        state.copyWith(
          status: GamesByCategoryStatus.success,
          games: gamesByCategory,
          categoryName: event.categoryName,
        ),
      );
    } catch (error) {
      emit(state.copyWith(status: GamesByCategoryStatus.error));
    }
  }
}
```

完美，下一步是检查视图中的状态以对它们做出反应。

```
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:infogames/ui/widgets/error_widget.dart';
import 'games_by_category_barrel.dart';

class GamesByCategoryWidget extends StatelessWidget {
  const GamesByCategoryWidget({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<GamesByCategoryBloc, GamesByCategoryState>(
      builder: (context, state) {
        return state.status.isSuccess
            ? GameByCategorySuccessWidget(
                categoryName: state.categoryName,
                games: state.games,
              )
            : state.status.isLoading
                ? Center(
                    child: CircularProgressIndicator(),
                  )
                : state.status.isError
                    ? ErrorGameWidget()
                    : const SizedBox();
      },
    );
  }
}
```
如您所见，我根据状态使用三个选项处理视图：
 * 错误：显示常见错误小部件。
 * 正在加载：显示 CircularProgressIndicator。
 * 成功：显示 GameByCategorySuccessWidget。这个小部件负责按类别显示游戏列表。
 这是小部件：

```
import 'package:flutter/material.dart';
import 'package:infogames/repository/models/result.dart';
import 'package:infogames/ui/home/widgets/games_by_category_widget/games_by_category_barrel.dart';

class GameByCategorySuccessWidget extends StatelessWidget {
  const GameByCategorySuccessWidget({
    Key? key,
    required this.categoryName,
    required this.games,
  }) : super(key: key);

  final String categoryName;
  final List<Result> games;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Padding(
          padding: const EdgeInsets.only(
            left: 24.0,
            bottom: 16.0,
          ),
          child: Text(
            categoryName,
            style: TextStyle(
              fontWeight: FontWeight.bold,
              fontSize: 18.0,
            ),
          ),
        ),
        Container(
          height: MediaQuery.of(context).size.height * .2,
          child: ListView.separated(
            padding: const EdgeInsets.only(
              left: 24.0,
              right: 24.0,
            ),
            scrollDirection: Axis.horizontal,
            itemBuilder: (context, index) {
              return GameByCategoryImage(
                name: games[index].name ?? 'No data',
                backgroundImage: games[index].backgroundImage ?? '',
              );
            },
            separatorBuilder: (_, __) => SizedBox(
              width: 25.0,
            ),
            itemCount: games.length,
          ),
        ),
      ],
    );
  }
}
```

最后是主布局的最后一部分，AllGamesWidget。

## AllGamesWidget
### AllGamesEvent

我创建了一个事件来从 API 获取所有游戏。

```
part of 'all_games_bloc.dart';

class AllGamesEvent extends Equatable {
  @override
  List<Object?> get props => [];
}

class GetGames extends AllGamesEvent {
  @override
  List<Object?> get props => [];
}
```

### AllGamesState

和之前一样，我创建了一个扩展来处理所有可能的状态，并且我还有一个 copyWith 方法可以在需要时创建对象的新副本。

```
part of 'all_games_bloc.dart';

enum AllGamesStatus { initial, success, error, loading }

extension AllGamesStatusX on AllGamesStatus {
  bool get isInitial => this == AllGamesStatus.initial;
  bool get isSuccess => this == AllGamesStatus.success;
  bool get isError => this == AllGamesStatus.error;
  bool get isLoading => this == AllGamesStatus.loading;
}

class AllGamesState extends Equatable {
  const AllGamesState({
    this.status = AllGamesStatus.initial,
    Game? games,
  }) : games = games ?? Game.empty;

  final Game games;
  final AllGamesStatus status;

  @override
  List<Object?> get props => [status, games];

  AllGamesState copyWith({
    Game? games,
    AllGamesStatus? status,
  }) {
    return AllGamesState(
      games: games ?? this.games,
      status: status ?? this.status,
    );
  }
}
```
### AllGamesBloc

在这里，我调用存储库，当它返回有效数据时，bloc 将通过游戏列表的副本发出成功，相反，如果存储库返回无效数据，bloc 将发出错误状态。

```
import 'package:equatable/equatable.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:infogames/repository/game_repository.dart';
import 'package:infogames/repository/models/model_barrel.dart';

part 'all_games_event.dart';
part 'all_games_state.dart';

class AllGamesBloc extends Bloc<AllGamesEvent, AllGamesState> {
  AllGamesBloc({
    required this.gameRepository,
  }) : super(const AllGamesState()) {
    on<GetGames>(_mapGetGamesEventToState);
  }

  final GameRepository gameRepository;

  void _mapGetGamesEventToState(
      GetGames event, Emitter<AllGamesState> emit) async {
    try {
      emit(state.copyWith(status: AllGamesStatus.loading));
      final games = await gameRepository.getGames();
      emit(
        state.copyWith(
          status: AllGamesStatus.success,
          games: games,
        ),
      );
    } catch (error) {
      emit(state.copyWith(status: AllGamesStatus.error));
    }
  }
}
```

### AllGamesWidget
这是所有游戏小部件。在这里，我有一个 BlocBuilder 来根据状态重建视图。

```
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:infogames/ui/home/widgets/all_games_widget/all_games_barrel.dart';
import 'package:infogames/ui/widgets/error_widget.dart';

class AllGamesWidget extends StatelessWidget {
  const AllGamesWidget({
    Key? key,
    required this.title,
  }) : super(key: key);

  final String title;

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<AllGamesBloc, AllGamesState>(
      builder: (context, state) {
        return state.status.isSuccess
            ? AllGamesSuccessWidget(
                title: title,
                games: state.games.results,
              )
            : state.status.isLoading
                ? Center(
                    child: CircularProgressIndicator(),
                  )
                : state.status.isError
                    ? ErrorGameWidget()
                    : const SizedBox();
      },
    );
  }
}
```

这些是三种状态：
 * 错误：显示常见错误小部件。
 * 正在加载：显示 CircularProgressIndicator。
 * 成功：显示 AllGamesSuccessWidget。这个小部件负责显示游戏列表。
 
这是小部件：

```
import 'package:flutter/material.dart';
import 'package:infogames/repository/models/result.dart';
import 'package:infogames/ui/home/widgets/all_games_widget/all_games_barrel.dart';

class AllGamesSuccessWidget extends StatelessWidget {
  const AllGamesSuccessWidget({
    Key? key,
    required this.games,
    required this.title,
  }) : super(key: key);

  final List<Result> games;
  final String title;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.stretch,
      children: [
        Padding(
          padding: const EdgeInsets.only(left: 24.0),
          child: Text(
            title,
            style: TextStyle(
              fontWeight: FontWeight.bold,
              fontSize: 18.0,
            ),
          ),
        ),
        Container(
          height:
              ((100 * games.length) + MediaQuery.of(context).size.width) + 24.0,
          child: ListView.separated(
            physics: NeverScrollableScrollPhysics(),
            padding: const EdgeInsets.only(
              left: 24.0,
              right: 24.0,
              top: 24.0,
            ),
            itemBuilder: (context, index) {
              return AllGamesItem(
                game: games[index],
              );
            },
            separatorBuilder: (_, __) => SizedBox(
              height: 20.0,
            ),
            itemCount: games.length,
          ),
        ),
      ],
    );
  }
}
```

### 额外的东西🙌

如果你想有一个日志来知道哪个是当前状态，哪个是下一个要添加的事件，你需要一个 Bloc Observer 类。您应该创建此类并将其初始化到您的主类中。

```
class AppBlocObserver extends BlocObserver {
  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    if (bloc is Cubit) print(change);
  }

  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print(transition);
  }
}

// main class

void main() async {
  BlocOverrides.runZoned(
    () => runApp(const MyApp()),
    blocObserver: AppBlocObserver(),
  );
}
```

## 结论

必须为您的Flutter应用程序使用良好的状态管理。 Flutter bloc 是一个很好的选择，你可以看到它使用起来并不复杂，而且很容易理解如何使用它的主要概念。此外，它还为您提供了很多管理视图或小部件的方法。
就个人而言，我喜欢创建具有特定逻辑的小块，以使我的代码更清晰和可维护，而不是管理很多事情的大块，但如果你的逻辑需要你也可以这样做。

感谢您阅读本文，我希望您现在可以更好地理解 Flutter bloc 的工作原理，并且能够毫无问题地将这个惊人的库添加到您的项目中。如果这对您有帮助，我将非常感谢您的反馈和鼓掌！👏