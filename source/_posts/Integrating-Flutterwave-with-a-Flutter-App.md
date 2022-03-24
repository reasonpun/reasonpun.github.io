---
title: 在Flutter中集成Flutterwave
date: 2022-03-24 10:07:47
categories:
- Payment
- Flutter
tags:
- Flutter
- Flutterwave
---

{% asset_img 0.png 示意图 width="400" %}

付款始终是企业不可或缺的一部分，当向错误的个人付款或向正确的人多付钱时，它就会成为一件非常麻烦的事情。 
作为开发人员，集成支付模块是需要额外重视的关键主题之一，以确保正确完成支付，同时确保支付安全并发送给正确的接收者。 
但是，如果商家希望从不同地点的客户进行付款/接收付款，但在如何无缝地做到这一点方面的选择有限，那么您如何进行这些集成呢？

<!--more-->

使用各种支付服务提供商进行这些集成的方法有很多，但是，在本教程中，我将向您展示如何在Flutter中集成将提供各种支付选项的支付服务提供商 Flutterwave的模块。

### Flutterwave是个啥？

Flutterwave 是一家支付服务提供商，提供各种服务来发送和接收付款。 如果您是企业主，您可以使用 Flutterwave 的各种选项向客户收取款项，例如：
 * 银行转账
 * 移动货币
 * 借记卡和信用卡
 * POS
 * 银行账户
 * Visa QR
 * USSD
 * Mpesa（肯尼亚）

### Flutterwave 集成选项

**Flutterwave Inline** — 这使企业能够通过将 FlutterwaveCheckout() JavaScript 函数嵌入到代码中来使用他们的 Inline Javascript 集成流程。
**Flutterwave Standard** — 这向您展示了如何使用 Flutterwave 标准集成流程使用其 /payments 端来接受付款。
**Direct Charge API** — 本指南介绍在 Flutterwave 上执行 Direct API 卡收费。
支付链接——支付链接允许商家在他们的网站上接受支付，而无需集成。 这对于没有开发人员资源的个人和商家来说非常有用。
**HTML Checkout** — 本文档将向您展示如何在 HTML 文件中使用 Flutterwave 内联向您的客户收取付款

### 入门

