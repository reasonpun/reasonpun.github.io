---
title: Nylo v3.x - Flutter的最新微框架
date: 2022-11-03 21:41:16
categories:
- Flutter
tags:
- Flutter
- Nylo
---

Nylo v3.x已经面世，可以帮助你更容易地构建Flutter应用程序。

{% asset_img 1.png 示意图 width="400" %}

<!--more-->

我很高兴地分享Nylo v3.x终于发布了。
这个版本是疯狂的，有一些小的变化，但也有一些大的补充，使开发Flutter应用程序变得轻而易举。

#### 首先，让我们开始吧
你可以通过以下方式下载Nylo。

 * [https://nylo.dev](https://nylo.dev)
 * git clone [https://github.com/nylo-core/nylo.git](https://github.com/nylo-core/nylo.git)

打开项目后，构建并运行该应用程序。
{% asset_img 2.png 示意图 width="400" %}

祝贺你! 现在让我们深入了解一下v3.x的新更新。

#### 突出的特点
 * 编写API服务的一种新的优雅方式
 * 事件+监听器
 * 可启动的提供者
 * 帮助者
 * 新网站
为了保持故事的短小精悍，我将只介绍上述内容。你可以在[https://nylo.dev](https://nylo.dev)，查看文档以了解更多。

#### API服务
我将从最酷的新增功能（在v3.x中）开始，它是新的网络助手。
在你的项目中，导航到"/lib/app/networking/"。在这里你可以为你的应用程序添加你的API服务。

#### 提出HTTP请求
在你的API服务中，使用网络方法来建立你的API请求。

```dart
class ApiService extends BaseApiService {
  ApiService({BuildContext? buildContext}) : super(buildContext);
  @override
  String get baseUrl => "https://jsonplaceholder.typicode.com";
  Future<User?> fetchUser() async {
    return await network<User>(
      request: (request) => request.get("/users/1"),
    );
  }
```

注意：```请求参数```是一个[Dio](https://pub.dev/packages/dio)实例。

使用泛型，你可以使用Nylo的模型解码器从响应中自动返回一个对象。

#### 引入模型解码器
模型解码器是Nylo v3.x中引入的一个新概念。它们使你很容易返回你的对象，如下面的例子:

```dart
class ApiService extends BaseApiService {
  ...
Future<User?> fetchUser() async {
    return await network<User>(
        request: (request) => request.get("/users/1"),
    );
 }
```
文件: lib/config/decoders.dart

```dart
final modelDecoders = {
  User: (data) => User.fromJson(data), 
  
  // add your model and handle the return of the object
List<User>: (data) => List.from(data).map((json) => User.fromJson(json)).toList(),
};
```

#### 这是怎么做到的？
在幕后，Nylo会查看你的config/decoders.dart文件，并使用modelDecoders变量来决定解码时使用哪种 "类型"。
下面是一个例子:

```dart
final modelDecoders = {
  List<User>: (data) => List.from(data).map((json) => User.fromJson(json)).toList(),
  User: (data) => User.fromJson(data),
};
```

 * 第一个键是你的模型的```type```，例如，使用上面的例子，将是``User``和```List<User>```。
 * ```(data)```参数将包含来自你的API请求的HTTP响应体。

接下来，创建一种方法，初始化你的模型。

```dart
class User {
  String? name;
  String? email;
  User.fromJson(dynamic data) {
    this.name = data['name'];
    this.email = data['email'];
  }
}
```

现在你可以像下面的例子一样，使用```network```自动返回你的模型的正确表示。
```dart
class ApiService extends BaseApiService {
  ...
Future<User?> fetchUser() async {
    return await network<User>(
        request: (request) => request.get("/users/1"),
    );
 }
Future<List<User>?> fetchUsers() async {
    return await network<List<User>>(
        request: (request) => request.get("/users"),
    );
 }
```

#### 没有模型解码器的HTTP请求
你也可以使用网络助手的handleSuccess参数来处理HTTP请求。

下面是一些例子:
```dart
class ApiService extends BaseApiService {
  ...
  // Example: returning an Object
  Future<User?> findUser() async {
    return await network(
        request: (request) => request.get("/users/1"),
        handleSuccess: (Response response) { 
        // response - Dio Response object
          dynamic data = response.data;
          return User.fromJson(data);
        }
    );
  }
  // Example: returning a String
  Future<String?> findMessage() async {
    return await network(
        request: (request) => request.get("/message/1"),
        handleSuccess: (Response response) { 
        // response - Dio Response object
          dynamic data = response.data;
          if (data['name'] == 'Anthony') {
            return "It's Anthony";
          }
          return "Hello world"; 
        }
    );
  }
  // Example: returning a bool
  Future<bool?> updateUser() async {
    return await network(
        request: (request) => request.put("/user/1", data: {"name": "Anthony"}),
        handleSuccess: (Response response) { 
        // response - Dio Response object
          dynamic data = response.data;
          if (data['status'] == 'OK') {
            return true;
          }
          return false;
        }
    );
  }
```

注意，你不需要在```network```上指定任何通用类型。在```handleSuccess```回调里面，你只需要自己处理返回。

你也可以使用```handleFailure```参数来处理失败的HTTP响应。

#### 在你的项目中使用API服务

```dart
class _MyHomePageState extends NyState<MyHomePage> {
ApiService _apiService = ApiService();
 @override
 init() async {
  User? user = await _apiService.getUser();
  print(user); // User? instance
    // or
 User? user = await api<ApiService>((request) => request.getUser());
 print(user); // User? instance
...
```

#### 拦截器
如果你对拦截器感到陌生，不要担心。它们提供了一种在你的HTTP请求被发送之前拦截它们的方法。

拦截器提供了以下的回调。

 * onRequest: 在发送之前修改请求。
 * onResponse: 对响应数据做一些处理。
 * onError。处理HTTP请求中的错误。
Nylo允许你为你的API服务添加新的拦截器，就像下面的例子一样。

```dart
class ApiService extends BaseApiService {
  ApiService({BuildContext? buildContext}) : super(buildContext);
  ...
  @override
  final interceptors = {
    LoggingInterceptor: LoggingInterceptor(),
    // Add more interceptors for the API Service e.g. below
    // BearerAuthInterceptor: BearerAuthInterceptor(),
  };
...
```

你可以在项目中的如下文件查看拦截器的实现: “lib/app/networking/dio/interceptors/logging_interceptor.dart”

#### 创建一个新的API服务

```
flutter pub run nylo_framework:main make:api_service shop
```

这条命令会在目录/lib/app/networking/*下创建一个新的API服务，名称为shop_api_service.dart。

#### 增加模型选项

```
flutter pub run nylo_framework:main make:api_service shop --model="Shop"
```

模型选项--model="Shop"告诉Nylo为你新创建的API服务添加以下方法：查找、创建、删除、更新和获取全部内容。

#### 增加了URL标志

```
flutter pub run nylo_framework:main make:api_service shop --url="https://myapi-baseurl.com"
```

```url```选项告诉Nylo将baseUrl设置为你提供的值。

#### 事件+监听器
当你需要在你的应用程序中发生一些事情后处理逻辑时，事件是强大的。Nylo提供了一个简单的事件实现，允许你调用为该事件注册的监听器。
监听器可以从事件的有效载荷中执行逻辑。
下面是一个事件的例子，我们设想我们拥有一个销售T恤的电子商务应用程序。
当用户成功付款后，我们想派发一个事件来处理以下事情。

 * 清除用户的购物车
 * 使用谷歌标签管理器来记录事件，以便进行分析。

```dart
import 'package:nylo_framework/nylo_framework.dart';
import 'package:google_tag_manager/google_tag_manager.dart' as gtm;
class PaymentSuccessfulEvent implements NyEvent {
final listeners = {
    SanitizeCheckoutListener: SanitizeCheckoutListener(),
    GTMPurchaseListener: GTMPurchaseListener(),
  };
}
class SanitizeCheckoutListener extends NyListener {
  handle(dynamic event) async {

    await NyStorage.store('cart', null); // clear the cart
  }
}
class GTMPurchaseListener extends NyListener {
  handle(dynamic event) async {
    // Get payload from event
    Order order = event['order'];
   // Push event to gtm (Google tag manager).
    gtm.push({
      'ecommerce': {
        'purchase': {
          'actionField': {
            'id': order.id,
            'revenue': order.revenue,
            'tax': order.tax,
            'shipping': order.shipping
          },
          'products': order.line_items
        }
      }
    });
  }
}
```

在Nylo中调度一个事件。
```dart
stripePay(List<Product> products) async {
  // create the order
  Order? order = await api<OrderApiService>((api) => api.createOrder(products));
  // dispatch the event
  await event<PaymentSuccessfulEvent>(data: {'order': order});
...
```

你可以在 "lib/app/events/"目录下添加事件。

#### 创建新事件

```
flutter pub run nylo_framework:main make:event message_created_event
```

#### Providers
在Nylo中，当你的应用程序首次运行时，提供程序将从你的main.dart文件中启动。所有的提供者都在"/lib/app/providers/"中，你可以修改这些文件或使用Metro创建你的提供者。

对提供者的需求是什么？
随着项目的发展，你可能需要在你的应用程序运行前初始化更多的包、类或代码。

提供者提供了一个整洁的解决方案，使你的main.dart文件不致臃肿。让我们来看看一些开箱即注册的提供者。

```dart
// file: lib/app/providers/app_provider.dart
class AppProvider implements NyProvider {

  boot(Nylo nylo) async {
    await NyLocalization.instance.init(
        localeType: localeType,
        languageCode: languageCode,
        languagesList: languagesList,
        assetsDirectory: assetsDirectory,
        valuesAsMap: valuesAsMap);

    return null;
  }
}
// file: lib/app/providers/route_provider.dart
class RouteProvider implements NyProvider {

  boot(Nylo nylo) async {
    nylo.addRouter(appRouter());
    return nylo;
  }
}
```

这两个提供者都有助于启动应用程序。一个是添加路由，另一个是初始化项目的本地化。

您可以修改您的提供者或创建新的提供者，以促进您的Flutter应用程序中的新服务。

#### 创建一个新的provider

```
flutter pub run nylo_framework:main make:provider firebase_provider
```

#### Helpers
一些新增加的小功能。

whenEnv - 这允许你在你的应用程序的环境处于某种状态时执行一些代码。下面是一个例子。

```dart
Your .env file
APP_NAME=MyApp
APP_ENV=developing // < -- this is set to "developing"
// Register page for users
class _RegisterPageState extends NyState<RegisterPage> {
  TextEditingController _txtNameController = new TextEditingController();
  TextEditingController _txtEmailController = new TextEditingController();
  
  @override
  init() async {
    whenEnv('developing', perform: () {
      var faker = new Faker();
      _txtEmailController.text = faker.internet.email();
      _txtNameController.text = faker.person.name();
      // E.g. Fill the fields with fake data to save time
    });
  }
```
希望上面的例子能告诉你如何使用whenEnv帮助器来使你的测试/开发工作更容易。

Backpack - 这个类被设计用来存储即时的小尺寸数据。它不需要异步等待，这使得它非常适合存储用户认证令牌、API会话等。

Here’s an example:

```dart
// storing a string
Backpack.instance.set('user_api_token', 'a secure token');
// storing an object
User user = User();
Backpack.instance.set('user', user);
// storing an int
Backpack.instance.set('my_lucky_no', 7);
// reading data
Backpack.instance.read('user_api_token'); // a secure token
Backpack.instance.read('user'); // User instance
Backpack.instance.read('my_lucky_no'); // 7
```

你可以在你的应用程序的任何地方使用Backpack，例如授权一个API请求。

```dart
class ApiService extends BaseApiService {
  ...
  Future<dynamic> accountDetails() async {
    return await network(
        request: (request) {
      String userToken = Backpack.instance.read('user_api_token');
      // Set auth header
      request.options.headers = {
          'Authorization': "Bearer " + userToken
      };
          
      return request.get("/account/1");
       
      },
    );
  }
}
```

注意：在Backpack类中设置的值不会持久化。如果你的应用程序需要，你可能需要实现NyStorage和Backpack的组合。

#### 新网站
官方网站已经被重新设计，以使学习Nylo的体验更好。你会发现，文档页面是全新的，看起来很简约。

{% asset_img 3.png 示意图 width="400" %}

#### 总结
非常兴奋，v3.x终于发布了。如果你是Nylo的新手，它是新项目的一个伟大的起点。它让你忘记小事，这样你就可以专注于创作。

一如既往，感谢您的阅读。

<!-- https://medium.com/@agordn52/nylo-v3-x-flutters-latest-micro-framework-updates-2d063c0b5945 -->