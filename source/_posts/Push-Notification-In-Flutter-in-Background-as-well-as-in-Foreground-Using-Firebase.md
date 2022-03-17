---
title: 使用 Firebase 在后台和前台推送通知
date: 2022-03-17 21:50:00
categories:
- Flutter
tags:
- Flutter
- Notification
- Firebase
---

{% asset_img 1.png 示意图 width="400" %}

在应用程序中，我们需要集成推送通知。
虽然我们成功集成了它，但有时在应用程序未运行或在后台打开时发送推送通知会出现问题或者导致应用程序的崩溃。
所以，本篇内容中我们将在Flutter中使用 firebase ，使用它的通知适用于所有状态：前台、后台，甚至在应用程序关闭（未运行）时。

开始吧：）

```
Firebase 和推送通知设置
```

### 第一步
 * 为您的应用创建 firebase 项目并在 android/app 文件夹中添加googleservices.json 文件。
 * 在你的 Flutter 项目中安装 firebase_messaging 和 firebase_core。

```
firebase_messaging: ^11.2.11
firebase_core: any
```

 * 像往常一样在 build.gradle（在应用程序级别和 android 级别）文件中进行 firebase 初始化设置。
 * 对于推送通知，请在应用级 build.gradle 文件的依赖项下添加以下行（有时我们会遇到 multidex 问题，因此最好在 defaultConfig{} 中添加 multidex）
默认配置下：

```
multiDexEnabled true
```

在依赖项下添加如下内容：

```
...
dependencies{
...
implementation ‘com.google.firebase:firebase-inappmessaging-display:19.1.5’
implementation 'com.android.support:multidex:1.0.3'
...
}
```

### 第二步

在 android/app/src/main/kotlin 中创建 Application.kt 文件（简单地说您在该文件夹中看到 MainActivity.kt 文件的位置创建 Application.kt 文件）并复制并粘贴以下代码：

```
package <your_package_name>
import io.flutter.app.FlutterApplication
import io.flutter.plugin.common.PluginRegistry
import io.flutter.plugins.firebase.messaging.FlutterFirebaseMessagingBackgroundService
import io.flutter.plugins.firebase.messaging.FlutterFirebaseMessagingPlugin

class Application: FlutterApplication(), PluginRegistry.PluginRegistrantCallback {
  override fun onCreate() {
    super.onCreate()FlutterFirebaseMessagingBackgroundService.setPluginRegistrant(this)
  }
  override fun registerWith(registry:PluginRegistry) {
    FlutterFirebaseMessagingPlugin.registerWith(registry?.registrarFor(“io.flutter.plugins.firebasemessaging.FirebaseMessagingPlugin”))
  }
}
```

### 第三步

在 pubspec.yaml 文件中添加 flutter_local_notifications 包

```
flutter_local_notifications:
```

### 第四步

在 android/app/src/main/res/AndroidManifest.xml 中添加以下行

```
<application
  android:name=".Application"
  ...>
  <activity
     ....>
       .....
       <intent-filter>
         <action android:name="FLUTTER_NOTIFICATION_CLICK"/>
         <category android:name="android.intent.category.DEFAULT"/>
       </intent-filter>
  </activity>
...
</application>
```

```
到目前为止，推送通知已完成，我们将开始编码部分。
```

### 第五步

在 main.dart 文件的 main 函数中添加这些行。

```
WidgetsFlutterBinding.ensureInitialized();
await Firebase.initializeApp();
```

### 第六步

创建一个页面并通过分别导入这些包来初始化 firebase_messaging 和 flutter_local_notifications。

```
import ‘package:firebase_messaging/firebase_messaging.dart’;
import ‘package:flutter_local_notifications/flutter_local_notifications.dart’;
// create an instance 
FirebaseMessaging messaging = FirebaseMessaging.instance;
FlutterLocalNotificationsPlugin fltNotification;
```

### 第七步

在这里，我们编写了在终端中获取 fcm 令牌的代码，以便我们可以在步骤 I 中创建的 firebase 项目中的云消息传递下使用这些令牌。

```
void pushFCMtoken() async {
  String token=await messaging.getToken();
  print(token);
}
```

### 第八步

在这一步中，我们编写用于接收通知格式和设置的代码。