#### 创建一个 Flutterwave 帐户。
导航到 Flutterwave 主页并创建一个帐户（如果您还没有帐户）。
[https://flutterwave.com/us](https://flutterwave.com/us)
#### 登录到您的 Flutterwave 帐户。
创建帐户后，登录到您的 Flutterwave 帐户，您将可以在其中看到您的仪表板。
仪表板包含您可以浏览的各种帐户详细信息项目，例如交易、转账、子帐户、计费选项、付款计划、发票等。
请参阅下图中的仪表板概述（当前使用测试模式）：

{% asset_img 1.png 示意图 width="400" %}

仪表板可帮助您在测试模式下进行集成，当准备好进行正式操作时，您可以切换到“生产模式”。

在本文中，我们将使用 Flutter + Flutterwave 插件，其实现模仿了支付集成的 Flutterwave Inline 方法。

### 步骤
#### 1. 创建一个示例 Flutter 项目
让我们创建我们的演示应用程序。 在终端上，输入 $ flutter create 命令，后跟应用程序的名称。 
在这种情况下，我们将使用名称 **flutterwave_flutter_app**。

```
$ flutter create flutterwave_flutter_app
```

#### 2.删除预生成的代码
成功创建flutter+flutter wave示例app后，在main.dart文件中，去掉默认生成的代码，替换为如下代码：
（这一步假设你对 Flutter 项目结构有一个基本的想法）。 如果您是 Flutter 新手，请查看 [Flutter 官方文档](https://flutter.dev/docs/get-started/codelab)。

```
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          primarySwatch: Colors.green,
        ),
        home: const HomePage());
  }
}

  class HomePage extends StatefulWidget{
    const HomePage({Key? key}) : super(key: key);
     
     @override
     HomePageState createState() => HomePageState();
  }

  class HomePageState extends State<HomePage> {

    @override
    Widget build(context){
      return Scaffold(
       appBar: AppBar(
              title: const Text('Flutter + Flutterwave'),
              centerTitle: true,
            ),
            body: Center(
              child:ElevatedButton(
                child: const Text('Pay with Flutterwave'),
                onPressed: () {},
              ),
            )
      );
    }
  }
```
上面的代码显示了一个简单的 UI，其标题为“Flutter +Flutterwave”，界面中心有一个按钮，用于使用 Flutterwave 发起支付。

#### 3. 在 pubspec.yml 中添加 Flutterwave 包

```
flutterwave: ^1.0.1
```

始终确保从 [Pub dev](https://pub.dev/packages/flutterwave#prerequisite) 下载包版本以获得最新包。

#### 4.在main.dart文件中导入Flutterwave包

```
import 'package:flutterwave/flutterwave.dart';
```

#### 5. 创建一个 FlutterWave 实例。

通过调用 Flutterwave.forUIPayment() 构造函数创建 Flutterwave 实例。
构造函数接受以下详细信息的**强制**实例：

**Context** , **publicKey** , **encryptionKey** , **amount** , **currency** , **email** , **fullName** , **txRef** , **isDebugMode** and **phoneNumber** 

这将返回 Flutterwave 的一个实例。
好吧，既然有一些必需的参数，让我们调整 UI 以使用户在付款时只从 UI 输入字段中填写必要的详细信息，例如电子邮件、金额、全名和电话号码。
注意：如果您希望使用不同货币的用户进行 Flutterwave 付款，您可以在 UI 中添加货币输入字段。
但是，如果您只想使用一种货币，您可以在代码中预先配置它，这样用户就不必在每次进行 Flutterwave 支付时都填写它。
有了这些新增功能，您的代码将如下所示：

```
  //add the following values to create the UI for Flutterwave Payment
class HomePageState extends State<HomePage> {
  //use the currency you would like the user to Pay In, in this case, I used KES currency
  final String currency = FlutterwaveCurrency.KES;
  final formKey = GlobalKey<FormState>();
  final TextEditingController fullname = TextEditingController();
  final TextEditingController phone = TextEditingController();
  final TextEditingController email = TextEditingController();
  final TextEditingController amount = TextEditingController();

  @override
  Widget build(context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter + Flutterwave'),
          centerTitle: true,
        ),
        body: Padding(
                padding: const EdgeInsets.only(top: 10.0),
       child: Form(
            key: formKey,
         child:Column(
          children: [
          const Padding(padding: EdgeInsets.all(10.0)),
          Container(
            margin: const EdgeInsets.only(bottom: 10),
            child: TextFormField(
              controller: fullname,
              decoration: const InputDecoration(labelText: "Full Name"),
                   validator: (value) =>
                      value!.isNotEmpty? null : "Please fill in Your Name",
            ),
          ),
          Container(
            margin: const EdgeInsets.only(bottom: 10),
            child: TextFormField(
              controller: phone,
              decoration: const InputDecoration(labelText: "Phone Number"),
               validator: (value) =>
                      value!.isNotEmpty? null : "Please fill in Your Phone number",
            ),
          ),
          Container(
            margin: const EdgeInsets.only(bottom: 10),
            child: TextFormField(
              controller: email,
              decoration: const InputDecoration(labelText: "Email"),
                validator: (value) =>
                      value!.isNotEmpty? null : "Please fill in Your Email",
            ),
          ),
          Container(
            margin: const EdgeInsets.only(bottom: 10),
            child: TextFormField(
              controller: amount,
              decoration: const InputDecoration(labelText: "Amount"),
              validator: (value) =>
                      value!.isNotEmpty? null : "Please fill in the Amount you are Paying",
                ),
          ),
          ElevatedButton(
            child: const Text('Pay with Flutterwave'),
            onPressed: () {

            },
          ),
        ]))));
  }
  ```

  上面的代码有一些输入字段，用户可以在使用 Flutterwave 付款之前填写这些字段，例如电子邮件、金额、全名和电话号码。 它还有一个简单的验证，以防止在启动支付时提交空值，因为这些值是必需的参数。
接下来，创建一个用于为 Flutterwave 实例配置以下值的方法，包括：**Context** 、**publicKey** 、**encryptionKey** 、**amount** 、**currency** 、**email** 、**fullName** 、**txRef** 、**isDebugMode** 和 **phoneNumber** 。
您可以为该方法使用您喜欢的名称。 在这个例子中，我使用 _makeFlutterwavePayment()。
可以从下图所示的 Flutterwave 帐户仪表板中检索所需的**公钥**和**加密密钥**：

{% asset_img 2.png 示意图 width="400" %}

在上图中，我使用的是我的 TestMode 公钥和加密密钥。

请参阅下面创建 Flutterwave 实例的完整代码：

```
 //Add a method to make the flutter wave payment
 //This Method includes all the values needed to create the Flutterwave Instance
void _makeFlutterwavePayment(BuildContext context, String fullname, String phone, String email, String amount) async {
    try {
      Flutterwave flutterwave = Flutterwave.forUIPayment(
          //the first 10 fields below are required/mandatory
          context: this.context,
          fullName: fullname,
          phoneNumber: phone,
          email: email,
          amount: amount,
          //Use your Public and Encription Keys from your Flutterwave account on the dashboard
          encryptionKey: "Your Encription Key",
          publicKey: "Your Public Key",
          currency: currency,
          txRef: DateTime.now().toIso8601String(),
          //Setting DebugMode below to true since will be using test mode.
          //You can set it to false when using production environment.
          isDebugMode: true,
          //configure the the type of payments that your business will accept
          acceptCardPayment: false,
          acceptUSSDPayment: false,
          acceptAccountPayment: false,
          acceptFrancophoneMobileMoney: false,
          acceptGhanaPayment: false,
          acceptMpesaPayment: true,
          acceptRwandaMoneyPayment: false,
          acceptUgandaPayment: false,
          acceptZambiaPayment: false
          );
```

注意：在这个 Demo 中，我们将使用 Mpesa 测试 Flutterwave 付款，可以通过 Mpesa 接受付款，这就是为什么在上述方法的第 26 行中将 acceptMpesaPayment 设置为 true 的原因。

但是，您可以根据您所在的国家/地区和可接受的付款类型，将您想要测试的其他付款设置为真实。
如果您也想测试 CardPayment，请随意将其设置为 true。

#### 6. 处理响应

接下来，在 Flutterwave 实例下方添加一个响应，该实例返回一个 Flutterwave 实例，然后我们在其上调用异步方法 .initializeForUiPayments()。
请参阅下面的示例代码：

```
 final response = await flutterwave.initializeForUiPayments();
    if (response == null) {
        print("Transaction Failed");
      } else {
  
        if (response.status == "Transaction successful") {
          print(response.data);
          print(response.message);

        } else {
          print(response.message);
        }
      }
    } catch (error) {
      print(error);
    }
  }
```

#### 7. 向 Flutterwave 付款。

现在您已经创建了 Flutterwave 实例和响应，请在 onPressed() 函数上调用您在上面（在步骤 6 中）创建的 _makeFlutterwavePayment 方法，以在按下按钮时执行付款。
请看下面的代码：

```
ElevatedButton(
            child: const Text('Pay with Flutterwave'),
            onPressed: () {
              final name = fullname.text;
              final userPhone = phone.text;
              final userEmail = email.text;
              final amountPaid = amount.text;

              if (formKey.currentState!.validate()) {
              _makeFlutterwavePayment(context,name,userPhone,userEmail,amountPaid);
             }
            },
        ),
```

是的，我们已经完成了代码实现。

#### 8. Flutterwave 支付确认

现在运行我们刚刚按照上述说明构建的 Flutterwave Flutter 示例应用程序，并以测试用户的身份填写 UI 付款详细信息。 
见下图：

{% asset_img 3.png 示意图 width="400" %}

然后按下使用 Flutterwave 付款按钮，您现在应该可以开始付款了。 这只会显示一个付款选项，即 Mpesa，因为这是我们设置的。 检查下面的屏幕截图。

{% asset_img 4.png 示意图 width="400" %}

这将触发事务验证，如下所示：

{% asset_img 5.png 示意图 width="400" %}

示例演示

{% asset_img 6.gif 示意图 width="400" %}

付款后，您将收到一封由使用 Flutterwave 付款的客户完成的交易报告的电子邮件。 检查下面的屏幕截图。

{% asset_img 7.png 示意图 width="400" %}

您还可以在 Flutterwave 仪表板中获取对您的 Flutterwave 帐户进行的所有交易的列表，如下所示：

{% asset_img 8.png 示意图 width="400" %}

### 总结

在本教程中，您学习了如何使用 Flutterwave Inline 方法将 Flutterwave 集成到 Flutter 应用程序中。 以及如何使用 Mpesa 付款选项以无缝方式确认您作为商家从客户那里收到付款。