---
title: An Introduction to MVVM in Flutter
date: 2022-03-12 21:35:03
categories:
- 设计模式
tags:
- Flutter
- MVVM
---

```
在这篇文章中，我们将使用 MVVM 设计模式来编写一个完整的 Flutter 应用程序
```

{% asset_img 1.jpeg 示意图 width="400" %}

默认情况下，Flutter 应用程序不使用任何特定的设计模式。
这意味着开发人员需要自己负责选择和实现适合他们需求的模式。 
由于Flutter的声明性特性，使得其成为 MVVM 设计模式的理想候选者。

### 了解 MVVM

MVVM 代表模型-视图-视图模型。
基本思想是创建一个向视图提供数据的视图模型。视图可以使用视图模型提供的数据来填充自己。创建视图模型层允许您编写模块化代码，这些代码可以被多个视图使用。

MVVM 设计模式起源于微软。 

MVVM 大量用于编写 Windows Presentation Foundation (WPF) 应用程序。 MVVM 模式是 PresentationModel 的一个花哨名称。

由于设计模式与平台无关，因此它可以与任何框架一起使用，包括 Flutter。


在这篇文章中，我们将创建一个电影应用程序，它将根据输入的关键字获取电影并将其显示给用户。这个应用程序将基于 MVVM 原则创建。
在深入研究代码之前，请查看下面的动画以了解我们将要构建的应用程序。

{% asset_img 2.gif 示意图 width="400" %}

### 实现网络服务

