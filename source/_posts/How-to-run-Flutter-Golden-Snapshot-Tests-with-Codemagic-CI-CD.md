---
title: 如何使用 Codemagic CI/CD 运行 Flutter Golden（快照）测试
date: 2022-03-18 20:29:02
categories:
- 单元测试
- Flutter
tags:
- Flutter
- CI/CD
---

{% asset_img 1.jpeg 示意图 width="400" %}

测试是开发的一个非常重要的方面，它有助于确保您的应用程序可以正常运行并为用户提供正确的外观。 很高兴有 CI/CD 工具可以自动为您检查应用程序质量、在出现问题时通知团队以及将应用程序发布给最终用户。
在本文中，我将和您唠一唠如何使用 Codemagic CI/CD 运行 Golden 测试。

<!--more-->

### 第 1 步：创建黄金（快照）测试

使用以下代码在计数器应用程序的测试文件夹中创建 golden_widget_test.dart：

```dart
void main() {
  testWidgets(‘Golden test’, (WidgetTester tester) async {
    await tester.pumpWidget(MyApp());
    await expectLater(find.byType(MyApp),
                      matchesGoldenFile(‘goldens/main.png’));
   });
}
```

生成快照：

```
flutter test — update-goldens
```

### 第 2 步：使用 Codemagic 运行 Golden（快照）测试

当您生成黄金（快照）测试时，不同的操作系统平台会生成不同的文件。 这意味着如果您在 Linux 上生成 Golden 文件，然后在 MacOS 上运行测试，您的 Golden 测试很可能会失败。

**使用 Codemagic 运行的所有测试，都是使用MacOS系统完成的。**

因此，使用 MacOS 的人具有优势。
无需任何额外工作，所有通过的黄金测试都将在 Codemagic CI/CD 上成功完成。
即使您是 MacOS 用户，虽然，您只需要按“开始新构建”并且一切都会有条不紊的执行并最终会成功，但是，千万不要忘记在本地生成（见上文）并提交黄金文件哈。

{% asset_img 2.png 示意图 width="400" %}

如果您团队中的每个人都使用 MacOS，那么您就完成了本次的学习 🥳。 无需继续阅读啦。88~

### 第 3 步：如果您不是 MacOS 用户，请运行黄金测试

Codemagic 支持与 MacOS 基础设施的免费远程连接，并且还具有 FCI_EXPORT_DIR 变量，用于在构建完成时从构建器机器上传任何文件。
你不需要在你的机器上生成 Golden 文件。相反，你可以使用 Codemagic 来完成。 
之后，您需要创建一个仅用于生成快照的新工作流，因为，你应该不希望花费了很多构建时间，却得到以无法躲避的错误结果吧？

{% asset_img 3.png 示意图 width="400" %}

在 Test 部分，您需要启用 Flutter 测试并将 Flutter 测试参数设置为：

```
test --update-goldens
```
{% asset_img 4.png 示意图 width="400" %}

上面的命令将为您生成快照。 然后，您应该在Post-test脚本部分使用以下命令将生成的快照复制到 $FCI_EXPORT_DIR：

```
cp -r test/goldens $FCI_EXPORT_DIR
```

{% asset_img 5.png 示意图 width="400" %}

不要忘记禁用构建和发布部分！

{% asset_img 6.png 示意图 width="400" %}

保存设置并开始新的构建。 完成构建后，您应该会在左侧的 Artifacts 部分中看到生成的黄金文件。

{% asset_img 7.png 示意图 width="400" %}

您现在需要的是下载这个 zip 并将生成的黄金文件提取到您的项目中。
提示：您的项目中存在有 Golden 测试的话，如果平台不是 MacOS，则应添加跳过参数。 然后，如果你在本地运行 flutter test，测试不会因为与 Mac 的黄金差异而失败。

```dart
testWidgets('Golden test', (WidgetTester tester) async {
    await tester.pumpWidget(MyApp());
    await expectLater(
      find.byType(MyApp), matchesGoldenFile('goldens/main.png'),
    );
}, skip: !Platform.isMacOS);

```