```
void initMessaging() {
  var androiInit = AndroidInitializationSettings(‘@mipmap/ic_launcher’);//for logo
  var iosInit = IOSInitializationSettings();
  var initSetting=InitializationSettings(android: androiInit,iOS:
   iosInit);
  fltNotification = FlutterLocalNotificationsPlugin();
  fltNotification.initialize(initSetting);
  var androidDetails =
   AndroidNotificationDetails(‘1’, ‘channelName’, ‘channel 
    Description’);
  var iosDetails = IOSNotificationDetails();
  var generalNotificationDetails =
   NotificationDetails(android: androidDetails, iOS: iosDetails);
FirebaseMessaging.onMessage.listen((RemoteMessage message) {     RemoteNotification notification=message.notification;
   AndroidNotification android=message.notification?.android;
   if(notification!=null && android!=null){
     fltNotification.show(
       notification.hashCode, notification.title, notification.
       body, generalNotificationDetails);
}
});
}
```

### 第九步

最终代码将是：

```
import 'package:flutter/material.dart';
import 'package:font_awesome_flutter/font_awesome_flutter.dart';
import 'package:flutter_datetime_picker/flutter_datetime_picker.dart';

class selClass extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: sel(),
    );
  }
}

class sel extends StatefulWidget {
  @override
  _selState createState() => _selState();
}

class _selState extends State<sel> {
  String _date = "Not set";

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(child:SingleChildScrollView(
    child: Column(
    children: <Widget>[
    Container(
    color: Colors.grey,
      width: MediaQuery
          .of(context)
          .size
          .width,
      height: MediaQuery
          .of(context)
          .size
          .height / 10,
      child: Center(child: Text("Attendance",
        style: TextStyle(fontWeight: FontWeight.bold, fontSize: 20.0),),),
    ),
    SizedBox(height: 30.0,),
    Padding(
    padding: const EdgeInsets.all(16.0),
    child: Container(
    child: Column(
    mainAxisSize: MainAxisSize.max,
    mainAxisAlignment: MainAxisAlignment.center,
    children: <Widget>[
    RaisedButton(
    shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(5.0)),
    elevation: 4.0,
    onPressed: () {
    DatePicker.showDatePicker(context,
    theme: DatePickerTheme(
    containerHeight: 210.0,
    ),
    showTitleActions: true,
    minTime: DateTime(2000, 1, 1),
    maxTime: DateTime(2022, 12, 31),
    onConfirm: (date) {
    print('confirm $date');
    _date = '${date.year} - ${date.month} - ${date.day}';
    setState(() {});
    },
    currentTime: DateTime.now(),
    locale: LocaleType.en);
    },
    child: Container(
    alignment: Alignment.center,
    height: 50.0,
    child: Row(
    mainAxisAlignment: MainAxisAlignment.spaceBetween,
    children: <Widget>[
    Row(
    children: <Widget>[
    Container(
    child: Row(
    children: <Widget>[
    Icon(
    Icons.date_range,
    size: 18.0,
    color: Colors.teal,
    ),
    Text(
    " $_date",
    style: TextStyle(
    color: Colors.teal,
    fontWeight: FontWeight.bold,
    fontSize: 18.0),
    ),
    ],
    ),
    )
    ],
    ),
    Text(
    "  Change",
    style: TextStyle(
    color: Colors.teal,
    fontWeight: FontWeight.bold,
    fontSize: 18.0),
    ),
    ],
    ),
    ),
    color: Colors.white,
    ),
    SizedBox(
    height: 20.0,
    ),
    ]
    ,
    )
    ,
    )
    )
    ]))));
  }
}
```

当您运行此代码时，您将在控制台中获得 fcm_token，然后复制该令牌并执行以下步骤：

### 第十步

打开 firebase 项目并转到云消息部分并编写通知标题和正文，如下所示：

{% asset_img 2.png 示意图 width="400" %}

完成上图后，点击发送消息，你会得到如下所示：

{% asset_img 3.png 示意图 width="400" %}

在这里粘贴 fcm_token（您在前面的步骤中从终端复制的内容）并点击测试按钮。
在完成所有这些步骤之后，您将在您的移动设备中收到通知，无论是应用程序处于运行状态还是在后台运行或已关闭。

最后，推送通知有效，您可以在未来的应用程序中使用此功能。
谢谢 ：）