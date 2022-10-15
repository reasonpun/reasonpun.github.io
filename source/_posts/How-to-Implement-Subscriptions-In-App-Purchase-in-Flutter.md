---
title: 如何在Flutter中实现订阅的应用内购买
date: 2022-06-06 16:46:05
categories:
- Flutter
tags:
- Flutter
- In-App Purchase
- Subscriptions
---

{% asset_img 1.jpeg 示意图 width="400" %}

我们以这种或那种方式建立我们的应用程序，期望着从中赚钱。
有时我们出售广告并要求用户升级以获得更好的体验，而有时我们有只针对付费用户的优质内容。

<!--more-->

为了添加高级用户功能，我们需要在我们的应用程序中添加一个支付网关。配置支付网关是一件很麻烦的事情。
所以，集成支付网关的最简单方法是在你的应用程序中添加应用内购买。

在我们开始之前，我想给你提个醒，这篇文章会有点长。
在处理支付相关的事情之前，我们应该了解每一个细节。所以，在你开始深入了解Flutter中的应用内订阅购买之前，先给自己拿杯咖啡，边喝边慢慢往下看。

在我们进入编码部分之前，让我们多谈谈应用内购买以及它在两个平台上的不同之处。

两个平台上的应用内购买产品都有些许的区别，让我们看看它们是什么：

#### 安卓系统中的产品

 * 消耗性产品：像游戏中的货币一样的产品是可消耗的产品。一旦用户消耗了它们，用户就可以再次购买它。
 * 非消耗性产品：这些产品只能购买一次，它提供了一个永久性的好处。
 * 订阅产品：这些产品在有限的时间内给用户带来好处。你可以将Netflix、Medium、Spotify的订阅与这些产品进行比较。订阅会自动更新，直到被取消。

#### iOS中的产品

 * 可消费产品，和安卓一样。
 * 非消费性产品和安卓一样。
 * 自动更新的订阅与安卓系统的订阅相同。
 * 非续订产品：它几乎与自动续订相似，唯一的区别是它不会自动更新。

> 对于iOS和Android，你不能在订阅到期前重新购买。

根据实际情况，你必须在Google Play Console和App Store Connect上创建对应的产品。
这两个平台都提供了诸如宽限期、试用期、升级或降级订阅等功能。

在这篇文章中，我们将看到如何在Flutter应用程序中添加订阅。为了测试应用内购买，你的应用必须在安卓中有一个alpha版本。

### 第1步：创建一个产品

#### 为安卓系统创建一个产品

1. 进入Google Play控制台。
2. 选择你想创建订阅的应用程序。
3. 在货币化>产品下的菜单内选择订阅。
4. 在表格中填写所需信息，这里产品ID是最重要的字段，产品ID是用来唯一地识别每个产品。

> 不要忘记激活新创建的订阅。

