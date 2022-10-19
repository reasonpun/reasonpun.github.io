---
title: 在Flutter使用NFC
date: 2022-10-19 08:21:06
categories:
- Flutter
tags:
- Flutter
- NFC
---

{% asset_img 1.jpeg 示意图 width="400" %}
近场通信技术有各种各样的用例，让我们看看我们如何在跨平台的Flutter应用程序中使用它。

<!--more-->

#### NFC究竟是什么？
{% asset_img 2.jpeg 示意图 width="400" %}

NFC（近场通信）实际上描述了一系列的通信技术，可以用来在两个设备之间进行短距离的通信（取决于硬件，在我的测试中最多3厘米）。
通常情况下，其中一个设备没有任何形式的电源（例如上面的贴纸，被动部分），由另一个设备无线供电（例如你的手机，主动部分）。
NFC硬件有多种类型，从塑料卡上的贴纸、小钥匙扣令牌到缝有芯片的衣服。实际的芯片也有不同的功能，比如:

  * 支持的不同协议/存储格式（后面会有更多介绍）
  * 存储大小（从几个字节到32KB不等）
  * 读/写速度
  * 内置的硬件加密（如CRYPTO 1、3DES、AES128）。
  * 读取距离（需要一个特殊的读卡器，例如恩智浦ICODE SLIX，最远可达1.5米）。
  * 支持写锁，支持读计数器，支持读锁

ISO/IEC 14443将NFC标签分为5个不同的组别，这取决于上述的特征集。
通常情况下，你会处理第二组兼容的标签，这些标签允许多次读写，大小在48字节到2KB之间（理论上规范允许达到1MB）。
根据你的使用情况，你也可能最终使用遵循ISO/IEC 15693的标签，该标签描述了范围为多米的非接触式芯片卡。

#### NDEF

NFC数据交换格式，正如其名称所示，描述了一种如何将内容保存在芯片上的方法。
所以，是的，我们终于在谈论软件了 :)
NDEF允许所有兼容设备了解存储在芯片上的数据结构和类型。所有的内容都被保存为信息，一条信息可以包含多个记录。

每条记录以一个头开始，其中包含关于该记录的元数据，例如包括有效载荷的类型和大小信息。头部之后是记录的有效内容。

> NDEF Record = [Record HEADER][Record playload]

整洁的部分是头的类型，默认情况下，你有7个类型的值:

 * 0: Empty — 一个没有类型和有效内容的记录
 * 1: Well-Known (WKT) — NFC论坛RTD中预先定义的类型之一，编码为统一资源名称（URN），如 "urn:wkt:T"，其中T代表文本。
 * 2: MIME media-type — RFC 2046中定义的类型，如text/csv
 * 3: Absolute URL — 在RFC 3986中定义
 * 4: External — 用户定义的值
 * 5: Unknown — 未知类型，要求长度为0
 * 6: Unchanged — 用于分块有效内容之间或末端（终止）的记录，长度必须为0
 * 7: Reserved — 保留给未来使用

你用它做的大多数事情要么属于1-Well-Known、2-MIME或4-External类别。
一个非常常见的4-External记录的例子是Android的 "android.com:pkg"，它被用来打开应用程序。
关于实际的结构和配置还有很多，但现在，我们将结束这个话题，最后看一下Flutter的代码。

{% asset_img 3.jpeg 示意图 width="400" %}

#### [Flutter和NFC](https://pub.flutter-io.cn/packages/nfc_manager)
你会发现pub.flutter-io.cn上有多个与NFC有点关系的包。
我选择的这个叫nfc_manager，支持Android和iOS，这个包在我们的应用程序中的日常使用被证明是可靠的。

我们将在此展示的项目是基于Flutter的计数器例子，并加入了NFC的元素。
这个想法很简单：你可以在一个标签上存储你当前的计数器值，或者从标签上读取一个值。
标签就像计数器应用的一个持久性存储层。
最酷的一点是，这种存储不在设备上，也不需要任何电源。

#### 必要的设置
对于这两个平台，在我们的例子中是安卓和iOS，你需要配置你的应用程序以便能够使用NFC功能。
上述链接软件包的pub.flutter-io.cn页面已经指出了所需的步骤。

#### 安卓
只要在你的flutter项目中的AndroidManifest.xml文件中添加权限请求，你就可以开始了。
如果你认为NFC是你的应用程序需要使用的一个要求，你也可以为NFC添加一个 "use-feature "标签。

#### iOS
首先，似乎只有当你拥有一个付费的苹果开发者账户时，你才能玩转NFC。
登录苹果开发者门户，选择

