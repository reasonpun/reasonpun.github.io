---
title: Flutter - 在谷歌地图中突出显示任何区域
date: 2022-10-29 16:16:57
categories:
- Flutter
tags:
- Flutter
- GoogleMap
---

我们已经向您展示了如何在Flutter应用程序中添加谷歌地图和自定义标记。
让我们更进一步，向您展示如何在您的地图上添加一个圆圈，或识别一个特定的区域市、州或国家。

{% asset_img 1.png 示意图 width="400" %}

<!--more-->

请注意。这里已经假设您知道如何将谷歌地图添加到Flutter应用中。如果没有，请查看我之前的文章👉[Flutter - 带有自定义标记的谷歌地图📍](https://pangz.fun/Flutter-%E2%80%94-Google-Map-with-Custom-Marker.html)。

#### 初始设置

现在是时候开始了! 
这是一个简单的屏幕，上面有一个谷歌地图小部件，设置了initialCameraPosition和一个标记。

```dart
class GoogleMapScreen extends StatefulWidget {
  const GoogleMapScreen({Key? key}) : super(key: key);
@override
  State<GoogleMapScreen> createState() => _GoogleMapScreenState();
}
class _GoogleMapScreenState extends State<GoogleMapScreen> {
  final Completer<GoogleMapController> _controller = Completer();
LatLng intialLocation = const LatLng(23.762912, 90.427816);
@override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          Expanded(
            child: GoogleMap(
              initialCameraPosition: CameraPosition(
                target: intialLocation,
                zoom: 15.6746,
              ),
              onMapCreated: (controller) {
                _controller.complete(controller);
              },
              markers: {
                Marker(
                  markerId: const MarkerId("1"),
                  position: intialLocation,
                ),
              },
              // ToDo: Add Circle
              // ToDo: Add polygon
            ),
          ),
        ],
      ),
    );
  }
}
```

{% asset_img 2.png 示意图 width="400" %}

#### 添加一个圈圈
第一个问题是如何在我们的地图上添加一个圆圈。用以下代码
```ToDo: Add Circle```用下面的代码添加圆圈👇
```dart
circles: {
  Circle(
    circleId: CircleId("1"),
    center: intialLocation,
    radius: 420,
    strokeWidth: 2,
    fillColor: Color(0xFF006491).withOpacity(0.2),
  ),
},
```
center设置圆的位置。我使用intialLocation，这样标记就会是圆的中间点。半径值为430比较合适的，但是如果地图没有被放大，那么就使用一个更大的值。
为了避免圆周围出现巨大的边界，我将strokeWidth设置为2。

{% asset_img 3.png 示意图 width="400" %}

#### 突出一个特定的区域
并不总是需要画一个圆，也许你只需要标记某个区域。
为了达到这个目的，我们使用谷歌地图的多边形。需要用下面的代码替换```//ToDo: Add polygon```

```dart
polygons: {
  Polygon(
    polygonId: const PolygonId("1"),
    fillColor: const Color(0xFF006491).withOpacity(0.1),
    strokeWidth: 2,
    points: const [
      LatLng(23.766315, 90.425778),
      LatLng(23.764691, 90.424767),
      LatLng(23.761916, 90.426862),
      LatLng(23.758532, 90.428588),
      LatLng(23.758825, 90.429228),
      LatLng(23.763698, 90.431324),
    ],
  ),
},
```

{% asset_img 4.png 示意图 width="400" %}

请看下效果：
{% asset_img 5.gif 示意图 width="400" %}

如果这篇文章对你有帮助，请点个赞👏 呗。

<!-- https://theflutterway.medium.com/flutter-highlight-any-area-in-google-maps-4f6380a203c3 -->