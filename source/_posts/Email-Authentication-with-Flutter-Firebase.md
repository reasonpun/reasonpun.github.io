---
title: 用Flutter进行电子邮件认证 - Firebase
date: 2022-10-15 16:20:52
categories:
- Flutter
tags:
- 认证
- Firebase
---

认证是每个应用程序应该具备的基本功能之一。下面是如何使用Firebase Auth为您的Flutter应用添加认证的方法。

{% asset_img 1.png 示意图 width="400" %}

Firebase是Flutter应用程序的首选后端之一，因为它提供了许多免费的功能，以及与Flutter的良好整合。
Firebase提供的功能之一是认证。
因此，我们就可以在我们的应用程序中整合电子邮件、电话、谷歌、苹果和更多的认证。

<!--more-->

{% asset_img 2.png 示意图 width="400" %}

为了让你快速了解文章的内容，我们将使用Firebase auth包进行Firebase认证。这篇文章是在Flutter 2.10版本上运行的一个例子中创建的!

那么，让我们开始进行基本的电子邮件/密码认证吧!

{% asset_img 3.gif 示意图 width="400" %}

#### 第0步：为auth创建一个私有包。

[Flutter中的包](https://docs.flutter.dev/development/packages-and-plugins/developing-packages)是可以在项目间共享的代码库，它独立于项目，开发者将其纳入并重用，使工作变得简单而不费时。

为Firebase Auth创建一个包是一个非常好的选择，从而减少包对我们主项目的依赖。

因此，为了创建一个包，我们必须首先在我们的项目中创建一个名为packages的文件夹。然后，在这个位置，运行下面的命令。

```shell
flutter create --template=package auth_service
```

上述命令将在packages中创建一个名为auth_service的文件夹。
当你在终端运行这个命令时，你会发现某些文件被创建。

{% asset_img 4.png 示意图 width="400" %}

一旦这些文件被创建，你会在软件包中发现一个名为auth_service的文件夹。

{% asset_img 5.png 示意图 width="400" %}

#### 第一步：从[Firebase控制台](https://console.firebase.google.com/)启用认证功能并选择电子邮件/密码。

一旦你把你的项目和应用程序添加到Firebase控制台，前提步骤是在控制台的右侧面板上启用认证功能，并从中启用电子邮件/密码。

当你点击认证时，你会得到一个欢迎屏幕，在那里你可以点击开始。

{% asset_img 6.png 示意图 width="400" %}

当你点击该按钮时，你将被转到登录方式列表。从那里你可以启用电子邮件/密码。

{% asset_img 7.png 示意图 width="400" %}

#### 第2步：为注册和登录创建一个简单的用户界面。

你可以用2个TextFields创建一个简单的用户界面，一个是电子邮件地址，一个是密码，还有一个按钮用来提交。
注意：使用StatelessWidget而不是StatefulWidget是一个很好的做法，因为它的重建成本比StatefulWidget低。

下面是一个例子。

login_view.dart

```dart
import 'package:auth_example/signup/view/signup_view.dart';
import 'package:auth_example/home/view/home_view.dart';
import 'package:flutter/material.dart';

class LoginView extends StatelessWidget {
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Login'),
        centerTitle: true,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            _LoginEmail(emailController: _emailController),
            const SizedBox(height: 30.0),
            _LoginPassword(passwordController: _passwordController),
            const SizedBox(height: 30.0),
            _SubmitButton(
              email: _emailController.text,
              password: _passwordController.text,
            ),
            const SizedBox(height: 30.0),
            _CreateAccountButton(),
          ],
        ),
      ),
    );
  }
}

class _LoginEmail extends StatelessWidget {
  _LoginEmail({
    Key? key,
    required this.emailController,
  }) : super(key: key);

  final TextEditingController emailController;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        controller: emailController,
        decoration: const InputDecoration(hintText: 'Email'),
      ),
    );
  }
}

class _LoginPassword extends StatelessWidget {
  _LoginPassword({
    Key? key,
    required this.passwordController,
  }) : super(key: key);

  final TextEditingController passwordController;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        controller: passwordController,
        obscureText: true,
        decoration: const InputDecoration(
          hintText: 'Password',
        ),
      ),
    );
  }
}

class _SubmitButton extends StatelessWidget {
  _SubmitButton({
    Key? key,
    required this.email,
    required this.password,
  }) : super(key: key);

  final String email, password;

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        print('hello');
      },
      child: const Text('Login'),
    );
  }
}

class _CreateAccountButton extends StatelessWidget {
  const _CreateAccountButton({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return TextButton(
      onPressed: () {
        Navigator.of(context).push(
          MaterialPageRoute(
            builder: (context) => SignUpView(),
          ),
        );
      },
      child: const Text('Create Account'),
    );
  }
}
```

signup_view.dart

```dart
import 'package:auth_example/home/view/home_view.dart';
import 'package:flutter/material.dart';


class SignUpView extends StatelessWidget {
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Create Account'),
        centerTitle: true,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            _CreateAccountEmail(emailController: _emailController),
            const SizedBox(height: 30.0),
            _CreateAccountPassword(passwordController: _passwordController),
            const SizedBox(height: 30.0),
            _SubmitButton(
              email: _emailController.text,
              password: _passwordController.text,
            ),
          ],
        ),
      ),
    );
  }
}

class _CreateAccountEmail extends StatelessWidget {
  _CreateAccountEmail({
    Key? key,
    required this.emailController,
  }) : super(key: key);
  final TextEditingController emailController;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        controller: emailController,
        decoration: const InputDecoration(hintText: 'Email'),
      ),
    );
  }
}

class _CreateAccountPassword extends StatelessWidget {
  _CreateAccountPassword({
    Key? key,
    required this.passwordController,
  }) : super(key: key);
  final TextEditingController passwordController;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        controller: passwordController,
        obscureText: true,
        decoration: const InputDecoration(
          hintText: 'Password',
        ),
      ),
    );
  }
}

class _SubmitButton extends StatelessWidget {
  _SubmitButton({
    Key? key,
    required this.email,
    required this.password,
  }) : super(key: key);
  final String email, password;

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        print('hello');
      },
      child: const Text('Create Account'),
    );
  }
}
```

正如你所看到的，这个用户界面是非常简单的。两个屏幕都有两个文本字段和一个按钮。当点击按钮时，我们只是添加了打印语句来打印用户提供的电子邮件和密码。

#### 第3步：创建后端代码，将凭证传递给你的Firebase。

你需要做的第一件事是在你的pubspec.yaml文件（你创建的auth包）中添加2个包；[firebase_core](https://pub.flutter-io.cn/packages/firebase_core)和[firebase_auth](https://pub.flutter-io.cn/packages/firebase_auth)。一旦完成，运行flutter pub get，这样Flutter框架就会将包的内容下载到你的本地系统。

现在，你需要在main.dart文件中的main()函数中初始化Firebase应用程序。

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(const MyApp());
}
```

第1行，WidgetsFlutterBinding.sureInitialized();确保在我们的初始化没有完成之前，用户界面不会被渲染。

现在，让我们创建一个文件来包含我们的认证相关函数和Firebase调用。让我们在我们的auth包中把它命名为firebase_auth_service.dart。
下一步是创建两个函数：登录和注册。

```dart
Future<UserEntity?> signInWithEmailAndPassword({
    required String email,
    required String password,
  }) async {
    try {
      final userCredential = await _firebaseAuth.signInWithEmailAndPassword(
        email: email,
        password: password,
      );

      return _mapFirebaseUser(userCredential.user);
    } on auth.FirebaseAuthException catch (e) {
      throw _determineError(e);
    }
  }

  @override
  Future<UserEntity?> createUserWithEmailAndPassword({
    required String email,
    required String password,
  }) async {
    try {
      final userCredential = await _firebaseAuth.createUserWithEmailAndPassword(
        email: email,
        password: password,
      );

      return _mapFirebaseUser(_firebaseAuth.currentUser);
    } on auth.FirebaseAuthException catch (e) {
      throw _determineError(e);
    }
  }
```

所以，由于我们使用Firebase进行认证，我们有预先建立的方法来登录和注册。
对于注册，我们可以使用createUserWithEmailAndPassword，它需要两个参数，即电子邮件和密码。
同样地，我们可以使用signInWithEmailAndPassword，它也需要两个参数，即电子邮件和密码。
在这里，我们创建了两个用户定义的方法来处理我们的Firebase调用。这些方法将电子邮件和密码作为参数，并将它们传递给Firebase函数。

在这里，如果你注意到，我们正在返回UserEntity。我们刚刚创建了一个简单的模型类，它看起来像这样:

```dart
import 'package:equatable/equatable.dart';

class UserEntity extends Equatable {
  const UserEntity({
    required this.id,
    required this.firstName,
    required this.lastName,
    required this.email,
    required this.imageUrl,
  });

  final String id;
  final String firstName;
  final String lastName;
  final String email;
  final String imageUrl;

  factory UserEntity.fromJson(Map<String, dynamic> json) => UserEntity(
        id: json['id'] ?? "",
        firstName: json['firstName'] ?? "",
        lastName: json['lastName'] ?? "",
        email: json['email'] ?? "",
        imageUrl: json['imageUrl'] ?? "",
      );

  Map<String, dynamic> toJson() => <String, dynamic>{
        'id': id,
        'firstName': firstName,
        'lastName': lastName,
        'email': email,
        'imageUrl': imageUrl,
      };
  factory UserEntity.empty() => const UserEntity(
        id: "",
        firstName: "",
        lastName: "",
        email: "",
        imageUrl: "",
      );
  @override
  List<Object?> get props => [id, firstName, lastName, email, imageUrl];
}
```

我们还创建了一个名为_determineError的方法，它可以确定哪个错误被抛出。
处理异常总是一个很好的做法，这样我们的代码才不会被破坏 下面是它的代码。

```dart
AuthError _determineError(auth.FirebaseAuthException exception) {
    switch (exception.code) {
      case 'invalid-email':
        return AuthError.invalidEmail;
      case 'user-disabled':
        return AuthError.userDisabled;
      case 'user-not-found':
        return AuthError.userNotFound;
      case 'wrong-password':
        return AuthError.wrongPassword;
      case 'email-already-in-use':
      case 'account-exists-with-different-credential':
        return AuthError.emailAlreadyInUse;
      case 'invalid-credential':
        return AuthError.invalidCredential;
      case 'operation-not-allowed':
        return AuthError.operationNotAllowed;
      case 'weak-password':
        return AuthError.weakPassword;
      case 'ERROR_MISSING_GOOGLE_AUTH_TOKEN':
      default:
        return AuthError.error;
    }
  }
}
```

这里，AuthError只是一个枚举。

```dart
enum AuthError {
  invalidEmail,
  userDisabled,
  userNotFound,
  wrongPassword,
  emailAlreadyInUse,
  invalidCredential,
  operationNotAllowed,
  weakPassword,
  error,
}
```

现在，让我们从我们的用户界面中调用这些方法吧!

#### 第4步：从用户界面调用函数!

下一步是在我们的用户界面中调用这些函数。
现在，要在我们的应用程序中使用我们创建的auth包，你可以把它添加到你的应用程序的pubspec.yaml中，然后在需要的地方导入它。

```yaml
auth:
    path: packages/auth
