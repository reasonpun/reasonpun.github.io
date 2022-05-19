---
title: Flutter中的请求权限服务的实现
date: 2022-05-17 08:23:17
categories:
- Flutter
- Permissions
tags:
- Flutter
- Permissions
---

{% asset_img 1.jpeg 示意图 width="400" %}

当你想在用户使用过程中获取很多很多权限的时候，请求权限通常会成为一个混乱的任务。
为了开始清理所有的权限代码，我们将把它包装在一个带有专门权限请求功能的服务中。

<!--more-->

#### 设置
在本教程中，我们将使用permissions_handler包来请求我们的权限，所以让我们把它添加到pubspec中。

```yaml
# Permission checking
permission_handler: ^9.2.0
```

我们将创建函数来请求两种类型的权限：我们将获取位置以及联系人权限。
首先，我们需要告诉操作系统，我们的应用程序将使用这些权限。

#### 安卓
在AndroidManifest中，为这两个功能添加uses-permission标签。

```xml
<uses-permission android:name="android.permission.READ_CONTACTS" />
<uses-permission android:name="android.permission.WRITE_CONTACTS" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

#### iOS
在info.plist文件中添加密钥和你的信息

```xml
<key>NSContactsUsageDescription</key>
<string>This app requires contacts access to function.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>This app requires access to your location when in use to show relevan information.</string>
<key>NSLocationAlwaysUsageDescription</key>
<string>This app requires always on access to to your location to notifiy you when are near a store.</string>
```

#### 实施
PermissionsService上将有一个专门的函数来处理你所需要的权限。
这样，当用户想使用需要的功能时，就可以调用它。
通过提供回调，它也将很容易为每个函数（组）编写自定义逻辑。
创建一个名为permissions_service.dart的新文件，在其中，我们要做一个类，它有一个来自包的PermissionHandler的实例。

```dart
import 'package:permission_handler/permission_handler.dart';
class PermissionsService {
  final PermissionHandler _permissionHandler = PermissionHandler();
}
```

然后我们可以添加一个通用函数，接收一个PermissionGroup来请求我们想要的权限。

```dart
...
 Future<bool> _requestPermission(PermissionGroup permission) async {
    var result = await _permissionHandler.requestPermissions([permission]);
    if (result[permission] == PermissionStatus.granted) {
      return true;
    }
    return false;
  }
...
```

而使用这个函数，我们可以为每个权限创建特定的函数。

```dart
...
  /// Requests the users permission to read their contacts.
  Future<bool> requestContactsPermission() async {
    return _requestPermission(PermissionGroup.contacts);
  }
  /// Requests the users permission to read their location when the app is in use
  Future<bool> requestLocationPermission() async {
    return _requestPermission(PermissionGroup.locationWhenInUse);
  }
...
```

#### 自定义许可逻辑

很多时候，当涉及到权限的时候，我们想做一些自定义的事情，或者想再次提示用户允许我们的权限。因为我们有专门的功能，我们现在可以用自定义的方式处理每个权限请求。
比方说，应用程序需要访问联系人才能工作，类似于WhatsApp。我们可以提供一个函数，当权限被拒绝时就会被调用，这样我们就可以在外面显示一个对话框。

我们将传入一个onPermissionDenied函数，当用户拒绝许可时，该函数将被调用。

```dart
/// Requests the users permission to read their contacts.
  Future<bool> requestContactsPermission({Function onPermissionDenied}) async {
    var granted = await _requestPermission(PermissionGroup.contacts);
    if (!granted) {
      onPermissionDenied();
    }
    return granted;
  }
```

现在在外面，你可以传入你的函数，当它被拒绝时显示你的对话框，然后如果用户选择再次请求它 :)

#### 拥有权限
另一件事是检查应用程序是否已经拥有权限。
我们将创建同样的设置，一个通用的函数接收一个PermissionGroup，然后为特定的权限使用专用函数。
你不必这样做，我只是觉得这样做更容易维护，而且它使外部代码对PermissionHandler包的依赖性降低。

```dart
...
  Future<bool> hasContactsPermission() async {
    return hasPermission(PermissionGroup.contacts);
  }
  Future<bool> hasPermission(PermissionGroup permission) async {
    var permissionStatus =
        await _permissionHandler.checkPermissionStatus(permission);
    return permissionStatus == PermissionStatus.granted;
  }
...
```

#### 使用方法
在代码中，我们现在可以像这样使用服务来请求权限。

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: Scaffold(
        body: Center(
          child: MaterialButton(
            color: Colors.yellow[300],
            child: Text('Request contacts permission'),
            onPressed: () {
              PermissionsService().requestContactsPermission(
                  onPermissionDenied: () {
                print('Permission has been denied');
              });
            },
          ),
        ),
      ));
  }
}
```

服务应该被注入或定位，如图所示，只使用提供者或像这样的架构中的提供者和get_it。
将你的功能包装在一个服务中，可以消除你的代码和第三方实现细节之间的任何关系，所以对我来说，这一直是一个首选解决方案。

<!-- https://medium.com/flutter-community/request-permissions-in-flutter-as-a-consumable-service-e6cd243f882f -->