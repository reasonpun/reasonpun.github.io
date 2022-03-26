---
title: Flutter的MVVM设计模式
date: 2022-03-12 23:08:44
categories:
- 设计模式
- Flutter
tags:
- Flutter
- MVVM
---

{% asset_img 1.png 示意图 width="400" %}

模型-视图-视图模型 (MVVM) 在软件开发方面是一种非常成熟的架构模式。 MVVM 有什么特别之处？
<!--more-->
{% asset_img 2.jpg 示意图 width="400" %}

我们需要在应用程序中放置一种架构，在 UI 和业务逻辑之间进行通信。 
MVVM 就是其中之一，它能够将业务逻辑与 UI 分离，这看起来很容易做到，但请相信我，如果你错误的使用了架构中的某些内容或做任何不准确的事情，那么请做好在应用程序开发的后期阶段遇到大麻烦的准备吧！最终您将在应用程序中进行很多令人抓狂的事情，以此来补救你的程序。

在这里，我将通过一个简单的示例来解释 MVVM，它将为您提供足够的知识来在您的应用程序中实现。

MVVM 对于将业务逻辑从视图移动到 ViewModel 和 Model 很有用。 ViewModel 是 View 和 Model 之间的中介，它携带所有用户事件并返回结果。

关键的好处是允许视图和模型之间的真正分离以及您从中获得的更大效率的实现效果。这实际上意味着当您的模型需要更改时，可以轻松更改它而无需视图，反之亦然。

应用 MVVM 会产生三个关键问题：

 * 可维护性：- 表示层和逻辑松散耦合，因为此代码易于维护和重用。由于代码库会随着时间的推移而增加，这将帮助您区分它们。
 * 可测试性：- ViewModel 比代码隐藏或事件驱动代码更容易进行单元测试。感谢具有分离逻辑的 MVVM😀
 * 可扩展性：- 此架构为您提供保证，使代码能够在一段时间内获得可扩展性。但请记住，一定要先保持组件的可重用性。

 因此，让我们分别浏览每个组件并尝试了解其用途。请在参考以下几点的同时查看图表，您将对流程有一个清晰的了解。

 ### Model
该模型主要工作任务是实时获取数据或与数据库相关的查询。
该层可以包含业务逻辑、代码验证等。该层与 ViewModel 交互以获取本地数据或实时数据。数据是响应 ViewModel 给出的。

### ViewModel
ViewModel 是 View 和 Model 之间的中介，它接受所有用户事件并向 Model 请求数据。一旦 Model 有数据，它就会返回给 ViewModel，然后 ViewModel 将该数据通知给 View。
ViewModel 可以被多个 View 使用，这意味着单个 ViewModel 可以为多个 View 提供数据。

{% asset_img 3.png 示意图 width="400" %}

### View
视图是用户与屏幕上显示的小部件交互的地方。这些用户事件请求一些导航到 ViewModel 的操作，而 ViewModel 的其余部分完成这项工作。一旦 ViewModel 拥有所需的数据，它就会更新 View。

现在，我们将通过演示 MVVM 架构的示例，为了通知数据，我们将使用 Provider 状态机制。

{% asset_img 4.png 示意图 width="400" %}

MediaService.dart

```dart
import 'dart:convert';
import 'dart:io';
import 'package:meta/meta.dart';

import 'package:http/http.dart' as http;
import 'package:mvvm_flutter_app/model/apis/app_exception.dart';

class MediaService {
  final String _baseUrl = "https://itunes.apple.com/search?term=";

  Future<dynamic> get(String url) async {
    dynamic responseJson;
    try {
      final response = await http.get(_baseUrl + url);
      responseJson = returnResponse(response);
    } on SocketException {
      throw FetchDataException('No Internet Connection');
    }
    return responseJson;
  }

  @visibleForTesting
  dynamic returnResponse(http.Response response) {
    switch (response.statusCode) {
      case 200:
        dynamic responseJson = jsonDecode(response.body);
        return responseJson;
      case 400:
        throw BadRequestException(response.body.toString());
      case 401:
      case 403:
        throw UnauthorisedException(response.body.toString());
      case 500:
      default:
        throw FetchDataException(
            'Error occured while communication with server' +
                ' with status code : ${response.statusCode}');
    }
  }
}
```

MediaRepository.dart