我们将使用[OMDb API](http://www.omdbapi.com/)来获取电影。确保您在他们的网站上注册以获取 API 密钥。如果没有有效的 API 密钥，您将无法成功执行请求。

现在您已经注册了 OMDb 服务并收到了您的密钥，我们可以继续开发 Web 服务/客户端。 Web 服务使用 HTTP 包发出网络请求并下载 JSON 响应，但您可以随意使用任何您想要的网络包。

下载 JSON 后，将其灌入到电影模型以获取电影对象列表，如下所示：

```
import 'package:movies_app/models/movie.dart';
import 'package:http/http.dart' as http; 

class Webservice {

  Future<List<Movie>> fetchMovies(String keyword) async {

    final url = "http://www.omdbapi.com/?s=$keyword&apikey=YOURAPIKEYHERE";
    final response = await http.get(url);
    if(response.statusCode == 200) {

       final body = jsonDecode(response.body); 
       final Iterable json = body["Search"];
       return json.map((movie) => Movie.fromJson(movie)).toList();

    } else {
      throw Exception("Unable to perform request!");
    }
  }
}
```
电影模型实现如下：

```
class Movie {

  final String title; 
  final String poster; 

  Movie({this.title, this.poster});

  factory Movie.fromJson(Map<String, dynamic> json) {
    return Movie(
      title: json["Title"], 
      poster: json["Poster"]
    );
  }

}
```

电影只是由标题和海报组成。它还公开了一个 fromJson 函数，它允许我们基于返回的JSON格式数据创建一个 Movie 对象。
我们使用模型来定义我们的电影对象，但实际上，它们是用来实现为数据传输对象 (DTO) 的服务的。
至此，我们拥有了所有需要的数据，下一步就是将其显示在屏幕上。在将数据显示在用户界面之前，我们必须创建负责向视图提供数据的视图模型。

### 实现视图模型

尽管您可以通过创建单个视图模型来获得所需的结果，但我们将在我们的应用程序中创建两个单独的视图模型。

一个视图模型 MoviesListViewModel 将用来表示电影列表数据并显示在整个屏幕上。
第二个视图模型 MovieViewModel 将表示一个单独的电影——它将显示在视图中。

视图模型的实现如下所示：

```
class MovieListViewModel extends ChangeNotifier {

  List<MovieViewModel> movies = List<MovieViewModel>(); 

  Future<void> fetchMovies(String keyword) async {
   // TODO 
  }

}

class MovieViewModel {

  final Movie movie; 

  MovieViewModel({this.movie});

  String get title {
    return this.movie.title; 
  }

  String get poster {
    return this.movie.poster; 
  }

}
```

MovieListViewModel 由 movies 属性组成，该属性将返回 MovieViewModel 对象的列表。 MovieViewModel 接受一个 Movie 对象并将标题和海报作为只读属性返回。下一步是使用 Web 服务获取电影。

### 设置 ChangeNotifier 和 ChangeNotifierProvider

MovieListViewModel 中的 fetchMovies 方法会通过使用Web服务，从 OMDb API 检索电影。
实现如下图所示：

```
class MovieListViewModel extends ChangeNotifier {

  List<MovieViewModel> movies = List<MovieViewModel>(); 

  Future<void> fetchMovies(String keyword) async {
    final results =  await Webservice().fetchMovies(keyword);
    this.movies = results.map((item) => MovieViewModel(movie: item)).toList();
    print(this.movies);
    notifyListeners(); 
  }

}
```

我们更新了 MovieListViewModel 以从 ChangeNotifier 继承。 ChangeNotifier 允许我们发布更改通知，视图可以使用它来更新自身。
在我们使用网络服务获取电影后，我们调用 notifyListeners 函数，通知所有订阅者/听众。目前没有人收听，所以没有人得到消息：电影已经下载完毕。

为了用更新的 MovieListViewModel 通知视图，我们必须使用 ChangeNotifierProvider，它是 Provider 包的一部分。通过在pubspec.yaml文件中添加依赖来添加provider包，如下图：

```
dependencies:
  flutter:
    sdk: flutter

  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^0.1.2
  http: ^0.12.0+2
  provider: ^3.2.0
```

接下来，我们需要找到一个从提供者那里注入值的好地方。
我们正在谈论的值是 MovieListViewModel 的一个实例，因为它扩展了 ChangeNotifier 并向侦听器发布通知。

在我们的例子中，我们可以使用 main.dart 文件并将值注入到 MovieListPage。这意味着 MovieListViewModel 将可用于 MovieListPage 及其所有子项。

```
void main() => runApp(App());

class App extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: "Movies",
      home: 
      ChangeNotifierProvider(
        create: (context) => MovieListViewModel(), 
        child: MovieListPage(),
      )
    );
  }

}
```

完美！最后一步是在屏幕上显示数据。

### 显示电影

对于我们的应用程序，我们将要求用户输入关键字并按键盘上的返回键。这将调用 MovieListViewModel 上的 fetchMovies 方法，如下所示：

```
 Container(
    padding: EdgeInsets.only(left: 10),
    decoration: BoxDecoration(
        color: Colors.grey, 
        borderRadius: BorderRadius.circular(10)
    ),
    child: TextField(
        controller: _controller,
        onSubmitted: (value) {
        if(value.isNotEmpty) {
            vm.fetchMovies(value);
            _controller.clear();
        }
        },
```

fetchMovies 将根据关键字获取所有电影并触发 notifyListeners。为了获取 MovieListViewModel 的更新实例，我们将从provider包中获得帮助，如下所示：

```
@override Widget build(BuildContext context) { 
final vm = Provider.of<MovieListViewModel>(context);
```

通过在 build 方法中添加 Provider，我们确保无论何时触发 notifyListener，我们都可以访问 MovieListViewModel 的实例。

现在，如果您运行您的应用程序，它将根据关键字获取电影并将其显示在用户界面上。

如果您想在页面加载时执行初始提取，则可以将 StatelessWidget 更新为 StatefulWidget 并从 initState 方法内部调用 fetchMovies，如下所示：

```
class _MovieListPageState extends State<MovieListPage> {

  final TextEditingController _controller = TextEditingController(); 

  @override
  void initState() {
    super.initState();
    // you can uncomment this to get all batman movies when the page is loaded
    Provider.of<MovieListViewModel>(context, listen: false).fetchMovies("batman");
  }
```

请注意，我们为 Provider 传递了 listen: false ，这意味着这只是一次调用，并且 Provider 不会跟踪更改。

谢谢阅读！