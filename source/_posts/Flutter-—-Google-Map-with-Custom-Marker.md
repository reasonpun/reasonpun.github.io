---
title: Flutter - 带有自定义标记的谷歌地图📍
date: 2022-10-29 16:21:16
categories:
- Flutter
tags:
- Flutter
- GoogleMap
---

让我们把谷歌地图添加到您的Flutter应用中，用一个花哨的标记来取代无聊的标记。

{% asset_img 1.png 示意图 width="400" %}

<!--more-->

#### 获取API密钥
如果您想在您的Flutter应用程序中使用谷歌地图，您需要在谷歌地图平台配置一个API项目。

 * 进入[谷歌地图平台](https://cloud.google.com/maps-platform/) > 创建新项目或使用现有项目。
 * 在库页面 > 搜索 "Maps SDK"。
 * 点击Maps SDK for iOS，然后点击Enable。
 * 点击Maps SDK for Android，然后点击Enable
 * 在Credentials页面，点击Create credentials > API key。(创建的API密钥对话框显示您新创建的API密钥）。
 * 新的API密钥被列在凭证页面的API密钥下。要重新命名它，请点击编辑图标。(专业提示：在生产中使用API密钥之前要对其进行限制。）

#### 将谷歌地图Flutter包作为依赖项添加
在Flutter中，包允许你添加额外的功能。运行此命令
```
flutter pub add google_maps_flutter
```

or add the package under dependencies

```
dependencies:
    google_maps_flutter: ^2.2.1
```

#### 为安卓应用添加API密钥
为了给安卓应用添加一个API密钥，编辑android/app/src/main中的AndroidManifest.xml文件。
在应用程序节点内添加一个单一的元数据条目，其中包含在上一步中创建的API密钥。

```
<manifest ...
  <application ...
    <meta-data android:name="com.google.android.geo.API_KEY"
               android:value="YOUR KEY HERE"/>
```

Also set the minSdkVersion to 20 in android/app/build.gradle

```
android {
    defaultConfig {
        minSdkVersion 20
    }
}
```

#### 为iOS应用程序添加API密钥
为了给iOS应用程序添加一个API密钥，请编辑ios/Runner中的AppDelegate.swift文件。将整个代码替换为以下内容👇

```objc
import UIKit
import Flutter
import GoogleMaps  // Add this import

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GeneratedPluginRegistrant.register(with: self)

    // TODO: Add your Google Maps API key
    GMSServices.provideAPIKey("YOUR-API-KEY")

    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```
#### 创建谷歌地图屏幕
现在是在屏幕上获取地图的时候了。
我们需要一个初始CameraPosition来显示地图。它可以是用户的当前位置，但为了简单起见，我们使用了一个固定值。

```dart
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
class MapScreen extends StatefulWidget {
  const MapScreen({super.key});
@override
  State<MapScreen> createState() => _MapScreenState();
}
class _MapScreenState extends State<MapScreen> {
  LatLng initialLocation = const LatLng(37.422131, -122.084801);
// ToDo: add custom marker
@override
  Widget build(BuildContext context) {
    return Scaffold(
      body: GoogleMap(
        initialCameraPosition: CameraPosition(
          target: initialLocation,
          zoom: 14,
        ),
        // ToDO: add markers
      ),
    );
  }
}
```

{% asset_img 2.png 示意图 width="400" %}

#### 在地图上添加标记
标记对于识别任何特定的位置很有用。你可以在地图上添加多个标记。替换为：用这个添加标记👇

```dart
markers: {
  Marker(
    markerId: const MarkerId("marker1"),
    position: const LatLng(37.422131, -122.084801),
    draggable: true,
    onDragEnd: (value) {
      // value is the new position
    },
    // To do: custom marker icon
  ),
  
  Marker(
    markerId: const MarkerId("marker2"),
    position: const LatLng(37.415768808487435, -122.08440050482749),
  ),
},
```

如果你允许用户改变标记的位置，那么就把可拖动设置为 "true"，默认为 "false"。 onDragEnd为你提供新的位置LatLng。

{% asset_img 3.png 示意图 width="400" %}

#### 在地图上设置自定义图像标记
在我的资产中添加了一个标记图片，然后创建一个方法来设置markerIcon。替换为：用以下代码添加自定义标记👇

```dart
BitmapDescriptor markerIcon = BitmapDescriptor.defaultMarker;
@override
void initState() {
  addCustomIcon();
  super.initState();
}
void addCustomIcon() {
  BitmapDescriptor.fromAssetImage(
          const ImageConfiguration(), "assets/Location_marker.png")
      .then(
    (icon) {
      setState(() {
        markerIcon = icon;
      });
    },
  );
}
```

回到第一个标记，markerId是marker1。设置```icon = markerIcon```

```dart
Marker(
  markerId: const MarkerId("marker1"),
  position: const LatLng(37.422131, -122.084801),
  draggable: true,
  onDragEnd: (value) {
    // value is the new position
  },
  icon: markerIcon,
),
```
{% asset_img 4.png 示意图 width="400" %}

完整的代码 🥳
```dart
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';

class MapScreen extends StatefulWidget {
  const MapScreen({super.key});

  @override
  State<MapScreen> createState() => _MapScreenState();
}

class _MapScreenState extends State<MapScreen> {
  LatLng initialLocation = const LatLng(37.422131, -122.084801);
  BitmapDescriptor markerIcon = BitmapDescriptor.defaultMarker;

  @override
  void initState() {
    addCustomIcon();
    super.initState();
  }

  void addCustomIcon() {
    BitmapDescriptor.fromAssetImage(
            const ImageConfiguration(), "assets/Location_marker.png")
        .then(
      (icon) {
        setState(() {
          markerIcon = icon;
        });
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: GoogleMap(
        initialCameraPosition: CameraPosition(
          target: initialLocation,
          zoom: 14,
        ),
        markers: {
          Marker(
            markerId: const MarkerId("marker1"),
            position: const LatLng(37.422131, -122.084801),
            draggable: true,
            onDragEnd: (value) {
              // value is the new position
            },
            icon: markerIcon,
          ),
          Marker(
            markerId: const MarkerId("marker2"),
            position: const LatLng(37.415768808487435, -122.08440050482749),
          ),
        },
      ),
    );
  }
}
```

<!-- https://medium.com/flutter-community/flutter-google-map-with-custom-marker-ea1555a37342 -->