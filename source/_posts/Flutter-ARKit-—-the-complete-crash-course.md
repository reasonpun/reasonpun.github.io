---
title: Flutter ARKit — the complete crash course
date: 2022-10-21 08:50:20
categories:
- Flutter
tags:
- Flutter
- ARKit
---

{% asset_img 1.jpeg 示意图 width="400" %}

有没有想过在Flutter应用程序中使用AR？
Well，ARKit可以帮助你。这是一个非常容易使用的软件包，只要你了解它是如何工作的。
这就是为什么我今天要向你介绍这个软件包的原因! 让我们开始吧!

<!--more-->

注意。ARKit只适用于iOS设备（因为ARKit是由苹果开发的）。对于安卓设备，你可以使用ar_core_flutter_plugin。

Happy reading!

#### 安装
创建我们的应用程序后，我们要做的第一件事是添加 arkit_plugin。为此，我们将使用命令
```flutter pub add arkit_plugin```。你应该熟悉这些命令，否则，我建议你再查一下Flutter的基础知识。

现在，让我们配置一些文件以在iOS设备上使用ARKit。

首先，允许你的应用程序使用摄像头。为此，我们必须提供一个```NSCameraUsageDescription```。

{% asset_img 2.png 示意图 width="400" %}

在这之后，我们必须更新Podfile。
最低支持的iOS版本是11.0，但如果你需要使用图像锚，请使用11.3。
对于图像跟踪配置或面部跟踪，将其设置为12.0，身体跟踪的最低版本为13.0。

{% asset_img 3.png 示意图 width="400" %}

#### 使用ARKit
首先，我们必须在一个StatefulWidget中创建一个ARitController。

{% asset_img 4.png 示意图 width="400" %}

现在，让我们来创建我们的ARKit视图。
为此，我们将使用ARKitSceneView。它有一个名为onARKitViewCreated的属性。
我们可以在那里调用一个函数并初始化我们的ARKITController。

{% asset_img 5.png 示意图 width="400" %}

但目前，我们什么都看不到。
让我们添加一个FloatingActionButton，调用一个函数，向akitController添加一个对象。

{% asset_img 6.png 示意图 width="400" %}

ARKitNode还有很多自定义选项，你可以在[这里](https://pub.dev/documentation/arkit_plugin/latest/arkit_plugin/ARKitNode-class.html)找到。

现在的ARKit可以是这样的:
{% asset_img 8.gif 示意图 width="400" %}

我们强烈建议你看一下ARKit的例子。
你可以在那里学习如何在节点上添加你自己的图像，使用自定义灯光设置，追踪脸部、身体和其他。
有很多东西可以发现，所以看一下这个[列表](https://pub.dev/packages/arkit_plugin#examples)，看看你喜欢什么。你简直可以用ARKit做任何事情。

在发布到App Store之前，一些给开发者们的建议：

> 该插件支持TrueDepth API。如果你不使用它，一定要通过修改你的Podfile文件来删除任何TrueDepth功能，否则，你的应用程序将被苹果拒绝。

{% asset_img 9.png 示意图 width="400" %}

#### 进一步阅读与结论
在这篇文章中，你已经了解了AR包 "ARKit "的基础知识。你已经看到了它是多么容易使用，以及它是多么强大。

如果你使用Freezed、Isar或Flutter Hooks等包，你可以展开ARKit的全部力量。

谢谢你的阅读，祝你有个愉快的一天！

<!-- https://tomicriedel.medium.com/4007f8b7f945 -->