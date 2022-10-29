---
title: Flutter：黄金测试 —— 比较 Widgets 和 Snapshots
date: 2022-03-18 21:26:02
categories:
- Flutter
tags:
- Flutter
- CI/CD
---

{% asset_img 1.jpeg 示意图 width="400" %}

测试对于交付具有最佳质量的应用程序非常重要。 在这篇文章中，我想谈谈 Widget 测试中鲜为人知的特性之一——黄金测试。

<!--more-->

### 什么是黄金测试？

黄金测试基本上是小部件测试。
黄金文件是从手动验证的小部件创建的图像文件。

### 如何创建黄金测试？

作为示例，让我们为 Flutter 的“Hello World”应用程序创建一个黄金测试。

{% asset_img 2.png 示意图 width="400" %}

在 Flutter 项目测试文件夹中，我们使用以下代码创建 golden_widget_test.dart：

```dart
void main() {
  testWidgets('Golden test', (WidgetTester tester) async {
    await tester.pumpWidget(MyApp());
    await expectLater(find.byType(MyApp),
                      matchesGoldenFile('main.png'));
  });
}
```

如您所见，建立黄金测试就像创建普通的小控件测试一样

```
await tester.pumpWidget(MyApp());
```

提取出小部件后，将其与对应的图像进行比较：

```
await expectLater(find.byType(MyApp),
                  matchesGoldenFile('main.png'));
```

我们使用expectLater，因为matchesGoldenFile是一个异步匹配器。 该方法与方法 expect 完全一样，唯一的区别是，当匹配器完成匹配时返回的是 Future 。

### 如何生成黄金文件？

现在是时候生成这些手动验证的图像了。 如果您想为所有 Golden 测试（或更新旧测试）生成图像，请运行：

```
flutter test --update-goldens
```

main.png 文件应该出现在您的项目测试文件夹中。 不要忘记将其添加到版本控制中。

{% asset_img 3.png 示意图 width="400" %}

对于一个特定的测试，您可以运行：

```
flutter test --update-goldens <path_to_test_file>
```

### 如何运行测试？

您现在可以像普通单元测试一样运行测试，以验证生成的 Golden 文件是否与您的小部件匹配。

```
flutter test
```

黄金测试有助于检查您的小部件是否符合预期。 它不必是整个屏幕。 您也可以为 UI 的一部分创建黄金测试。 如果您在一段时间内更改了 UI，请不要忘记生成新图像；）

### Underhood

当您生成图像时，LocalFileComparator（默认的黄金文件比较器）用于创建/更新磁盘上的文件以匹配渲染。 当您运行测试时，LocalFileComparator 从本地文件系统加载图像文件，并对编码的 PNG 执行简单的逐字节比较。 如果存在完全匹配，则返回 true。 即使您的文件代表相同的像素，但编码不同，测试也会失败。
您可以通过实现您的来覆盖默认比较器。 为此，您的比较器需要扩展 GoldenFileComparator 类。 然后将该类的一个实例分配给goldenFileComparator。 代码片段如下所示：

```dart
void main() {
  goldenFileComparator = YourComparator();

  testWidgets('Golden test', (WidgetTester tester) async {
    await tester.pumpWidget(MyApp());
    await expectLater(find.byType(MyApp),
                      matchesGoldenFile('main.png'));
  });
}

class YourComparator extends GoldenFileComparator {
  @override
  Future<void> update(Uri golden, Uint8List imageBytes) {
    // TODO: implement update
    return null;
  }

  @override
  Future<bool> compare(Uint8List imageBytes, Uri golden) {
    // TODO: implement compare
    return null;
  }
}
```
要为所有测试设置 Golden File Comparator，您可以在 flutter_test_config.dart 中指定它。