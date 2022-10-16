---
title: 在Flutter中使用GitHub Actions
date: 2022-10-17 07:20:19
categories:
- Flutter
- GitHub
tags:
- Flutter
- GitHub
---

{% asset_img 1.jpeg 示意图 width="400" %}

自动化是我们作为开发者可以拥有的最好的东西之一。
这就是为什么当一个新的拉动请求被创建时，自动测试或构建是非常重要的，这样我们就不需要手动操作了。
今天我将向你展示如何使用GitHub Actions和Flutter来创建一个完整的持续集成流程。
让我们开始吧!

<!--more-->

阅读愉快!

#### 创建我们的基本结构
首先，我们进入我们的应用程序的根，创建一个名为.github的文件夹。
在这个文件夹中，我们创建一个文件夹workflows。
现在，我们在workflows里面创建一个叫ci.yml的文件。ci.yml现在是我们的第一个工作流程。
我们将给我们的工作流一个名字（本例中为持续集成），并指定它应该在什么情况下运行。
在我们的例子中，它将是在一个拉动请求上。

{% asset_img 2.png 示意图 width="400" %}

#### 创建一个测试工作
现在我们要创建一个作业，来运行我们的Flutter应用程序的测试。
在一个工作流程中，可以有多个作业被执行。因此，我们首先要创建一个作业，我们称之为flutter_test，名称为运行Flutter应用测试。
我们还必须定义它应该运行哪个操作系统。你可以选择windows、macOS和不同的Linux版本，所以我们将只使用windows-latest。

{% asset_img 3.png 示意图 width="400" %}

现在我们来到了有趣的部分。
我们将指定工作所要做的步骤。
首先，我们必须指定，工作必须使用 actions/checkout@v2，用 actions/setup-java@v1 进行 java 设置。 
我们还必须指定用于 flutter 的操作，我们将使用 subosito 的项目：subosito/flutter-actions 在主分支上。
有了这个动作，我们就可以使用我们需要的所有flutter命令。

{% asset_img 4.png 示意图 width="400" %}
{% asset_img 5.png 示意图 width="400" %}

因此，首先，我们只是运行 flutter pub get，然后是 flutter analyze 和 flutter test。如果分析或测试失败，整个行动就会失败。

{% asset_img 6.png 示意图 width="400" %}

如果你现在把它推送到 GitHub 上的主分支，并创建一个新的拉取请求，你应该看到它自动运行了这个测试。
之后，你也可以break你的测试，提交到该分支，看看测试会如何失败。

{% asset_img 7.png 示意图 width="400" %}

#### 来点高级的

您也可以做不同的事情。
例如，如果您想为iOS构建，您只需创建一个新的作业（在flutter_test下面），将其称为 "创建iOS构建"，并取代```flutter analyze```和```flutter test```，我们将运行```flutter clean```，然后```flutter build ios --no-codeign```。
除此以外，在 "名称 "下，你还必须添加 needs: [flutter_test]，所以只有当我们的测试成功时，我们才会构建一个ios应用程序。你也可以对安卓和其他平台做同样的事情。

#### 进一步阅读和结论
在这篇文章中，你已经了解了Flutter中GitHub Actions的基础知识。
您已经看到它是多么容易使用，以及它能为您节省多少时间。

有一些很好的Flutter软件包，如Freezed、Isar或Riverpod，它们将进一步提高你的工作效率。

谢谢你的阅读，祝你有个愉快的一天!

<!-- https://tomicriedel.medium.com/857faad16671 -->