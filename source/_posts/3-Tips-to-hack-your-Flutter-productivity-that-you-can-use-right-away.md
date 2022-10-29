---
title: 现在就开始提升你的Flutter生产力吧!
date: 2022-05-24 08:47:43
categories:
- Flutter
tags:
- Flutter
- hack
---

{% asset_img 1.png 示意图 width="400" %}

今天，我将向您介绍3个技巧，这些技巧将使您的Flutter开发更上一层楼
如果您喜欢这个内容，请给点个赞呗！
让我们马上开始吧!

<!--more-->

### 修复所有错误

在VSCode中看到这个消息是不是非常令人恼怒呢？

通常是不需要的导入，缺少const关键词等等。但是，难道没有一些方法可以在几秒钟内修复整个项目中的这些错误吗？是的，有的，而且比你想象的要容易得多。只要在终端导航到你的项目，输入
```
dart fix --apply
```
一切就都解决了；)

### 更好的错误屏幕

每个人，真的是每个人都讨厌这个错误屏幕。
{% asset_img 2.png 示意图 width="400" %}

但是，如果我告诉你，你可以用几行代码让这个屏幕变得更好呢？
来，来，来，我将向你展示如何做到这一点。

1. 在你的主函数中，添加以下内容。
{% asset_img 3.png 示意图 width="400" %}

现在你可以在material widget中创建你自己的屏幕。这可以看起来像这样：
{% asset_img 4.png 示意图 width="400" %}

但是，等等，我现在是怎么把错误信息显示在屏幕上的呢？
嗯，很简单，通过调用 

```dart
details.exception.toString()。
```

### 查看你所有的依赖性

您想知道您的 Flutter 项目中的所有依赖项吗？为此，有一个命令：
```
flutter pub deps
```
现在您会得到一个看起来像这样的树

{% asset_img 5.png 示意图 width="400" %}

谢谢你的阅读，别忘了给我点赞哟!

<!-- https://tomicriedel.medium.com/3-tips-to-hack-your-flutter-productivity-that-you-can-use-right-away-d809812d7079 -->