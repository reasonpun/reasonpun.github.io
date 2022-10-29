---
title: Flutter高级BLoC和服务操作
date: 2022-09-22 10:27:24
categories:
- Flutter
tags:
- Flutter
- BLoC
---

> 在这篇文章中，我将解释Bloc和Vexana库的使用，它是一个状态管理解决方案，处于高级水平。我在这个项目中也使用了Provider。我将在下一篇文章中对此进行解释。

{% asset_img 1.png 示意图 width="400" %}

Vexana是一个由 Veli Bacık 基于Dio库之上编写的。它对服务操作非常有用。
与其他库不同。

<!--more-->

我们创建模型类的方法如下：

```dart
import 'package:vexana/vexana.dart';

class QuoteModel extends INetworkModel<QuoteModel> {
  String? id;
  String? author;
  String? content;
  List<String>? tags;
  String? authorSlug;
  int? length;
  String? dateAdded;
  String? dateModified;

  QuoteModel({
    this.id,
    this.author,
    this.content,
    this.tags,
    this.authorSlug,
    this.length,
    this.dateAdded,
    this.dateModified,
  });

  @override
  QuoteModel fromJson(Map<String, dynamic> json) => QuoteModel.fromJson(json);

  factory QuoteModel.fromJson(Map<String, dynamic> json) {
    return QuoteModel(
      id: json['_id'] as String?,
      author: json['author'] as String?,
      content: json['content'] as String?,
      tags: (json['tags'] as List<dynamic>?)?.map((e) => e as String).toList(),
      authorSlug: json['authorSlug'] as String?,
      length: json['length'] as int?,
      dateAdded: json['dateAdded'] as String?,
      dateModified: json['dateModified'] as String?,
    );
  }

  @override
  Map<String, dynamic>? toJson() => _toJson();

  Map<String, dynamic> _toJson() {
    return {
      'id': id,
      'author': author,
      'content': content,
      'tags': tags,
      'authorSlug': authorSlug,
      'length': length,
      'dateAdded': dateAdded,
      'dateModified': dateModified,
    };
  }
}
```

这里唯一的区别是一个从INetworkModel延伸出来的模型类。

你可以使用这个链接，它也支持Vexana的Json到Dart操作：[https://dartj.web.app/#/](https://dartj.web.app/#/)

现在我们可以开始编写服务操作了。

```dart
class VexanaManager extends NetworkManager {
  VexanaManager()
      : super(
            options: BaseOptions(
              baseUrl: AppConstants.baseUrl,
              followRedirects: true,
            ),
            isEnableLogger: true,
            isEnableTest: true);
}
```

就像在Dio库中一样，我们在这里定义NetworkManager类。它给了我们一些选择。我们以后将使用这个VexanaManager类。这是一个重要的定义。

让我们继续!

```dart
abstract class IQuoteService {
  final INetworkManager networkManager;

  IQuoteService(this.networkManager);

  Future<QuoteModel> getAllQuotes();
}

class QuoteService extends IQuoteService {
  QuoteService(INetworkManager networkManager) : super(networkManager);

  @override
  Future<QuoteModel> getAllQuotes() async {
    var response = await networkManager.send(
      AppConstants.baseUrl,
      parseModel: QuoteModel(),
      method: RequestType.GET,
    );
    return response.data;
  }
}
```

现在让我们仔细研究一下。
我们有一个名为IQuoteService的抽象类。
在命名时，抽象类通常以I开头，在这里我们将定义我们的方法。我们可以不创建这个抽象类而直接在一个具体的类上定义它，但这不是最佳做法。

无论如何，首先我们从INetworkManager中创建一个对象（就像从Dio库中创建一个对象）。INetworkManager类是Vexana库带给我们的一个类。

接下来，定义唯一的方法，这个应用程序的唯一服务是要从API中获取数据。

现在我们创建一个具体的类，在这个类中我们将定义这个方法，从刚才创建的IQuoteService抽象类中扩展这个类（请连同这些文字一起查看代码）。

这里我们使用send方法向API发送请求：networkManager.send()。
send()期望我们提供各种参数。
其中一个是发送请求的路径（AppConstants.baseUrl）。
另一个是我们将使用的模型（QuoteModel），以及请求类型。
我们使用GET方法是因为我们将为这个应用程序接收数据。如果可以发送一个数据，我们将使用POST等。

好了，服务过程的最后一步是存储过程。

```dart
class QuotesRepository {
  final quoteService = QuoteService(VexanaManager());

  Future<QuoteModel> getAllQuotes() {
    return quoteService.getAllQuotes();
  }
}
```

在这里我们创建一个类。这个类将在后面的BLoC结构中使用。
一个提示：你可以把Repository类想象成Bloc和服务之间的一个层。
我们在最开始创建的VexanaManager是一个定义Vexana的类。

不多说了，让我们看看我们的Bloc结构。

> 我在这里的目的不是要深入研究Bloc的结构。我的目的是帮助那些已经有一些Bloc知识的人在高级水平上做到这一点。

```dart
abstract class QuoteEvent extends Equatable {
  const QuoteEvent();
}

class FetchQuotes extends QuoteEvent {
  @override
  List<Object> get props => [];
}
```

让我们先看一下我们的事件文件。在这个应用程序中只有一个事件，那就是从API中获取数据。提醒一下：事件是指用户的行为，状态是指他因这些行为而得到的状态。关键句子是：事件进入块结构，然后状态退出。

现在来检查一下状态文件。

```dart
abstract class QuoteState extends Equatable {
  const QuoteState();
}

class QuoteInitial extends QuoteState {
  @override
  List<Object?> get props => [];
}

class QuoteLoading extends QuoteState {
  @override
  List<Object> get props => [];
}

class QuoteLoaded extends QuoteState {
  final QuoteModel quoteModel;

  const QuoteLoaded(this.quoteModel);

  @override
  List<Object> get props => [quoteModel];
}

class QuoteError extends QuoteState {
  final String message;

  const QuoteError(this.message);

  @override
  List<Object> get props => [];
}
```

让我们想一想。我们的事件可能是什么？
首先，它可能处于初始状态。所以还没有什么。
然后用户按下按钮，想显示一个新的报价。
在这种情况下，从API获取数据的过程开始发挥作用，QuoteLoading()就从这里开始。我们从API中获取数据。
正如你所猜测的，这就是QuoteLoaded()发挥作用的地方。如果发生错误，则运行QuoteError状态。

提醒一下。在QuoteLoaded类中，我们定义了将从服务中获取的数据类型。我们将接收类型为QuoteModel的数据。

```dart
class QuoteBloc extends Bloc<QuoteEvent, QuoteState> {
  QuoteBloc() : super(QuoteInitial()) {
    final QuotesRepository quotesRepository = QuotesRepository();

    on<FetchQuotes>((event, emit) async {
      emit(QuoteLoading());
      try {
        var response = await quotesRepository.getAllQuotes();
        emit(QuoteLoaded(response));
      } catch (e) {
        emit(QuoteError(e.toString()));
      }
    });
  }
}
```

是的，由于上面所有的定义，你现在可以猜到这里发生了什么。
提醒一下：我们不要忘记在这里定义我们在服务操作中使用的资源库类 :)

然后我们把这个Bloc结构整合到接口编码中。

谢谢你的阅读。我希望它能对你有所帮助。保持健康!

<!-- https://medium.com/@bedirhanssaglam/flutter-advanced-bloc-and-service-operations-c3cd4a1b400f -->