```

现在，对于我们的用户界面，目前我们只是在用户点击登录和注册的按钮时添加了打印语句。现在，是时候采取一些行动了!

```dart
onPressed: () async {
                try {
                  await _authService.createUserWithEmailAndPassword(
                    email: _emailController.text,
                    password: _passwordController.text,
                  );
                  Navigator.of(context).pushReplacement(
                      MaterialPageRoute(builder: (context) => Home()));
                } catch (e) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(
                      content: Text(e.toString()),
                    ),
                  );
                }
              },
```

在这里，上面的代码是用于创建账户页面。在这里，我们调用了在auth_service.dart中创建的createUserWithEmailAndPassword，如果我们没有得到任何异常，我们就会导航到主屏幕，如果有任何错误就会显示SnackBar。
同样的方法，我们也可以对登录进行操作。
因此，你的最终代码将如下所示。

signup_view.dart

```dart
import 'package:auth_example/home/view/home_view.dart';
import 'package:auth_service/auth.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter/material.dart';

class SignUpView extends StatelessWidget {
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Create Account'),
        centerTitle: true,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            _CreateAccountEmail(emailController: _emailController),
            const SizedBox(height: 30.0),
            _CreateAccountPassword(passwordController: _passwordController),
            const SizedBox(height: 30.0),
            _SubmitButton(
              email: _emailController.text,
              password: _passwordController.text,
            ),
          ],
        ),
      ),
    );
  }
}