```
Certificates, Identifiers & Profiles -> Identifiers → Select your app -> Enable NFC Tag Reading -> Save.
```

一旦你完成了这些，你可能需要同步/下载更新的配置文件（对我来说，它开箱即用）。
下一步是打开XCode，导航到你的项目和下面显示的屏幕。

{% asset_img 4.png 示意图 width="400" %}

这应该已经生成了你读写NDEF标签所需的所有文件。
你可能想更新你的.plist文件中的文字，说明你为什么要使用NFC。如果你需要更多的标签类型，请查看pub.flutter-io.cn页面或[苹果的这个链接](https://developer.apple.com/documentation/corenfc/building_an_nfc_tag-reader_app)。

{% asset_img 5.jpeg 示意图 width="400" %}

#### 值得注意的是

安卓和iOS对NFC的处理方式有很大区别！
安卓用户可以在系统设置中关闭设备的NFC，而iOS不允许用户关闭NFC。安卓用户可以在系统设置中关闭设备的NFC，如果他们愿意，iOS不允许用户关闭NFC。如果有应用程序要求，它将始终处于开启状态。

更重要的是，安卓允许在后台扫描多个标签，而不向用户提供任何视觉指示。
iOS总是一个接一个地扫描标签，并总是将你的应用程序覆盖在下面所示的视图上。这意味着你必须添加某种UI元素来触发阅读，并且一旦扫描了一个标签，就总是启动/停止监听器。
这可能是一个问题，也可能不是，这在很大程度上取决于你的应用程序和所需的工作流程。

{% asset_img 6.jpeg 示意图 width="400" %}

#### 示例代码
这个例子的所有代码都在main.dart文件中，下面的片段逐一展示了更相关的部分。为了以防万一，下面是该应用程序在Android上运行的截图。

{% asset_img 7.png 示意图 width="400" %}

#### 如何检查NFC是否可用捏

```dart
import 'package:nfc_manager/nfc_manager.dart';
//...
void main() async {
  WidgetsFlutterBinding.ensureInitialized(); // Required for the line below
  isNfcAvalible = await NfcManager.instance.isAvailable();
  runApp(const MyApp());
}
//...
```

该包提供了一个方法来检查NFC是否可用，你可以在Flutter环境准备好后（在第4行完成）或在runApp被调用后随时调用。
如果你是在安卓系统上，并且这个方法返回错误，这可能意味着:

 * 该手机没有NFC硬件
 * NFC设置是关闭的
 * 你没有使用NFC的权限

如果你在iOS设备上得到错误信息，这意味着该设备根本没有NFC硬件。

#### 以NDEF的形式写和读一个标签
NDEF是所有平台上最常用的格式，也是以下代码中唯一使用的格式。如果你需要其他格式，请确保你所有的目标平台都支持该格式。

```dart
//...
Future<void> _listenForNFCEvents() async {
    //Always run this for ios but only once for android
    if (Platform.isAndroid && listenerRunning == false || Platform.isIOS) {
      //Android supports reading nfc in the background, starting it one time is all we need
      if (Platform.isAndroid) {
        _alert(
          'NFC listener running in background now, approach tag(s)',
        );
        //Update button states
        setState(() {
          listenerRunning = true;
        });
      }

      NfcManager.instance.startSession(
        onDiscovered: (NfcTag tag) async {
          bool succses = false;
          //Try to convert the raw tag data to NDEF
          final ndefTag = Ndef.from(tag);
          //If the data could be converted we will get an object
          if (ndefTag != null) {
            // If we want to write the current counter vlaue we will replace the current content on the tag
            if (writeCounterOnNextContact) {
              //Ensure the write flag is off again
              setState(() {
                writeCounterOnNextContact = false;
              });
              //Create a 1Well known tag with en as language code and 0x02 encoding for UTF8
              final ndefRecord = NdefRecord.createText(_counter.toString());
              //Create a new ndef message with a single record
              final ndefMessage = NdefMessage([ndefRecord]);
              //Write it to the tag, tag must still be "connected" to the device
              try {
                //Any existing content will be overrwirten
                await ndefTag.write(ndefMessage);
                _alert('Counter written to tag');
                succses = true;
              } catch (e) {
                _alert("Writting failed, press 'Write to tag' again");
              }
            }
            //The NDEF Message was already parsed, if any
            else if (ndefTag.cachedMessage != null) {
              var ndefMessage = ndefTag.cachedMessage!;
              //Each NDEF message can have multiple records, we will use the first one in our example
              if (ndefMessage.records.isNotEmpty &&
                  ndefMessage.records.first.typeNameFormat ==
                      NdefTypeNameFormat.nfcWellknown) {
                //If the first record exists as 1:Well-Known we consider this tag as having a value for us
                final wellKnownRecord = ndefMessage.records.first;

                ///Payload for a 1:Well Known text has the following format:
                ///[Encoding flag 0x02 is UTF8][ISO language code like en][content]
                if (wellKnownRecord.payload.first == 0x02) {
                  //Now we know the encoding is UTF8 and we can skip the first byte
                  final languageCodeAndContentBytes =
                      wellKnownRecord.payload.skip(1).toList();
                  //Note that the language code can be encoded in ASCI, if you need it be carfully with the endoding
                  final languageCodeAndContentText =
                      utf8.decode(languageCodeAndContentBytes);
                  //Cutting of the language code
                  final payload = languageCodeAndContentText.substring(2);
                  //Parsing the content to int
                  final storedCounters = int.tryParse(payload);
                  if (storedCounters != null) {
                    succses = true;
                    _alert('Counter restored from tag');
                    setState(() {
                      _counter = storedCounters;
                    });
                  }
                }
              }
            }
          }
          //Due to the way ios handles nfc we need to stop after each tag
          if (Platform.isIOS) {
            NfcManager.instance.stopSession();
          }
          if (succses == false) {
            _alert(
              'Tag was not valid',
            );
          }
        },
        // Required for iOS to define what type of tags should be noticed
        pollingOptions: {
          NfcPollingOption.iso14443,
          NfcPollingOption.iso15693,
        },
      );
    }
  }
//...example code
```
让我们从最初的平台检查（第4行至第14行）开始逐节回顾代码。
使用平台类，我们可以检查我们是否在Android上运行，并使用一个本地标志防止启动读取器两次。
如果读取器没有运行，或者我们在iOS上运行，我们使用
```NfcManager.instance.startSession```方法，通过提供```onDiscovered```回调（第16行）启动一个新的阅读器会话。

所有的读和写都发生在```onDiscovered```回调中，首先检查提供的NfcTag对象。
我们只想处理NDEF标签，请看第20行，我们可以直接调用```Ndef.from(tag)```来检查提供的标签是否可以被解析为NDEF格式。

> 当你调用此方法时，nfc_manager将自动解析标签上的NDEF信息及其记录。

#### 写入
向标签写入数据是通过创建NdefMessage对象并将NdefRecord对象的列表作为内容来完成的（见第30至32行）。
我们将使用Well-Known类型的文本，默认语言编码设置为 "en"。nfc_manager包提供了一些帮助对象来创建所需的数据。
写入数据的最后一步是在我们的NDEF标签对象上调用写入方法，并传递NdefMessage对象（第36行）。

> 请注意：要写一个标签，你首先需要读取它，或者更准确地说，它仍然必须在手机的范围内。

#### 读取
读取NDEF格式的数据发生在第44至63行。
该代码首先检查标签上是否有任何NDEF信息。如果找到了，我们要确保它有记录，并且第一条记录是基于Well-Known格式的。
现在，代码确保有效数据（一串字节）以0x2开始，这是UTF-8编码文本的标志。有效数据应该始终是这样的格式。

```
[ENCODING_FLAG][ISO_LANGUAGE_CODE][TEXT]
```

剩下要做的就是将这些字节（我们可以跳过第一个字节）解码为文本，并使用```substring(2)```切断语言代码。
最后一部分与示例应用程序有关（第65至70行），确保文本是一个有效的整数，并将计数器设置为它。

#### 摘要&链接&小知识
NFC可以被整合到Flutter应用程序中，而不需要做很大的改变，而且现在的大部分手机都支持NFC。
NFC标签的存储空间不大，但仍可用于触发功能、分享信息或将用户指向特定的资源。
最重要的是，通过NDEF，所有的兼容设备都可以读取信息，而不局限于一个设备/品牌或应用程序。

例如，在日常工作中使用NFC来触发我们应用中的各种功能，以替代扫描二维码或手动搜索。
对用户来说，最大的好处是NFC不需要摄像头，而且如果需要改变上下文，标签可以被重写。

举个简单的例子:
你想看看一个存储区的内容？把你的手机靠近NFC标签，手机就会向你显示备件的清单，不管你之前做了什么，都不需要你按下按钮（至少在安卓系统上是这样的......）。

Have FUN！
<!-- https://medium.com/flutter-community/nfc-with-flutter-f8c3515cb0e0 -->