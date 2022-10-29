---
title: 用Bloc进行电子邮件认证 - Flutter | Firebase
date: 2022-10-15 15:50:53
categories:
- Flutter
tags:
- Bloc
- 认证
- Firebase
---

将flutter_bloc整合到现有的电子邮件认证中，该认证以前是用Firebase认证做的。

{% asset_img 1.png 示意图 width="400" %}

<!--more-->

在[上一篇文章](https://pangz.fun/Email-Authentication-with-Flutter-Firebase.html)中，我们看到了如何使用Firebase Auth将电子邮件认证整合到我们的Flutter应用中。
但是，这还没有结束! 
在这篇文章中，我们将看到我们如何将flutter_bloc整合到我们现有的电子邮件认证中。

Bloc 是广泛使用的状态管理解决方案之一，因为它使我们能够控制大量的数据以及数据的传递方式。如果你对Bloc完全陌生，也可以随时查看我们关于flutter_bloc的文章。

所以在我们之前的文章中，我们集成了电子邮件认证，并使用了功能驱动架构，看起来有点像这样。

{% asset_img 2.png 示意图 width="400" %}

现在，让我们在login文件夹内创建一个bloc文件夹，并创建3个文件：login_bloc.dart、login_event.dart和login_state.dart。

{% asset_img 3.png 示意图 width="400" %}

现在是时候把我们的业务逻辑与UI层分开了！让我们根据我们的需要创建几个事件类。

让我们根据我们的需要创建一些事件类。我们将创建3个类。

```dart
class LoginButtonPressedEvent extends LoginEvent {
  const LoginButtonPressedEvent();
}
```

2. LoginEmailChangedEvent。这将被用于文本字段的onChanged方法。所以这个事件将保持电子邮件的最新值。

```dart
class LoginEmailChangedEvent extends LoginEvent {
  const LoginEmailChangedEvent({required this.email});

  final String email;

  @override
  List<Object> get props => [email];
}
```

3. LoginPasswordChangedEvent。这将以同样的方式用于密码。

```dart
class LoginPasswordChangedEvent extends LoginEvent {
  const LoginPasswordChangedEvent({required this.password});

  final String password;

  @override
  List<Object> get props => [password];
}
```

现在，让我们来创建状态类!

login_state.dart

```dart
part of 'login_bloc.dart';

enum LoginStatus {
  success,
  failure,
  loading,
}

class LoginState extends Equatable {
  const LoginState({
    this.message = '',
    this.status = LoginStatus.loading,
    this.email = '',
    this.password = '',
  });

  final String message;
  final LoginStatus status;
  final String email;
  final String password;

  LoginState copyWith({
    String? email,
    String? password,
    LoginStatus? status,
    String? message,
  }) {
    return LoginState(
      email: email ?? this.email,
      password: password ?? this.password,
      status: status ?? this.status,
      message: message ?? this.message,
    );
  }

  @override
  List<Object?> get props => [
        message,
        status,
        email,
        password,
      ];
}
```

在这里，我们创建了一个类，LoginState和enum LoginStatus。
同时，我们创建了一个copyWith方法，这基本上意味着我们将为所有的状态变异同一个状态变量。
这里的主要变化是，如果我们没有向copyWith方法提供任何值，它将使用我们在创建状态对象时提供的值。因此，状态对象将总是有一些值，我们可以在有最新值的时候更新这些值。

现在，是时候创建主块部分了!

login_bloc.dart

在bloc部分，我们需要处理我们创建的3个事件类。

1. 
```dart
on<LoginButtonPressedEvent>(_handleLoginWithEmailAndPasswordEvent)
```

```dart
Future<void> _handleLoginWithEmailAndPasswordEvent(
    LoginButtonPressedEvent event,
    Emitter<LoginState> emit,
  ) async {
    try {
      await _authService.signInWithEmailAndPassword(
        email: state.email,
        password: state.password,
      );

      emit(state.copyWith(message: 'Success', status: LoginStatus.success));
    } catch (e) {
      emit(state.copyWith(message: e.toString(), status: LoginStatus.failure));
    }
  }
```

在这里，我们只是从我们在上一篇文章中创建的auth包中调用登录方法。
如果我们获得了成功，我们就发出带有新的成功状态和消息的状态。
另一方面，如果我们得到任何异常或失败，我们将以失败的状态和异常的信息发出状态。

2. 
```dart
on<LoginEmailChangedEvent>(_handleLoginEmailChangedEvent)
```

```dart
Future<void> _handleLoginEmailChangedEvent(
    LoginEmailChangedEvent event,
    Emitter<LoginState> emit,
  ) async {
    emit(state.copyWith(email: event.email));
  }
```
在这个事件中，我们只是用我们从用户那里得到的新邮件来更新状态。这样，每当我们访问时，我们将得到最新版本的电子邮件。

3.
```dart
on<LoginPasswordChangedEvent>(_handleLoginPasswordChangedEvent)
```

```dart
Future<void> _handleLoginPasswordChangedEvent(
    LoginPasswordChangedEvent event,
    Emitter<LoginState> emit,
  ) async {
    emit(state.copyWith(password: event.password));
  }
```

就像我们为电子邮件所做的那样，我们也要为密码发送状态。

是时候为注册功能创建Bloc文件了!

在signup文件夹中创建一个bloc文件夹，并在其中创建事件、状态和bloc文件。

{% asset_img 4.png 示意图 width="400" %}

我们将采用与登录部分相同的方法。

所以，让我向你快速展示一下注册区的3个文件应该是什么样子的。

```dart
part of 'signup_bloc.dart';

@immutable
abstract class SignupEvent extends Equatable {
  const SignupEvent();

  @override
  List<Object?> get props => [];
}

class SignupButtonPressedEvent extends SignupEvent {
  const SignupButtonPressedEvent();
}

class SignupEmailChangedEvent extends SignupEvent {
  SignupEmailChangedEvent({required this.email});

  final String email;

  @override
  List<Object> get props => [email];
}

class SignupPasswordChangedEvent extends SignupEvent {
  SignupPasswordChangedEvent({required this.password});

  final String password;

  @override
  List<Object> get props => [password];
}
```
signup_state.dart

```dart
part of 'signup_bloc.dart';

enum SignupStatus {
  success,
  failure,
  loading,
}

class SignupState extends Equatable {
  SignupState({
    this.email = '',
    this.password = '',
    this.message = '',
    this.status = SignupStatus.loading,
  });

  final String message;
  final SignupStatus status;
  final String email;
  final String password;

  SignupState copyWith({
    String? email,
    String? password,
    SignupStatus? status,
    String? message,
  }) {
    return SignupState(
      email: email ?? this.email,
      password: password ?? this.password,
      status: status ?? this.status,
      message: message ?? this.message,
    );
  }

  @override
  List<Object?> get props => [
        message,
        status,
        email,
        password,
      ];
}
```

signup_bloc.dart

```dart
import 'package:auth_service/auth.dart';
import 'package:bloc/bloc.dart';
import 'package:meta/meta.dart';
import 'package:equatable/equatable.dart';

part 'signup_event.dart';

part 'signup_state.dart';

class SignupBloc extends Bloc<SignupEvent, SignupState> {
  SignupBloc({
    required AuthService authService,
  })  : _authService = authService,
        super(SignupState()) {
    on<SignupButtonPressedEvent>(_handleCreateAccountEvent);
    on<SignupEmailChangedEvent>(_handleSignupEmailChangedEvent);
    on<SignupPasswordChangedEvent>(_handleSignupPasswordChangedEvent);
  }

  final AuthService _authService;

  Future<void> _handleSignupEmailChangedEvent(
    SignupEmailChangedEvent event,
    Emitter<SignupState> emit,
  ) async {
    emit(state.copyWith(email: event.email));
  }

  Future<void> _handleSignupPasswordChangedEvent(
    SignupPasswordChangedEvent event,
    Emitter<SignupState> emit,
  ) async {
    emit(state.copyWith(password: event.password));
  }

  Future<void> _handleCreateAccountEvent(
    SignupButtonPressedEvent event,
    Emitter<SignupState> emit,
  ) async {
    try {
      await _authService.createUserWithEmailAndPassword(
        email: state.email,
        password: state.password,
      );
      emit(state.copyWith(status: SignupStatus.success));
    } catch (e) {
      emit(state.copyWith(message: e.toString(), status: SignupStatus.failure));
    }
  }
}
```

只是这里的变化是，我们正在调用我们在Auth包中创建的注册方法!

现在，让我们在我们的用户界面中使用这个模块吧

首先，我们需要向我们的widget树提供这个bloc。

如果你注意到，我们的模块构造器将AuthService作为参数。所以，我们必须先把我们的FirebaseAuthService提供给widget树。要做到这一点，我们可以在main.dart中提供它

```dart
@override
Widget build(BuildContext context) {
  return MaterialApp(
    title: 'Material App',
    home: RepositoryProvider(
      create: (context) => FirebaseAuthService(
        authService: FirebaseAuth.instance,
      ),
      child: LoginPage(),
    ),
  );
}
```

在这里，我们使用了RepositoryProvider，因为我们要把它作为一个参数注入到我们的Bloc类中

对于登录，让我们创建一个名为login_page.dart的新文件，它只是一个skeleton类，将为login_view.dart提供我们的LoginBloc

```dart
import 'package:auth_service/auth.dart';
import 'package:firebase_auth_bloc_example/login/bloc/login_bloc.dart';
import 'package:firebase_auth_bloc_example/login/view/login_view.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class LoginPage extends StatelessWidget {
  const LoginPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => LoginBloc(
        authService: context.read<FirebaseAuthService>(),
      ),
      child: LoginView(),
    );
  }
}
```

对于注册，类似地，我们可以创建一个名为signup_page.dart的文件

```dart
import 'package:firebase_auth_bloc_example/signup/bloc/signup_bloc.dart';
import 'package:firebase_auth_bloc_example/signup/view/signup_view.dart';
import 'package:auth_service/auth.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class SignupPage extends StatelessWidget {
  const SignupPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => SignupBloc(
        authService: context.read<FirebaseAuthService>(),
      ),
      child: SignUpView(),
    );
  }
}
```

在这里，我们使用bloc库中作为上下文扩展的read方法访问FirebaseAuthService。
现在，让我们做一些真正的事情吧 让我们在我们的用户界面部分使用这个bloc!

#### 登录
因此，我们创建了一个事件，它将保存我们最新版本的登录用电子邮件（LoginEmailChangedEvent）。我们可以在我们的TextField的onChanged方法中添加这个事件

```dart
class _LoginEmail extends StatelessWidget {
  _LoginEmail({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        onChanged: ((value) {
          context.read<LoginBloc>().add(LoginEmailChangedEvent(email: value));
        }),
        decoration: const InputDecoration(hintText: 'Email'),
      ),
    );
  }
}
```
在这里，我们从widget树上读取LoginBloc的实例，并添加LoginEmailChangedEvent与文本字段的最新值。

类似地，我们可以对密码这样做。

```dart
class _LoginPassword extends StatelessWidget {
  _LoginPassword({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        onChanged: ((value) {
          context
              .read<LoginBloc>()
              .add(LoginPasswordChangedEvent(password: value));
        }),
        obscureText: true,
        decoration: const InputDecoration(
          hintText: 'Password',
        ),
      ),
    );
  }
}
```
现在，我们可以在按钮的onPressed方法中添加LoginButtonPressedEvent。

```dart
class _SubmitButton extends StatelessWidget {
  _SubmitButton({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        context.read<LoginBloc>().add(
              LoginButtonPressedEvent(),
            );
      },
      child: const Text('Login'),
    );
  }
}
```

最后，我们需要用BlocListener来包装我们的表单，这样我们就可以监听我们的成功和失败状态。

```dart
class LoginView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocListener<LoginBloc, LoginState>(
      listener: (context, state) {
        if (state.status == LoginStatus.success) {
          Navigator.of(context).pushReplacement(Home.route());
        }
        if (state.status == LoginStatus.failure) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text(state.message),
            ),
          );
        }
      },
      child: const _LoginForm(),
    );
  }
}
```

#### 创建账户
现在，我们也可以做完全相同的事情来创建一个账户!

电子邮件：
```dart
class _CreateAccountEmail extends StatelessWidget {
  _CreateAccountEmail({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        onChanged: (value) => context
            .read<SignupBloc>()
            .add(SignupEmailChangedEvent(email: value)),
        decoration: const InputDecoration(hintText: 'Email'),
      ),
    );
  }
}
```

密码：

```dart
class _CreateAccountPassword extends StatelessWidget {
  _CreateAccountPassword({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: MediaQuery.of(context).size.width / 2,
      child: TextField(
        obscureText: true,
        decoration: const InputDecoration(
          hintText: 'Password',
        ),
        onChanged: (value) => context
            .read<SignupBloc>()
            .add(SignupPasswordChangedEvent(password: value)),
      ),
    );
  }
}
```
提交按钮：

```dart
class _SubmitButton extends StatelessWidget {
  _SubmitButton({
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () => context.read<SignupBloc>().add(
            SignupButtonPressedEvent(),
          ),
      child: const Text('Create Account'),
    );
  }
}
```

Bloc Listener:

```dart
class SignUpView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocListener<SignupBloc, SignupState>(
      listener: (context, state) {
        if (state.status == SignupStatus.success) {
          Navigator.of(context).pushReplacement(Home.route());
        }
        if (state.status == SignupStatus.failure) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text(state.message),
            ),
          );
        }
      },
      child: const _SignupForm(),
    );
  }
}
```

也就是这样啦！

我们已经成功地将bloc集成到我们的电子邮件认证中。

我希望你今天能学到一些新的东西!

<!-- https://medium.com/somnio-software-flutter-agency/email-authentication-with-bloc-flutter-firebase-ea99fd03bea6 -->