#### 为iOS创建一个产品
1. 进入App Store Connect。
2. 选择你想创建订阅的应用程序。
3. 在应用内购买下的侧面菜单中，部分选择管理选项。
4. 点击加号来添加新产品，并选择你想要的订阅类型。(在本教程中，我们将选择自动续订）。
5. 输入产品名称和产品ID。这里的产品ID与android相同。
6. 选择一个订阅组或创建一个新的组。
7. 填写每一个强制性的细节。如果产品缺少任何细节，那么该产品的状态将是缺少元数据。填写每个细节，直到状态变为 "准备提交"。

> 为了简单起见，两个平台上的相同产品使用相同的产品ID。
> 用户可以在一个订阅组中激活任何一个产品，用户可以在该组中升级和降级他们的订阅。

耶! 你已经成功地为两个平台创建了一个产品。

现在让我们进入第二步。

### 第2步：设置测试账户

在发布到稳定版之前，你一定需要测试应用内购买流程。

你不希望在测试应用内购买流程时，每次购买产品都要付费。为此，你必须为Android和iOS设置一个测试者账户。

#### 为安卓设置测试账户
为安卓设置测试账户非常容易。只要遵循以下两个步骤，你就可以进行测试了。

 1. 在应用程序的许可测试者中添加测试者的电子邮件地址。
 2. 在应用程序的测试者列表中添加相同的电子邮件地址。

#### 为iOS设置测试账户

为iOS设置沙盒测试账户不像Android那样简单：

 1. 转到应用程序商店连接
 2. 转到用户和访问
 3. 在左侧，你会发现沙盒下的测试者选项。点击它。
 4. 点击加号，添加一个沙盒测试者。

添加一个沙盒测试者就像创建一个新的苹果ID。你不能使用现有的苹果ID作为沙盒测试者的电子邮件地址。你必须创建一个新的电子邮件地址用于测试。

> 提示：你可以使用临时电子邮件地址服务来创建一个新的沙盒测试员账户。

在iOS中处理应用内购买时，用这个账户登录。

在我们真正开始编码之前，我们需要了解应用内购买在安卓和iOS上是如何运作的。让我们来看看。

### 第3步：了解购买订阅的工作方式

{% asset_img 2.png 示意图 width="400" %}

两个平台的购买流程或多或少有些类似。

在应用程序开始时，你启动与谷歌/苹果计费服务器的连接。
如果计费SDK不支持特定的操作系统版本，连接初始化可能会失败。

连接初始化后，你的应用程序会订阅计费服务器的PurchaseUpdateStream。计费服务器通过PurchaseUpdateStream通知你当前用户的每次购买状态（通过应用内部或外部从Play Store/App Store获取）。

在你的应用程序成功订阅PurchaseUpdateStream后，你为用户加载一个所有可用产品的列表。

用户从列表中选择任何产品并进一步购买该产品。现在会发生以下事件：

 1. 应用程序通知计费服务器，当前用户想要购买这个产品ID的产品。
 2. 计费服务器进行购买并返回响应。
 3. 响应通过PurchaseUpdateStream到达应用程序。
 4. 你的应用程序检查购买的状态并采取相应的行动。
 5. 如果购买成功，你的应用程序将通知你的应用程序的后端，这个用户已经成功购买了一个产品，这是我从计费服务器得到的购买令牌。在最终确认前，应用程序的后端将与计费服务器核实购买情况。验证成功后，你的应用程序的后端将把当前用户标记为高级用户并响应你的应用程序。
 6. 现在你的应用程序必须完成与计费服务器的交易。交易的完成是告诉计费服务器你已经成功地将产品交付给用户的一种方式。

> 交易的完成对于安卓和iOS是不同的
> 在安卓系统中，你只需要完成成功的交易即可。
> 在iOS中，你必须完成每一笔交易，无论交易的状态如何。

交易完成是非常重要的，因为如果你在安卓系统中没有完成交易，谷歌将退还购买的金额，认为购买失败了。
在交易过程中，如果应用程序崩溃或发生网络问题，当用户再次打开应用程序时，你的应用程序将通过PurchaseUpdateStream收到所有未完成交易的通知。
因此，你可以继续购买产品的过程。
根据从应用程序的后端得到的响应，你将向用户展示一个特定的提示信息。
现在让我们进入最后一步，也是最重要的一步，在Flutter中实际整合应用内购买。

### 第4步：将应用内购买与Flutter结合起来

这是所有步骤中最重要的一步。你必须处理购买和付款的相关细节。你必须处理在购买产品时可能发生的每一个例外。

如果你已经了解了购买订阅的流程，那么这一步对你来说就会更有意义。

好哒，让我们来编写代码！

我在我的应用内购买项目中使用了flutter_inapp_purchase插件。原因是我发现这个插件有很好的文档，而且很容易理解。

为了管理我们的代码，我们将创建一个名为payment_service.dart的新文件。

```dart
class PaymentService {
  /// We want singelton object of ``PaymentService`` so create private constructor
  /// 
  /// Use PaymentService as ``PaymentService.instance``
  PaymentService._internal();

  static final PaymentService instance = PaymentService._internal();
}
```
现在，让我们创建一些变量来存储数据和流来监听所有购买的最新状态。

```dart
class PaymentService {
  /// We want singelton object of ``PaymentService`` so create private constructor
  /// 
  /// Use PaymentService as ``PaymentService.instance``
  PaymentService._internal();

  static final PaymentService instance = PaymentService._internal();

  /// To listen the status of connection between app and the billing server 
  StreamSubscription<ConnectionResult> _connectionSubscription;

  /// To listen the status of the purchase made inside or outside of the app (App Store / Play Store)
  /// 
  /// If status is not error then app will be notied by this stream
  StreamSubscription<PurchasedItem> _purchaseUpdatedSubscription;


  /// To listen the errors of the purchase
  StreamSubscription<PurchaseResult> _purchaseErrorSubscription;

  /// List of product ids you want to fetch
  final List<String> _productIds = [
    'monthly_subscription'
  ];

  /// All available products will be store in this list
  List<IAPItem> _products;

  /// All past purchases will be store in this list
  List<PurchasedItem> _pastPurchases;

  /// view of the app will subscribe to this to get notified 
  /// when premium status of the user changes
  ObserverList<Function> _proStatusChangedListeners =
      new ObserverList<Function>();

  /// view of the app will subscribe to this to get errors of the purchase
  ObserverList<Function(String)> _errorListeners =
      new ObserverList<Function(String)>();

  /// logged in user's premium status
  bool _isProUser = false;

  bool get isProUser => _isProUser;
}
```

如果视图（UI）想获得最新的更新，那么它可以通过以下方法订阅，也可以取消订阅。

```dart
/// view can subscribe to _proStatusChangedListeners using this method
  addToProStatusChangedListeners(Function callback) {
    _proStatusChangedListeners.add(callback);
  }
  /// view can cancel to _proStatusChangedListeners using this method
  removeFromProStatusChangedListeners(Function callback) {
    _proStatusChangedListeners.remove(callback);
  }
  /// view can subscribe to _errorListeners using this method
  addToErrorListeners(Function callback) {
    _errorListeners.add(callback);
  }
  /// view can cancel to _errorListeners using this method
  removeFromErrorListeners(Function callback) {
    _errorListeners.remove(callback);
  }
```
PaymentService将使用以下方法来通知所有监听器。

```dart
/// Call this method to notify all the subsctibers of _proStatusChangedListeners
  void _callProStatusChangedListeners() {
    _proStatusChangedListeners.forEach((Function callback) {
      callback();
    });
  }

  /// Call this method to notify all the subsctibers of _errorListeners
  void _callErrorListeners(String error) {
    _errorListeners.forEach((Function callback) {
      callback(error);
    });
  }
```
在PaymentService内部创建initConnection和dispose方法。不要忘记在你的应用程序启动时调用initConnection方法。

```dart
/// Call this method at the startup of you app to initialize connection 
  /// with billing server and get all the necessary data
  void initConnection() async {
    await FlutterInappPurchase.instance.initConnection;
    _connectionSubscription =
        FlutterInappPurchase.connectionUpdated.listen((connected) {});
    
    _purchaseUpdatedSubscription =
        FlutterInappPurchase.purchaseUpdated.listen(_handlePurchaseUpdate);

    _purchaseErrorSubscription =
        FlutterInappPurchase.purchaseError.listen(_handlePurchaseError);

    _getItems();
    _getPastPurchases();
  }
  /// call when user close the app
  void dispose() {
    _connectionSubscription.cancel();
    _purchaseErrorSubscription.cancel();
    _purchaseUpdatedSubscription.cancel();
    FlutterInappPurchase.instance.endConnection;
  }
```

使用以下方法处理购买错误。

```dart
void _handlePurchaseError(PurchaseResult purchaseError) {
    _callErrorListeners(purchaseError.message);
}
```

现在让我们看看如何处理购买更新。

```dart
/// Called when new updates arrives at ``purchaseUpdated`` stream
  void _handlePurchaseUpdate(PurchasedItem productItem) async {
    if (Platform.isAndroid) {
      await _handlePurchaseUpdateAndroid(productItem);
    } else {
      await _handlePurchaseUpdateIOS(productItem);
    }
  }
```

为Android和iOS创建两个独立的方法来处理购买更新。
在iOS中，每一种情况都调用finishTransaction。
在Android中，只有当状态为购买时才调用finishTransaction。

```dart
Future<void> _handlePurchaseUpdateIOS(PurchasedItem purchasedItem) async {
    switch (purchasedItem.transactionStateIOS) {
      case TransactionState.deferred:
        // Edit: This was a bug that was pointed out here : https://github.com/dooboolab/flutter_inapp_purchase/issues/234
        // FlutterInappPurchase.instance.finishTransaction(purchasedItem);
        break;
      case TransactionState.failed:
        _callErrorListeners("Transaction Failed");
        FlutterInappPurchase.instance.finishTransaction(purchasedItem);
        break;
      case TransactionState.purchased:
        await _verifyAndFinishTransaction(purchasedItem);
        break;
      case TransactionState.purchasing:
        break;
      case TransactionState.restored:
        FlutterInappPurchase.instance.finishTransaction(purchasedItem);
        break;
      default:
    }
  }
  /// three purchase state https://developer.android.com/reference/com/android/billingclient/api/Purchase.PurchaseState
  /// 0 : UNSPECIFIED_STATE
  /// 1 : PURCHASED
  /// 2 : PENDING
  Future<void> _handlePurchaseUpdateAndroid(PurchasedItem purchasedItem) async {
    switch (purchasedItem.purchaseStateAndroid) {
      case 1:
        if (!purchasedItem.isAcknowledgedAndroid) {
          await _verifyAndFinishTransaction(purchasedItem);
        }
        break;
      default:
        _callErrorListeners("Something went wrong");
    }
  }
```

现在让我们来验证一下购买的状态是否是成功的。

```dart
/// Call this method when status of purchase is success
  /// Call API of your back end to verify the reciept
  /// back end has to call billing server's API to verify the purchase token
  _verifyAndFinishTransaction(PurchasedItem purchasedItem) async {
    bool isValid = false;
    try {
      // Call API
      isValid = await _verifyPurchase(purchasedItem);
    } on NoInternetException {
      _callErrorListeners("No Internet");
      return;
    } on Exception {
      _callErrorListeners("Something went wrong");
      return;
    }

    if (isValid) {
      FlutterInappPurchase.instance.finishTransaction(purchasedItem);
      _isProUser = true;
      // save in sharedPreference here
      _callProStatusChangedListeners();
    } else {
      _callErrorListeners("Varification failed");
    }
  }
```

在_verifyPurchase中调用你后端的API来验证购买，并根据响应来处理用户界面。

为了获得所有可用的产品，在initConnection()中调用_getItems()。

```dart
Future<List<IAPItem>> get products async {
  if (_products == null) {
    await _getItems();
  }
  return _products;
}

Future<void> _getItems() async {
  List<IAPItem> items =
      await FlutterInappPurchase.instance.getSubscriptions(_productIds);
  _products = List();
  for (var item in items) {
    this._products.add(item);
  }
}
```

如果你已经注意到，在initConnection中我们正在调用_getPastPurchases()方法。
在iOS中，这个方法会返回过去所有的购买行为（只限于已完成的）。
这个方法在iOS中的另一个用途是当用户更换设备时，你想让用户恢复他/她的购买行为，那么就调用这个方法。在Android中，这个方法只返回活跃的订阅（完成的和未完成的都有）。

```dart
void _getPastPurchases() async {
    // remove this if you want to restore past purchases in iOS
    if (Platform.isIOS) {
      return;
    }
    List<PurchasedItem> purchasedItems =
        await FlutterInappPurchase.instance.getAvailablePurchases();

    for (var purchasedItem in purchasedItems) {
      bool isValid = false;

      if (Platform.isAndroid) {
        Map map = json.decode(purchasedItem.transactionReceipt);
          // if your app missed finishTransaction due to network or crash issue
          // finish transactins
        if (!map['acknowledged']) {
          isValid = await _verifyPurchase(purchasedItem);
          if (isValid) {
            FlutterInappPurchase.instance.finishTransaction(purchasedItem);
            _isProUser = true;
            _callProStatusChangedListeners();
          }
        } else {
          _isProUser = true;
          _callProStatusChangedListeners();
        }
      }
    }

    _pastPurchases = List();
    _pastPurchases.addAll(purchasedItems);
  }
```
要购买该产品，请调用以下方法。

```dart
Future<Null> buyProduct(IAPItem item) async {
  try {
    await FlutterInappPurchase.instance.requestSubscription(item.productId);
  } catch (error) {
  }
}
```
如果在iOS的交易过程中发生任何错误，那么下一次当用户打开应用程序时，相同的购买项目将传到PurchaseUpdateStream。在Android中，你将通过_pastPurchases获得未完成的交易。

现在你所要做的就是在你的应用程序启动时调用initConnection方法。从PaymentService中获取所有项目到用户界面并展示你的产品。
当用户选择一个产品并点击购买按钮时，调用所选产品的buyProduct()方法。使用_proStatusChangedListeners和_errorListeners来处理所有的情况。

这样就做完了！

要阅读更多关于应用内购买的信息，请阅读这些官方博客。

安卓：[https://developer.android.com/google/play/billing/integrate](https://developer.android.com/google/play/billing/integrate)

iOS: [https://developer.apple.com/documentation/storekit/in-app_purchase](https://developer.apple.com/documentation/storekit/in-app_purchase)

<!-- https://medium.com/bosc-tech-labs-private-limited/how-to-implement-subscriptions-in-app-purchase-in-flutter-7ce8906e608a -->