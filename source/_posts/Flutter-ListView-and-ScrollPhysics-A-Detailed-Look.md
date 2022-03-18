---
title: Flutter ListView 和 ScrollPhysics 之详解
date: 2022-03-15 18:08:13
categories:
- Widget
- Flutter
tags:
- Flutter
- ListView
- Dive Into
---

{% asset_img 1.png 示意图 width="400" %}

本文旨在对 ListView 类、ScrollPhysics 以及通用小部件的调整和优化进行更详细的探索。
Flutter 中的 ListView 是可滚动项的线性列表。我们可以使用它来制作可滚动的项目列表或制作重复项目的列表。
<!--more-->
## 探索 ListView 的类型

我们将从查看 ListView 的类型开始，然后查看其他功能和对其进行的巧妙修改。

我们来看看 ListView 的类型有：
 * 列表显示
 * ListView.builder
 * ListView.separated
 * ListView.custom
让我们一一探索这些类型：

### ListView

这是 ListView 类的默认构造函数。 ListView 只需要一个子列表并使其可滚动。

{% asset_img 2.gif 示意图 width="400" %}

代码的一般格式是：

```
ListView(
  children: <Widget>[
    ItemOne(),
    ItemTwo(),
    ItemThree(),
  ],
),
```

通常这应该与少量子项一起使用，因为 List 也会在列表中构造不可见的元素，并且大量元素可能会导致效率低下。

### ListView.builder()

builder() 构造函数构造一个重复的项目列表。构造函数有两个主要参数：一个 itemCount 用于列表中的项目数，一个 itemBuilder 用于构造每个列表项。

{% asset_img 3.gif 示意图 width="400" %}

代码的一般格式是：

```
ListView.builder(
  itemCount: itemCount,
  itemBuilder: (context, position) {
    return listItem();
  },
),
```

列表项是惰性构建的，这意味着仅构建特定数量的列表项，当用户向前滚动时，较早的列表项将被销毁。

巧妙的技巧：由于元素是延迟加载的，并且只加载了所需数量的元素，因此我们实际上不需要 itemCount 作为强制参数，并且列表可以是无限的。

```
ListView.builder(
  itemBuilder: (context, position) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Text(position.toString(), style: TextStyle(fontSize: 22.0),),
      ),
    );
  },
),
```

{% asset_img 4.gif 示意图 width="400" %}

### ListView.separated()

在 separator() 构造函数中，我们生成一个列表，我们可以指定每个项目之间的分隔符。

{% asset_img 5.gif 示意图 width="400" %}

本质上，我们构造了两个交织的列表：一个作为主列表，一个作为分隔列表。
请注意，这里不能使用前面构造函数中讨论的无限计数，并且此构造函数强制使用 itemCount。
这种类型的代码如下：

```
ListView.separated(
      itemBuilder: (context, position) {
        return ListItem();
      },
      separatorBuilder: (context, position) {
        return SeparatorItem();
      },
      itemCount: itemCount,
),
```
这种类型的列表允许您动态定义分隔符，为不同类型的项目设置不同类型的分隔符，在需要时添加或删除分隔符等。

此实现还可用于轻松**插入其他类型**的元素（例如广告），而无需对列表项中间的主列表进行任何修改。

{% asset_img 6.png 示意图 width="400" %}

注意：分隔符列表长度需要比项目列表小1，因为在最后一个元素之后不存在分隔符。

### ListView.custom()

顾名思义，custom() 构造函数可让您构建具有自定义功能的 ListViews，以了解如何构建列表的子项。为此所需的主要参数是构建项目的 SliverChildDelegate。 
SliverChildDelegates 的类型是

 * SliverChildListDelegate
 * SliverChildBuilderDelegate

SliverChildListDelegate 接受子级的直接列表，而 SliverChildBuiderDelegate 接受 IndexedWidgetBuilder（我们使用的构建器函数）。
您可以使用或子类化这些来构建您自己的委托。

```
ListView.builder 本质上是一个带有 SliverChildBuilderDelegate 的 ListView.custom。
```

ListView 默认构造函数的行为类似于带有 SliverChildListDelegate 的 ListView.custom。
现在我们已经完成了 ListViews 的类型，让我们来看看 ScrollPhysics。

## 探索 ScrollPhysics

为了控制滚动发生的方式，我们在 ListView 构造函数中设置了物理参数。
不同类型的物理是：

**NeverScrollableScrollPhysics**
NeverScrollableScrollPhysics 使列表不可滚动。使用它来完全禁用 ListView 的滚动。

**BouncingScrollPhysics**
BouncingScrollPhysics 在列表结束时弹回列表。这种效果在iOS上比较常见。

{% asset_img 7.gif 示意图 width="400" %}

**ClampingScrollPhysics**
这是 Android 上使用的默认滚动物理。该列表在最后停止并给出指示它的效果。

{% asset_img 8.gif 示意图 width="400" %}

**FixedExtentScrollPhysics**
这与此列表中的其他内容略有不同，因为它仅适用于 FixedExtendScrollControllers 和使用它们的列表。例如，我们将采用 ListWheelScrollView 来制作一个类似轮子的列表。
FixedExtentScrollPhysics 仅滚动到项目而不是两者之间的任何偏移量。

{% asset_img 9.gif 示意图 width="400" %}

这个例子的代码非常简单：

```
FixedExtentScrollController fixedExtentScrollController =
    new FixedExtentScrollController();
ListWheelScrollView(
  controller: fixedExtentScrollController,
  physics: FixedExtentScrollPhysics(),
  children: monthsOfTheYear.map((month) {
    return Card(
        child: Row(
      children: <Widget>[
        Expanded(
            child: Padding(
          padding: const EdgeInsets.all(8.0),
          child: Text(
            month,
            style: TextStyle(fontSize: 18.0),
          ),
        )),
      ],
    ));
  }).toList(),
  itemExtent: 60.0,
),
```

## 还有一些事情要知道
### 如何使被破坏的元素在列表中保持活跃？
Flutter 提供了一个 KeepAlive() 小部件，它可以让一个本来会被销毁的项目保持活动状态。在列表中，元素默认包装在 AutomaticKeepAlive 小部件中。

{% asset_img 10.png 示意图 width="400" %}

可以通过将 addAutomaticKeepAlives 字段设置为 false 来禁用 AutomaticKeepAlives。这在元素不需要保持活动状态或 KeepAlive 的自定义实现的情况下很有用。

### 为什么我的 ListView 在列表和外部小部件之间有空间？

默认情况下，ListView 在它和外部小部件之间有填充，要删除它，请将填充设置为 EdgeInsets.all(0.0)。
