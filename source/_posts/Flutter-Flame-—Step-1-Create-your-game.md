---
title: Flutter & Flame — 第 1 步：创建您的游戏
date: 2022-03-24 19:22:47
categories:
- 深入浅出系列
- Flutter
tags:
- Flutter
- Flame
- Game
---

{% asset_img 1.jpeg 第一次运行新项目 width="400" %}

从一个想法到一个商店就绪的游戏，所有这些都是用 Flutter 和 Flame 制作的。 加入我们本系列的第一部分，学习如何使用 Flutter 建立一个 Flame 项目并在屏幕上绘制你的第一个元素。

<!--more-->

### 介绍

这是创建从零到商店就绪状态的手机游戏的系列文章中的第一篇。 我将从一个没有代码的想法开始，到一个准备好发布的手机游戏应用程序。
为确保您可以跟进，让我推荐以下几点：

 * 至少为 Android 平台安装和配置 Flutter SDK
 * 您选择的 IDE，我将使用 Visual Studio Code
 * Flutter基础知识及其核心概念

### 工具

如果您需要设置 Flutter，请按照以下链接为您选择的平台进行配置（我尽可能选择 Linux）：

[https://flutter.dev/docs/get-started/install](https://flutter.dev/docs/get-started/install)

上述指南还包括 IDE 的安装，我将再次使用 Visual Studio Code。 如果您想进一步阅读，请查看以下 Microsoft 的链接：

[https://code.visualstudio.com/](https://code.visualstudio.com/)

### Flame Engine

{% asset_img 2.png 第一次运行新项目 width="400" %}

完整的教程将使用火焰引擎和相关包。 
Flame Engine 是一个用 Dart 为 Flutter 编写的开源 2D 游戏引擎，支持 Web 和移动平台以及桌面。

[https://flame-engine.org/](https://flame-engine.org/)

Flame 提供了多种元素来构建您的游戏——如果需要，该工具集甚至可以使用物理引擎进行扩展。 
要获得有关所有已发布包的概述，请查看以下内容：

[https://pub.dev/publishers/flame-engine.org/packages](https://pub.dev/publishers/flame-engine.org/packages)

首先，我们将单独使用 Flame 包。

Moon Lander——手机上的复古乐趣

{% asset_img 3.jpeg 第一次运行新项目 width="400" %}

每个游戏都基于一个概念或想法，我会选择 Lunar Lander 类型的游戏。 
如果这是您第一次听说它，请查看下面链接的 Wikipedia 文章。 但为了简短起见：

 * 您需要将宇宙飞船安全降落在另一个行星或月球的表面
 * 有限的燃料和重力使您的着陆成为挑战
 * 你必须在燃料耗尽之前选择一个合适的着陆点

[https://en.wikipedia.org/wiki/Lunar_Lander_%28video_game_genre%29](https://en.wikipedia.org/wiki/Lunar_Lander_%28video_game_genre%29)

早期的游戏是用简单的绘图构建的（我真的很喜欢简单的风格）。
但是，我们将使用精灵（绘制的图像）来展示火焰要完成的工作。 
如果您想了解一些复古的东西，请查看这个很酷的实现：
[http://moonlander.seb.ly/](http://moonlander.seb.ly/)

### 来吧，让我们开始编码！

{% asset_img 4.jpeg 第一次运行新项目 width="400" %}

#### 起步

让我们通过运行以下命令来生成一个新的 Flutter 项目：

```
flutter create moonlander
```

Flutter 将创建包含所有必需文件和配置的项目结构，这还包括一个示例屏幕（著名的计数器应用程序）。 
在我们处理任何代码文件（*.dart）之前，让我们在我们选择的代码编辑器中打开文件夹并将我们的依赖项添加到 pubspec.yaml 文件中：

```
flame: ^1.0.0
```

依赖项的部分应该大致如下所示，请注意“flutter create”工具将添加一些进一步的行和注释，你现在可以忽略它：

```
dependencies:  
  flutter:    
    sdk: flutter  
  flame: ^1.0.0
```

为确保您确实下载了新添加的 Flame 包，只需在控制台中运行：

```
flutter pub get
```

### 快速、干净的代码

{% asset_img 5.jpeg 第一次运行新项目 width="400" %}

如果您从 GitHub 下载代码，您可能会在 pubspec.yaml 文件的 dev_dependencies 部分下发现另一个依赖项，名为：

```
very_good_analysis
```

此软件包可帮助您根据准则验证代码，使其干净并遵循许多项目共享的标准。 
你会发现多个可用于 Flutter（和 Dart）的“linter”项目：

[https://pub.dev/packages?q=lint](https://pub.dev/packages?q=lint)

Dart 项目中还有一个官方的：

[https://pub.dev/packages/lints](https://pub.dev/packages/lints)

您不是必须要在项目中使用 linter，它不会为您的游戏添加任何功能，但它支持您保持代码干净并符合社区标准。

### 回到游戏开发

如果你现在就开始项目，你仍然会有前面提到的计数器应用程序示例。 让我们把它改成我们自己游戏的开始。 
将以下内容复制到您的 main.dart 文件中。 
不用担心，您会遇到一些错误，我们会立即逐步修复。

```dart
import 'dart:async';

import 'package:flame/flame.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';
import 'package:moonlander/components/rocket_component.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Flame.device.setLandscape();

  final game = MoonlanderGame();

  runApp(GameWidget(game: game));
}

/// This class encapulates the whole game.
class MoonlanderGame extends FlameGame with HasCollidables {
  @override
  Future<void> onLoad() async {
    unawaited(add(RocketComponent(position: size / 2, size: Vector2.all(20))));

    return super.onLoad();
  }
}
```

#### 9-14 行

我们通过调用来确保 Flutter 已准备就绪：

```
WidgetsFlutterBinding.ensureInitialized();
```

完成后，我们通知 Flame，我们的游戏应该以横向模式运行。 这是使用内置的火焰设备类完成的：

```
await Flame.device.setLandscape();
```

Flame 在这个实用程序类中提供了更多功能和帮助程序，如果您想进一步阅读，请查看下面的文档。

[https://docs.flame-engine.org/1.0.0-releasecandidate.16/util.html](https://docs.flame-engine.org/1.0.0-releasecandidate.16/util.html)

最后两行创建我们的游戏并将其提供给 GameWidget。 GameWidget 将确保我们在游戏中所做的任何事情都呈现在屏幕上，它提供了更多有用的方法，我们将在稍后探索。

#### 第 18–24 行

这 6 行是目前完整的游戏，我们创建一个新类并扩展 FlameGame 和 mixin HasCollidables。 在游戏内部，我们重写了 onLoad 函数来为游戏添加一个组件。

与任何其他 Flutter App 一样，您的游戏由单一元素构建：组件（您可以说它们是游戏的小部件）。 onLoad 事件只是众多生命周期事件之一，它在游戏启动期间被调用一次。
一般而言，创建和加载您稍后将使用的内容是一件好事。 不用担心，我们将在下一篇文章中更深入地介绍所有这些元素，目前，这就是我们需要了解的全部内容。

### 添加您的第一个组件

创建一个名为 components 的目录并添加一个包含以下内容的新文件：rocket_component.dart：

```dart
import 'package:flame/components.dart';
import 'package:flame/extensions.dart';
import 'package:flame/geometry.dart';
import 'package:flutter/material.dart';

///
class RocketComponent extends PositionComponent with Hitbox, Collidable {
  /// Create a new Rocket component at the given [position].
  RocketComponent({
    required Vector2 position,
    required Vector2 size,
  }) : super(position: position, size: size);

  @override
  Future<void> onLoad() async {
    await super.onLoad();
    addHitbox(HitboxRectangle());
  }

  @override
  void render(Canvas canvas) {
    super.render(canvas);

    // Temporary render item
    renderHitboxes(canvas, paint: Paint()..color = Colors.white);
  }
}

```

Rocket 组件将是我们的播放器对象，它将被绘制在屏幕上并需要与其他元素交互。 您在屏幕上绘制的所有内容都将以给定大小绘制在给定位置。 
Flame 提供 PositionComponent 类来处理绘图。 
为了启用交互，在我们的例子中是碰撞，我们将 Hitbox 和 Collidable Mixin 添加到组件中。 
我们将在下一篇文章中详细介绍，目前，我们只需要大致了解类和 mixin 的用途。

### 组件也有生命周期

与我们的主游戏类类似，每个组件都有一个生命周期。 我们正在使用 onLoad 事件来实现与我们的游戏相同的效果，让一切准备就绪。
我们通知 Flame 我们想要一个围绕我们元素的矩形碰撞框（第 17 行）。 
这是确保以后正确碰撞所必需的。 
为了在屏幕上显示某些内容，我们在 render 方法中渲染 hitbox。 
默认情况下，hitbox 不是游戏的可见部分，出于调试和测试的原因，绘制它们可能会有所帮助。

###  起飞

如果您启动应用程序，您应该会在手机上看到以下结果：

{% asset_img 6.png 第一次运行新项目 width="400" %}

恭喜，您成功地使用 Flame 和 Flutter 启动了游戏！ 
这还不算多，但我们正朝着正确的方向前进 :) 
白色矩形是我们的火箭组件的命中框，绘制到屏幕的中心。 
中心是通过将可用屏幕大小除以 2 来计算的（在 main.dart 中）：

```
RocketComponent(position: size / 2, size: Vector2.all(20))
```

Hitbox 可以有不同的形状，我们决定使用矩形，这就是为什么 Flames 内置方法 (renderHitboxes) 会绘制一个。

玩转 Paint() 对象，也许你可以改变颜色，或者只是在矩形的外部绘制但不填充它。 这可能对以后有用；）

### 结论

{% asset_img 7.jpeg 第一次运行新项目 width="400" %}

您已经使用 Flutter 创建了自己的 Flame 游戏项目的基础。 
我们概述了我们游戏的想法以及我们想要创建的内容，并了解到 Flame 游戏也是小部件。 
每个游戏都由组件构建并遵循生命周期事件。 
游戏组件可以相互交互，并以给定的位置和大小绘制在屏幕上。
除了游戏开发，我们对 linter 有一个侧面的看法，可以帮助您生成适用于任何即将出现的 Flutter 项目的干净且可共享的代码。

### 下一步是什么？

在下一篇文章中，我们将介绍有关游戏生命周期的更多内容，并了解渲染和（尚未显示的）更新功能背后的一些含义。 
随着第二篇文章的结束，我们将有一个坚实的基础开始为我们的小火箭组件创造一个危险的环境:)