class _CreateAccountEmail extends StatelessWidget {
  _CreateAccountEmail({
    Key? key,
    required this.emailController,
  }) : super(key: key);
  final TextEditingController emailController;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        controller: emailController,
        decoration: const InputDecoration(hintText: 'Email'),
      ),
    );
  }
}

class _CreateAccountPassword extends StatelessWidget {
  _CreateAccountPassword({
    Key? key,
    required this.passwordController,
  }) : super(key: key);
  final TextEditingController passwordController;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        controller: passwordController,
        obscureText: true,
        decoration: const InputDecoration(
          hintText: 'Password',
        ),
      ),
    );
  }
}

class _SubmitButton extends StatelessWidget {
  _SubmitButton({
    Key? key,
    required this.email,
    required this.password,
  }) : super(key: key);
  final String email, password;
  final AuthService _authService = FirebaseAuthService(
    authService: FirebaseAuth.instance,
  );
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        try {
          await _authService.createUserWithEmailAndPassword(
            email: email,
            password: password,
          );
          Navigator.of(context).pushReplacement(
            MaterialPageRoute(
              builder: (context) => Home(),
            ),
          );
        } catch (e) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text(e.toString()),
            ),
          );
        }
      },
      child: const Text('Create Account'),
    );
  }
}
```

{% asset_img 8.gif 示意图 width="400" %}

login_view.dart

```dart
import 'package:auth_example/signup/view/signup_view.dart';
import 'package:auth_example/home/view/home_view.dart';
import 'package:auth_service/auth.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter/material.dart';

