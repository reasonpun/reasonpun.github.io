---
title: 深入了解 Flutter TextField
date: 2022-03-15 18:02:16
categories:
- Widget
tags:
- Flutter
- TextField
- Dive Into
---

{% asset_img 1.png 示意图 width="400" %}

在本文中，我们将全面介绍 Flutter TextField 小部件，并找出它的功能和可能的定制。

## 文本控件简介

TextField 小部件允许从用户那里收集信息。最基本的使用TextField 的代码很简单：
```
TextField()
```

这将创建一个基本的 TextField：
{% asset_img 2.png 示意图 width="400" %}

## 从 TextField 中检索信息

由于 TextField 没有 Android 中的 ID，因此无法按需检索文本，而必须在更改时将其存储在变量中或使用控制器。
1. 最简单的方法是使用 onChanged 方法并将当前值存储在一个简单的变量中。这是它的示例代码：

```
String value = "";
TextField(
  onChanged: (text) {
    value = text;
  },
)
```
2. 第二种方法是使用 TextEditingController。控制器附加到 TextField 并让我们也可以监听和控制 TextField 的文本。

```
TextEditingController controller = TextEditingController();
TextField(
  controller: controller,
)
```

我们通过监听时间得到控件的变化
```
controller.addListener(() {
  // Do something here
});
```
并获取或设置值
```
print(controller.text); // Print current value
controller.text = "Demo Text"; // Set new value
```
## 来自 TextField 的其他回调

TextField 小部件还提供其他回调，例如
 * onEditingCompleted
 * onSubmitted

```
onEditingComplete: () {},
onSubmitted: (value) {},
```

这些是在用户单击 iOS 上的“完成”按钮时调用的回调。

## 在 TextField 中处理焦点
TextField获取焦点，意味着使 TextField 处于活动状态，并且来自键盘的任何输入都将导致在焦点 TextField 中输入数据。
1. 使用自动对焦
要在创建小部件时自动聚焦于 TextField，请将 autofocus 字段设置为 true。

```
TextField(
  autofocus: true,
),
```

默认情况下，这会将焦点设置在 TextField 上。
{% asset_img 3.gif 示意图 width="400" %}

2. 使用自定义焦点
如果我们想改变一下需求，而不仅仅是自动对焦怎么办？
由于我们需要某种方式来引用我们接下来要关注的 TextField，因此我们将 FocusNode 附加到 TextField 并使用它来切换焦点。

```
// Initialise outside the build method
FocusNode nodeOne = FocusNode();
FocusNode nodeTwo = FocusNode();
// Do this inside the build method
TextField(
  focusNode: nodeOne,
),
TextField(
  focusNode: nodeTwo,
),
RaisedButton(
  onPressed: () {
    FocusScope.of(context).requestFocus(nodeTwo);
  },
  child: Text("Next Field"),
),
```
我们创建两个焦点节点并将它们附加到 TextFields。当按钮被按下时，我们使用 FocusScope 请求焦点到下一个 TextField。

{% asset_img 4.gif 示意图 width="400" %}

## 更改文本字段的键盘属性

Flutter 中的 TextField 还允许您自定义与键盘相关的属性。
### 1.键盘类型
TextField 允许您自定义当 TextField 成为焦点时显示的键盘类型。我们为此更改了keyboardType 属性。

```
TextField(
  keyboardType: TextInputType.number,
),
```

类型有：
1. TextInputType.text（普通完整键盘）
2. TextInputType.number（数字键盘）
3. TextInputType.emailAddress（带有“@”的普通键盘）
4. TextInputType.datetime（带有“/”和“:”的数字键盘）
5. TextInputType.numberWithOptions（带有启用有符号和十进制模式选项的数字键盘）
6. TextInputType.multiline（多行信息优化）

### 2. TextInputAction

更改 TextField 的 textInputAction 可让您更改键盘本身的操作按钮。
举个例子：

```
TextField(
  textInputAction: TextInputAction.continueAction,
),
```

这会导致“完成”按钮被“继续”按钮替换：
{% asset_img 5.png 示意图 width="400" %}

或者也可以这样

```
TextField(
  textInputAction: TextInputAction.send,
),
```

{% asset_img 6.png 示意图 width="400" %}

### 3. Autocorrect

启用或禁用特定 TextField 的自动更正。使用自动更正字段进行设置。
```
TextField(
  autocorrect: false,
),
```

这也会禁用建议。

### 4. Text Capitalization

TextField 提供了一些关于如何将用户输入中的字母大写的选项。

```
TextField(
  textCapitalization: TextCapitalization.sentences,
),
```

类型有：

1. TextCapitalization.sentences

这是我们期望的正常大小写类型，每个句子的第一个字母都大写。

{% asset_img 7.png 示意图 width="400" %}

2. TextCapitalization.characters

将句子中的所有字符大写。

{% asset_img 8.png 示意图 width="400" %}

3. TextCapitalization.words

将每个单词的第一个字母大写。
{% asset_img 9.png 示意图 width="400" %}

## 文本样式、对齐方式和光标选项

Flutter 允许对 TextField 内的文本样式和对齐方式以及 TextField 内的光标进行自定义。