```dart
import 'package:mvvm_flutter_app/model/media.dart';
import 'package:mvvm_flutter_app/model/services/media_service.dart';

class MediaRepository {
  MediaService _mediaService = MediaService();

  Future<List<Media>> fetchMediaList(String value) async {
    dynamic response = await _mediaService.get(value);
    final jsonData = response['results'] as List;
    List<Media> mediaList =
        jsonData.map((tagJson) => Media.fromJson(tagJson)).toList();
    return mediaList;
  }
}
```

MediaViewModel.dart

```dart
import 'package:flutter/cupertino.dart';
import 'package:mvvm_flutter_app/model/apis/api_response.dart';
import 'package:mvvm_flutter_app/model/media.dart';
import 'package:mvvm_flutter_app/model/media_repository.dart';

class MediaViewModel with ChangeNotifier {
  ApiResponse _apiResponse = ApiResponse.loading('Fetching artist data');

  Media _media;

  ApiResponse get response {
    return _apiResponse;
  }

  Media get media {
    return _media;
  }

  /// Call the media service and gets the data of requested media data of
  /// an artist.
  Future<void> fetchMediaData(String value) async {
    try {
      List<Media> mediaList = await MediaRepository().fetchMediaList(value);
      _apiResponse = ApiResponse.completed(mediaList);
    } catch (e) {
      _apiResponse = ApiResponse.error(e.toString());
      print(e);
    }
    notifyListeners();
  }

  void setSelectedMedia(Media media) {
    _media = media;
    notifyListeners();
  }
}
```

HomScreen.dart

```dart
import 'package:flutter/material.dart';
import 'package:mvvm_flutter_app/model/apis/api_response.dart';
import 'package:mvvm_flutter_app/model/media.dart';
import 'package:mvvm_flutter_app/view/widgets/player_list_widget.dart';
import 'package:mvvm_flutter_app/view/widgets/player_widget.dart';
import 'package:mvvm_flutter_app/view_model/media_view_model.dart';

import 'package:provider/provider.dart';

class HomeScreen extends StatefulWidget {
  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  @override
  void initState() {
    // TODO: implement initState
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    final _inputController = TextEditingController();
    ApiResponse apiResponse = Provider.of<MediaViewModel>(context).response;
    List<Media> mediaList = apiResponse.data as List<Media>;
    return Scaffold(
        appBar: AppBar(
          title: Text('Media Player'),
        ),
        body: Column(
          children: <Widget>[
            Padding(
              padding: const EdgeInsets.symmetric(vertical: 10.0),
              child: Row(
                children: <Widget>[
                  Expanded(
                    child: Container(
                      margin: EdgeInsets.symmetric(horizontal: 20.0),
                      decoration: BoxDecoration(
                        color: Theme.of(context).accentColor.withAlpha(50),
                        borderRadius: BorderRadius.circular(30.0),
                      ),
                      child: TextField(
                          style: TextStyle(
                            fontSize: 15.0,
                            color: Colors.grey,
                          ),
                          controller: _inputController,
                          onChanged: (value) {},
                          onSubmitted: (value) {
                            if (value.isNotEmpty) {
                              Provider.of<MediaViewModel>(context)
                                  .setSelectedMedia(null);
                              Provider.of<MediaViewModel>(context,
                                      listen: false)
                                  .fetchMediaData(value);
                            }
                          },
                          decoration: InputDecoration(
                            border: InputBorder.none,
                            enabledBorder: InputBorder.none,
                            focusedBorder: InputBorder.none,
                            prefixIcon: Icon(
                              Icons.search,
                              color: Colors.grey,
                            ),
                            hintText: 'Enter Artist Name',
                          )),
                    ),
                  ),
                ],
              ),
            ),
            mediaList != null && mediaList.length > 0
                ? Expanded(
                    child: PlayerListWidget(mediaList, (Media media) {
                    Provider.of<MediaViewModel>(context)
                        .setSelectedMedia(media);
                  }))
                : Expanded(
                    child: Center(
                      child: Text('Search the song by Artist'),
                    ),
                  ),
            if (Provider.of<MediaViewModel>(context).media != null)
              Align(
                  alignment: Alignment.bottomCenter,
                  child: PlayerWidget(
                    function: () {
                      setState(() {});
                    },
                  )),
          ],
        ));
  }
}
```

MVVM 现在被大量使用，因为它支持事件驱动的方法，这与许多组件是基于事件执行密切相关。