class LoginView extends StatelessWidget {
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Login'),
        centerTitle: true,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            _LoginEmail(emailController: _emailController),
            const SizedBox(height: 30.0),
            _LoginPassword(passwordController: _passwordController),
            const SizedBox(height: 30.0),
            _SubmitButton(
              email: _emailController.text,
              password: _passwordController.text,
            ),
            const SizedBox(height: 30.0),
            _CreateAccountButton(),
          ],
        ),
      ),
    );
  }
}

class _LoginEmail extends StatelessWidget {
  _LoginEmail({
    Key? key,
    required this.emailController,
  }) : super(key: key);

  final TextEditingController emailController;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        controller: emailController,
        decoration: const InputDecoration(hintText: 'Email'),
      ),
    );
  }
}

class _LoginPassword extends StatelessWidget {
  _LoginPassword({
    Key? key,
    required this.passwordController,
  }) : super(key: key);

  final TextEditingController passwordController;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        controller: passwordController,
        obscureText: true,
        decoration: const InputDecoration(
          hintText: 'Password',
        ),
      ),
    );
  }
}

class _SubmitButton extends StatelessWidget {
  _SubmitButton({
    Key? key,
    required this.email,
    required this.password,
  }) : super(key: key);

  final String email, password;
  final AuthService _authService = FirebaseAuthService(
    authService: FirebaseAuth.instance,
  );
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        try {
          await _authService.signInWithEmailAndPassword(
            email: email,
            password: password,
          );
          Navigator.of(context)
              .pushReplacement(MaterialPageRoute(builder: (context) => Home()));
        } catch (e) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text(e.toString()),
            ),
          );
        }
      },
      child: const Text('Login'),
    );
  }
}

class _CreateAccountButton extends StatelessWidget {
  const _CreateAccountButton({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return TextButton(
      onPressed: () {
        Navigator.of(context).push(
          MaterialPageRoute(
            builder: (context) => SignUpView(),
          ),
        );
      },
      child: const Text('Create Account'),
    );
  }
}
```

{% asset_img 9.gif 示意图 width="400" %}

所以，现在这样，我们可以在任何我们想使用的项目中使用auth_service包。
如果在未来的任何时候，我们想把后端从Firebase改为其他服务，我们只需要更新这个包，我们的任务就完成了 这使我们能够使我们的应用程序更具可扩展性!

所以，总结一下步骤:

 * 在你的Firebase控制台中添加项目和应用程序。
 * 从控制台启用认证和电子邮件/密码认证。
 * 为认证服务创建一个包。
 * 创建简单的用户界面来获取用户的电子邮件和密码。
 * 从用户界面中调用这些函数
等等，你喜欢这篇文章吗？

这是一个适合初学者的解决方案，您可以轻松地将Flutter - Firebase纳入并实现有效的电子邮件认证。
然而，为了创建一个更具可扩展性和强大的产品，有必要使用一个状态管理解决方案。

如果你有兴趣，我们将分享关于如何建立一个高质量产品的见解。 
所以，请继续关注我们的下一篇文章😉!
<!-- https://somniosoftware.com/post/email-authentication-with-firebase-flutter -->