### TextField 内的文本对齐方式
使用 textAlign 属性调整光标在 TextField 内的位置。
```
TextField(
  textAlign: TextAlign.center,
),
```
这会导致光标和文本从 TextField 的中间开始。
{% asset_img 10.png 示意图 width="400" %}

这具有通常的对齐属性：开始、结束、左、右、中心、对齐。

### 对 TextField 中的文本进行样式设置
我们使用 style 属性来改变 TextField 中文本的外观。使用它来更改颜色、字体大小等。这类似于 Text 小部件中的样式属性，因此我们不会花太多时间来探索它。
```
TextField(
  style: TextStyle(color: Colors.red, fontWeight: FontWeight.w300),
),
```
{% asset_img 11.png 示意图 width="400" %}

### 更改 TextField 中的光标
光标可直接从 TextField 小部件自定义。
您可以更改拐角的光标颜色、宽度和半径。例如，这里我随便制作了一个圆形红色光标。

```
TextField(
  cursorColor: Colors.red,
  cursorRadius: Radius.circular(16.0),
  cursorWidth: 16.0,
),
```

{% asset_img 12.png 示意图 width="400" %}

### 控制 TextField 中的大小和最大长度

TextFields 可以控制在其中写入的最大字符数、最大行数以及在键入文本时扩展。

### 控制最大字符数
```
TextField(
  maxLength: 4,
),
```
{% asset_img 13.png 示意图 width="400" %}

通过设置 maxLength 属性，强制执行最大长度，并且默认情况下将计数器添加到 TextField。

### 制作可扩展的 TextField

有时，我们需要一个在一行完成时扩展的 TextField。在 Flutter 中，这有点奇怪（但很容易）。为此，我们将 maxLines 设置为 null，默认为 1。设置为 null 不是我们非常习惯的事情，但它很容易做到。

{% asset_img 14.png 示意图 width="400" %}

注意：将 maxLines 设置为直接值将默认将其扩展到该行数。

```
TextField(
  maxLines: 3,
)
```
{% asset_img 15.png 示意图 width="400" %}

## 遮蔽文本

要隐藏 TextField 中的文本，请将 obscureText 设置为 true。
```
TextField(
  obscureText: true,
),
```
{% asset_img 16.png 示意图 width="400" %}

## 最后，装饰 TextField
到目前为止，我们专注于 Flutter 为输入提供的功能。现在我们将开始实际设计一个精美的 TextField，而不是对您的设计师说不。
为了装饰 TextField，我们使用带有 InputDecoration 的装饰属性。由于 InputDecoration 类非常庞大，我们将尝试快速浏览大部分重要属性。

### 使用提示和标签属性向用户提供信息

提示和标签都是字符串，可帮助用户理解要在 TextField 中输入的信息。不同之处在于，一旦用户开始输入，而标签漂浮在 TextField 上，提示就会消失。

{% asset_img 17.png 示意图 width="400" %}
{% asset_img 18.png 示意图 width="400" %}

### 您可以使用“icon”、“prefixIcon”和“suffixIcon”添加图标

您可以将图标直接添加到 TextFields。您也可以使用 prefixText 和 suffixText 代替 Text。

```
TextField(
  decoration: InputDecoration(
    icon: Icon(Icons.print)
  ),
),
```
{% asset_img 19.png 示意图 width="400" %}

```
TextField(
  decoration: InputDecoration(
    prefixIcon: Icon(Icons.print)
  ),
),
```
{% asset_img 20.png 示意图 width="400" %}

### 同样对于任何其他小部件，使用“prefix”而不是“prefixIcon”
要使用通用小部件而不是图标，请使用前缀字段。同样，没有明显的原因，让我们在 TextField 中添加一个循环进度指示器。
```
TextField(
  decoration: InputDecoration(
    prefix: CircularProgressIndicator(),
  ),
),
```
{% asset_img 21.png 示意图 width="400" %}

### 提示、标签等每个属性都有其各自的样式字段

要设置提示样式，请使用hintStyle。要设置标签样式，请使用 labelStyle。

```
TextField(
  decoration: InputDecoration(
    hintText: "Demo Text",
    hintStyle: TextStyle(fontWeight: FontWeight.w300, color: Colors.red)
  ),
),
```
{% asset_img 22.png 示意图 width="400" %}

**如果您不想要标签但想要为用户提供持久消息，请使用“helperText”。**
```
TextField(
  decoration: InputDecoration(
    helperText: "Hello"
  ),
),
```
{% asset_img 23.png 示意图 width="400" %}
使用“decoration: null”或 InputDecoration.collapsed 删除 TextField 上的默认下划线
使用这些来删除 TextField 上的默认下划线。

```
TextField(
  decoration: InputDecoration.collapsed(hintText: "")
),
```
{% asset_img 24.png 示意图 width="400" %}

### 使用“border”为 TextField 设置边框

```
TextField(
  decoration: InputDecoration(
    border: OutlineInputBorder()
  )
),
```
{% asset_img 25.png 示意图 width="400" %}

您可以进一步进行大量装饰，但我们不能在一篇文章中介绍所有内容。
但我希望这可以清楚地让你了解自定义 Flutter TextFields 是多么容易，能达到这个目的就可以啦。