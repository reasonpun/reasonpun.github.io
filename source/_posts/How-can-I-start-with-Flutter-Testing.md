---
title: How can I start with Flutter Testing?
date: 2022-03-27 16:03:26
categories:
- 单元测试
- Flutter
tags:
- Flutter
- CI/CD
---

{% asset_img 1.jpeg 示意图 width="400" %}

大家好! 👋
这些天我开始了我的Flutter测试之旅。这个世界对我来说是未知的，但我知道在我们的应用程序中进行测试是超级重要的，我决定自己学习。
也许我现在给你看的代码可以做得更好，但请记住，这是我第一次做flutter测试，我想向你展示我所做的，当然，如果你有任何建议，代码将是开源的。
我对这个资源库的想法是为每个对学习测试感兴趣的人提供一个地方，创建新的内容，PR，并一起合作。

<!--more-->

所以让我们开始吧! 🙌
首先，我想解释一下我创建的是一个什么样的项目。我创建了一个显示猫的应用程序，它来自一个获取猫猫的API。
在这种情况下，我创建了一个底部导航小部件，有三个页面。在这篇文章中，我们将重点讨论第一页。

### 随机猫咪🐱

这个页面是一个随机猫咪的可视化工具，你只需要简单的点击就可以拥有看到世界上每个地方的猫咪的能力，很棒吧？
玩笑归玩笑，这个页面的想法是让用户在点击浮动的行动按钮时可以看到随机的猫咪。
为了得到这个结果，我把应用程序的结构保持得很简单，这篇文章不是关于架构的，所以我决定不花很多时间来做这个，而是努力专注于这个项目的目的：测试。🧪

{% asset_img 2.png 示意图 width="400" %}

总之，以下这就是应用程序文件夹的结构。

{% asset_img 3.png 示意图 width="400" %}

### 请求成功的情况

在这个流程中，用户需要点击浮动按钮，这会触发了一个Bloc事件。
Bloc负责与存储库通信以获得下一只猫。
存储库调用一个API，获得下一只随机猫。
在这种情况下，流程是成功的，所以服务将返回一个猫对象给存储库，它将返回给Bloc。Bloc将检查该猫是否正确（如果它有一个非空的品种列表），在这种情况下会发出

```
RandomCatStatus.success
```

视图有一个BlocConsumer，它有一个构建器，正在监听可能的状态变化，以重建视图。

{% asset_img 4.png 示意图 width="400" %}

### 品种列表为空或空的成功案例

在这种情况下，服务返回一个正确的答案，但是猫的品种列表为空或空。
这个列表对于显示猫的信息是必要的。因此，当存储库将对象返回给bloc时，它将负责验证这个列表是否为空或空，在这个例子中，这个列表是空的，所以bloc将发出RandomCatStatus.emptyBreed。视图将在BlocConsumer的监听器上监听这个状态，当它发生时，视图应该再次调用该事件以获得另一只随机猫。

{% asset_img 5.png 示意图 width="400" %}

### 当response.body为空时抛出一个错误

这是另一种可能发生的情况，当服务响应的状态代码为200，但响应体是空的，在这种情况下，服务将抛出一个错误。
当一个错误被抛出时，Bloc上的try/catch将捕获这个错误并发出RandomCatStatus.failure。视图将监听这个状态，以显示一个带有错误的消息。

{% asset_img 6.png 示意图 width="400" %}

### 当response.statusCode不是200时，抛出一个错误。
例如，如果响应状态代码返回404或400，应用程序将抛出一个错误。Bloc会捕捉到这个错误，并向视图发出一个RandomCatStatus.failure。

{% asset_img 7.png 示意图 width="400" %}

这些是该应用程序的所有可能情况。

### 让我们进入测试 🧪
现在是在flutter中进行测试的时候了。我们需要的第一件事是在pubspec.yaml中建立正确的依赖关系来进行测试。

```yaml
dependencies:
  mocktail: ^0.3.0 // to mock classes
  network_image_mock: ^2.0.1 // to mock image network
dev_dependencies:
  flutter_test:
    sdk: flutter
  bloc_test: ^9.0.3 // to do bloc testing
```

很好，现在我们可以开始了! ✅

在根项目→测试文件夹中，我已经创建了这个。

{% asset_img 8.png 示意图 width="400" %}

在flutter中，我们可以做不同类型的测试。
 * 单元测试
 * 块状测试
 * 小工具测试
 * 集成测试
在这里，我们将专注于前三种。
当你在测试时，你有不同的方法可以在做测试之后或之前使用。

```dart
// Registers a function to be run once before all tests
setUpAll(() {});

// Registers a function to be run before tests
setUp(() {});

// Registers a function to be run after tests
tearDown(() {});

// Registers a function to be run once after all tests
tearDownAll(() {});
```

### 单元测试
在这一部分，我将测试我在服务和资源库类上创建的方法。
这是 **service.dart类**

```dart
import 'dart:convert';

import 'package:catsapp/repository/model/cat.dart';
import 'package:catsapp/repository/model/result_error.dart';
import 'package:http/http.dart' as http;
import 'package:http/http.dart';

class CatService {
  CatService({
    http.Client? httpClient,
    this.baseUrl = 'https://api.thecatapi.com/v1',
  }) : _httpClient = httpClient ?? http.Client();

  final String baseUrl;
  final Client _httpClient;

  Future<Cat> search() async {
    final response = await _httpClient
        .get(Uri.parse('$baseUrl/images/search?has_breeds=true'));
    if (response.statusCode == 200) {
      if (response.body.isNotEmpty) {
        return Cat.fromJson(json.decode(response.body)[0]);
      } else {
        throw ErrorEmptyResponse();
      }
    } else {
      throw ErrorSearchingCat();
    }
  }
}
```

在这种情况下，我必须测试该服务可能出现的所有情况。要做到这一点，我需要做的第一件事就是为响应、对象等创建模拟类。

```dart
class MockHttp extends Mock implements http.Client {}
 
class MockResponse extends Mock implements http.Response {}

class FakeUri extends Fake implements Uri {}
```

然后在main方法中，我创建了一个组，拥有所有与服务类相关的测试。在**setUpAll**方法中，我注册了一个**registerFallbackValue(FakeUri())**，因为我需要它在测试中使用一个假的URI。
如果我不这样做，我就不能使用any()方法。
另外，我已经为模拟创建了所需的变量。这些都将被实例化到setUp方法中。此外，我还在一个单独的类中创建了一个JSON变量来伪造API的结果（当API返回一个正确的猫时）。

```dart
group('Service', () {
  late CatService catService;
  late MockHttp httpClient;

  setUpAll(() {
    registerFallbackValue(FakeUri());
  });

  setUp(() {
    httpClient = MockHttp();
    catService = CatService(httpClient: httpClient);
  });
});
```

我准备做的第一个测试是检查构造函数是否不需要httpClient。

```dart
group('constructor', () {
   test('does not required a httpClient', () {
      expect(CatService(), isNotNull);
   });
});
```

现在我打算创建一个小组，让所有的测试得到一只随机的猫。

```dart
group(('catSearch'), () {});
```

在这个组里面，我需要检查不同的东西。
1. 当服务做了一个正确的HTTP请求但主体是空的。

```dart
test(
          'make correct http request with empty response,'
          ' throw [ErrorEmptyResponse]', () async {
        final response = MockResponse();

        when(() => response.statusCode).thenReturn(200);
        when(() => response.body).thenReturn('');
        when(() => httpClient.get(any())).thenAnswer((_) async => response);
        try {
          await catService.search();
          fail('should throw error empty body');
        } catch (error) {
          expect(
            error,
            isA<ErrorEmptyResponse>(),
          );
        }
        verify(
          () => httpClient.get(
            Uri.parse(
                'https://api.thecatapi.com/v1/images/search?has_breeds=true'),
          ),
        ).called(1);
      });
```

2. 当服务抛出一个错误时，响应不是200。

```dart
test('throws ResultError on non-200 response', () async {
  final response = MockResponse();
  when(() => response.statusCode).thenReturn(404);
  when(() => httpClient.get(any())).thenAnswer((_) async => response);
  expect(
    catService.search(),
    throwsA(
      isA<ErrorSearchingCat>(),
    ),
  );
});
```

3. 当服务在一个有效的响应上返回一个Cat.json。

```dart
test('return Cat.json on a valid response', () async {
        final response = MockResponse();
        when(() => response.statusCode).thenReturn(200);
        when(() => response.body).thenReturn(TestHelper.searchCatJsonResponse);
        when(() => httpClient.get(any())).thenAnswer((_) async => response);

        await catService.search();
        expect(
          Cat.fromJson(jsonDecode(response.body)[0]),
          isA<Cat>(),
        );
      });
```

如果你运行所有这些测试，你将得到最好的一句话：所有的测试都通过了！。

{% asset_img 9.png 示意图 width="400" %}

这里是一个完整的类。

```dart
import 'dart:convert';

import 'package:catsapp/repository/model/cat.dart';
import 'package:catsapp/repository/model/result_error.dart';
import 'package:catsapp/repository/service.dart';
import 'package:catsapp/utils/test_helper.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:http/http.dart' as http;
import 'package:mocktail/mocktail.dart';

class MockHttp extends Mock implements http.Client {}

class MockResponse extends Mock implements http.Response {}

class FakeUri extends Fake implements Uri {}

void main() {
  group('Service', () {
    late CatService catService;
    late MockHttp httpClient;

    setUpAll(() {
      registerFallbackValue(FakeUri());
    });

    setUp(() {
      httpClient = MockHttp();
      catService = CatService(httpClient: httpClient);
    });

    group('constructor', () {
      test('does not required a httpClient', () {
        expect(CatService(), isNotNull);
      });
    });

    group(('catSearch'), () {
      test(
          'make correct http request with empty response,'
          ' throw [ErrorEmptyResponse]', () async {
        final response = MockResponse();

        when(() => response.statusCode).thenReturn(200);
        when(() => response.body).thenReturn('');
        when(() => httpClient.get(any())).thenAnswer((_) async => response);
        try {
          await catService.search();
          fail('should throw error empty body');
        } catch (error) {
          expect(
            error,
            isA<ErrorEmptyResponse>(),
          );
        }
        verify(
          () => httpClient.get(
            Uri.parse(
                'https://api.thecatapi.com/v1/images/search?has_breeds=true'),
          ),
        ).called(1);
      });

      test('throws ResultError on non-200 response', () async {
        final response = MockResponse();
        when(() => response.statusCode).thenReturn(404);
        when(() => httpClient.get(any())).thenAnswer((_) async => response);
        expect(
          catService.search(),
          throwsA(
            isA<ErrorSearchingCat>(),
          ),
        );
      });

      test('return Cat.json on a valid response', () async {
        final response = MockResponse();
        when(() => response.statusCode).thenReturn(200);
        when(() => response.body).thenReturn(TestHelper.searchCatJsonResponse);
        when(() => httpClient.get(any())).thenAnswer((_) async => response);

        await catService.search();
        expect(
          Cat.fromJson(jsonDecode(response.body)[0]),
          isA<Cat>(),
        );
      });
    });
  });
}
```
现在是时候对版本库类进行测试了。这里是cat_respository类。

```dart
import 'model/cat.dart';
import 'service.dart';

class CatRepository {
  const CatRepository({required this.service});
  final CatService service;

  Future<Cat> search() async => service.search();
}
```

首先，我创建了必要的模拟

```dart
class MockService extends Mock implements cat_service.CatService {}
```

第二，我创建了一个组，拥有与这个类相关的所有测试。这里是创建所需变量并将其初始化到setUp方法的地方。

```dart
group('Cat Repository', () {
    late cat_service.CatService catService;
    late CatRepository catRepository;
    setUp(() {
      catService = MockService();
      catRepository = CatRepository(service: catService);
    });
});
```

接下来，我为构造函数创建了一个测试，在这里我检查了cat服务是否需要。

```dart
test('instantiates CatRepository with a required carService', () {                                         
   expect(catRepository, isNotNull);                                  
});
```

完成这个类的所有覆盖的其他检查:
1. 对搜索方法的调用。

```dart
test('calls search method', () async {
  try {
    await catRepository.search();
  } catch (_) {}
  verify(() => catService.search()).called(1);
});
```

2. 当搜索方法失败时抛出错误。

```dart
test('throws Result exception when search fails', () async {
    // first create a exception mock instance
    final exception = ErrorSearchingCat();
    // when calls and api and throw an exception
    when(() => catService.search()).thenThrow(exception);
    // then expect an error result
    expect(() async => catRepository.search(), throwsA(exception));
});
```

以下就是cat_repository_test.dart类的代码。

```dart
import 'package:catsapp/repository/cat_repository.dart';
import 'package:catsapp/repository/model/result_error.dart';
import 'package:catsapp/repository/service.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

class MockService extends Mock implements CatService {}

void main() {
  group('Cat Repository', () {
    late CatService catService;
    late CatRepository catRepository;

    setUp(() {
      catService = MockService();
      catRepository = CatRepository(service: catService);
    });

    group('constructor', () {
      test('instantiates CatRepository with a required carService', () {
        expect(catRepository, isNotNull);
      });
    });

    group(('search next cat'), () {
      test('calls search method', () async {
        try {
          await catRepository.search();
        } catch (_) {}
        verify(() => catService.search()).called(1);
      });

      test('throws Result exception when search fails', () async {
        // first create a exception mock instance
        final exception = ErrorSearchingCat();
        // when calls and api and throw an exception
        when(() => catService.search()).thenThrow(exception);
        // then expect an error result
        expect(() async => catRepository.search(), throwsA(exception));
      });
    });
  });
}
```

所有测试都通过了!

{% asset_img 10.png 示意图 width="400" %}

酷! 现在我们已经把所有的测试都传到了我们的服务和资源库中，现在是时候进入bloc测试了。

### Bloc Test

#### random_cat_bloc_test.dart

首先，这是random_cat_bloc类。

```dart
import 'package:equatable/equatable.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../random_cat.dart';

part 'random_cat_state.dart';
part 'random_cat_event.dart';

class RandomCatBloc extends Bloc<RandomCatEvent, RandomCatState> {
  RandomCatBloc({required this.catRepository}) : super(const RandomCatState()) {
    on<SearchRandomCat>((event, emit) => _mapSearchEventToState(event, emit));
  }

  final CatRepository catRepository;
  void _mapSearchEventToState(
      SearchRandomCat event, Emitter<RandomCatState> emit) async {
    try {
      emit(state.copyWith(status: RandomCatStatus.loading));
      final cat = await catRepository.search();
      if (cat.breeds == null || cat.breeds!.isEmpty) {
        emit(state.copyWith(status: RandomCatStatus.emptyBreeds));
      } else {
        emit(state.copyWith(cat: cat, status: RandomCatStatus.success));
      }
    } catch (_) {
      emit(state.copyWith(status: RandomCatStatus.failure));
    }
  }
}
```

在这种类型的测试中，语法与单元测试有一点不同，但它的理念是一样的。以下是关于如何在flutter中测试Bloc的官方文档：[Bloc状态管理库](https://bloclibrary.dev/#/testing)

我按照这个文档，我得到了运行我自己的bloc测试。

让我们开始吧! 🙌
像往常一样，我们需要做的第一步是创建需要的模拟类。

```dart
class MockRepository extends Mock implements CatRepository {}

class MockCat extends Mock implements Cat {}
```

接下来，我们需要创建并初始化所需的变量给我们的模拟类。
此外，我还创建了第一个测试，以检查该集团的初始状态是否为 *RandomCatStatus.initial*。

```dart
group('RandomCatBloc', () {
  late catRepository.CatRepository catRepositoryMock;
  late cat.Cat catMock;

  setUp(() {
    catRepositoryMock = MockRepository();
    catMock = MockCat();
  });
});

test('initial state of the bloc is [RandomCatStatus.initial]', () {
  expect(RandomCatBloc(catRepository: catRepositoryMock).state,
      const RandomCatState());
});
```

现在我们可以开始进行我们的bloc测试。
当品种列表为空时，执行RandomCatStatus.loading和RandomCatStatus.emptyBreeds。

```dart
blocTest<RandomCatBloc, RandomCatState>(
  'emits [RandomCatStatus.loading, RandomCatStatus.emptyBreeds] '
  'state when cat.breeds are empty',
  setUp: () {
    when(() => catMock.breeds).thenReturn([]);
    when(() => catRepositoryMock.search()).thenAnswer((_) async => catMock);
  },
  build: () => RandomCatBloc(catRepository: catRepositoryMock),
  act: (bloc) => bloc.add(SearchRandomCat()),
  expect: () => <RandomCatState>[
    const RandomCatState(status: RandomCatStatus.loading),
    const RandomCatState(status: RandomCatStatus.emptyBreeds),
  ],
);
```
2. 当品种列表为空时，发出RandomCatStatus.loading和RandomCatStatus.emptyBreeds。

```dart
blocTest<RandomCatBloc, RandomCatState>(
  'emits [RandomCatStatus.loading, RandomCatStatus.emptyBreeds] '
  'state when cat.breeds are null',
  setUp: () {
    when(() => catMock.breeds).thenReturn(null);
    when(() => catRepositoryMock.search()).thenAnswer((_) async => catMock);
  },
  build: () => RandomCatBloc(catRepository: catRepositoryMock),
  act: (bloc) => bloc.add(SearchRandomCat()),
  expect: () => [
    const RandomCatState(status: RandomCatStatus.loading),
    const RandomCatState(status: RandomCatStatus.emptyBreeds)
  ],
);
```
3. 当响应正确且猫的数据包含品类时，发出RandomCatStatus.loading和RandomCatStatus.success。

```dart
 blocTest<RandomCatBloc, RandomCatState>(
      'emits [RandomCatStatus.loading, RandomCatStatus.success]'
      ' with a copyWith of other cat'
      'state for successful',
      setUp: () {
        when(() => catMock.breeds).thenReturn(
          List.generate(
            1,
            (index) => const Breed(id: '1'),
          ),
        );
        when(() => catRepositoryMock.search()).thenAnswer((_) async => catMock);
      },
      build: () => RandomCatBloc(catRepository: catRepositoryMock),
      act: (bloc) => bloc.add(SearchRandomCat()),
      expect: () => [
        const RandomCatState(status: RandomCatStatus.loading),
        RandomCatState(status: RandomCatStatus.success, cat: catMock),
      ],
    );
```

4. 当不成功时发出RandomCatStatus.loading和RandomCatStatus.failation。

```dart
blocTest<RandomCatBloc, RandomCatState>(
  'emits [RandomCatStatus.loading, RandomCatStatus.failure] '
  'when unsuccessful',
  setUp: () {
    when(() =>    catRepositoryMock.search()).thenThrow(ErrorSearchingCat());
  },
  build: () => RandomCatBloc(catRepository: catRepositoryMock),
  act: (bloc) => bloc.add(SearchRandomCat()),
  expect: () => <RandomCatState>[
    const RandomCatState(status: RandomCatStatus.loading),
    const RandomCatState(status: RandomCatStatus.failure)
  ],
);
```

接下来看看全部的random_cat_bloc_test类!

```dart
import 'package:bloc_test/bloc_test.dart';
import 'package:catsapp/repository/cat_repository.dart';
import 'package:catsapp/repository/model/cat.dart';
import 'package:catsapp/ui/random_cat/random_cat.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

class MockRepository extends Mock implements CatRepository {}

class MockCat extends Mock implements Cat {}

void main() {
  group('RandomCatBloc', () {
    late CatRepository catRepositoryMock;
    late Cat catMock;

    setUp(() {
      catRepositoryMock = MockRepository();
      catMock = MockCat();
    });

    test('initial state of the bloc is [RandomCatStatus.initial]', () {
      expect(RandomCatBloc(catRepository: catRepositoryMock).state,
          const RandomCatState().copyWith());
    });

    blocTest<RandomCatBloc, RandomCatState>(
      'emits [RandomCatStatus.loading, RandomCatStatus.emptyBreeds] '
      'state when cat.breeds are empty',
      setUp: () {
        when(() => catMock.breeds).thenReturn([]);
        when(() => catRepositoryMock.search()).thenAnswer((_) async => catMock);
      },
      build: () => RandomCatBloc(catRepository: catRepositoryMock),
      act: (bloc) => bloc.add(SearchRandomCat()),
      expect: () => <RandomCatState>[
        const RandomCatState(status: RandomCatStatus.loading),
        const RandomCatState(status: RandomCatStatus.emptyBreeds),
      ],
    );

    blocTest<RandomCatBloc, RandomCatState>(
      'emits [RandomCatStatus.loading, RandomCatStatus.emptyBreeds] '
      'state when cat.breeds are null',
      setUp: () {
        when(() => catMock.breeds).thenReturn(null);
        when(() => catRepositoryMock.search()).thenAnswer((_) async => catMock);
      },
      build: () => RandomCatBloc(catRepository: catRepositoryMock),
      act: (bloc) => bloc.add(SearchRandomCat()),
      expect: () => [
        const RandomCatState(status: RandomCatStatus.loading),
        const RandomCatState(status: RandomCatStatus.emptyBreeds)
      ],
    );

    blocTest<RandomCatBloc, RandomCatState>(
      'emits [RandomCatStatus.loading, RandomCatStatus.success]'
      ' with a copyWith of other cat'
      'state for successful',
      setUp: () {
        when(() => catMock.breeds).thenReturn(
          List.generate(
            1,
            (index) => const Breed(id: '1'),
          ),
        );
        when(() => catRepositoryMock.search()).thenAnswer((_) async => catMock);
      },
      build: () => RandomCatBloc(catRepository: catRepositoryMock),
      act: (bloc) => bloc.add(SearchRandomCat()),
      expect: () => [
        const RandomCatState(status: RandomCatStatus.loading),
        RandomCatState(status: RandomCatStatus.success, cat: catMock),
      ],
    );

    blocTest<RandomCatBloc, RandomCatState>(
      'emits [RandomCatStatus.loading, RandomCatStatus.failure] '
      'when unsuccessful',
      setUp: () {
        when(() => catRepositoryMock.search()).thenThrow(ErrorSearchingCat());
      },
      build: () => RandomCatBloc(catRepository: catRepositoryMock),
      act: (bloc) => bloc.add(SearchRandomCat()),
      expect: () => <RandomCatState>[
        const RandomCatState(status: RandomCatStatus.loading),
        const RandomCatState(status: RandomCatStatus.failure)
      ],
    );
  });
}
```

此外，你可以测试你的应用程序的状态和事件。
下面是random_cat_state_test.dart。

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:catsapp/ui/random_cat/random_cat.dart';

void main() {
  group('RandomCatStatusX ', () {
    test('returns correct values for RandomCatStatus.initial', () {
      const status = RandomCatStatus.initial;
      expect(status.isInitial, isTrue);
      expect(status.isLoading, isFalse);
      expect(status.isSuccess, isFalse);
      expect(status.isFailure, isFalse);
      expect(status.isEmptyBreeds, isFalse);
    });

    test('returns correct values for RandomCatStatus.loading', () {
      const status = RandomCatStatus.loading;
      expect(status.isInitial, isFalse);
      expect(status.isLoading, isTrue);
      expect(status.isSuccess, isFalse);
      expect(status.isFailure, isFalse);
      expect(status.isEmptyBreeds, isFalse);
    });

    test('returns correct values for RandomCatStatus.isSuccess', () {
      const status = RandomCatStatus.success;
      expect(status.isInitial, isFalse);
      expect(status.isLoading, isFalse);
      expect(status.isSuccess, isTrue);
      expect(status.isFailure, isFalse);
      expect(status.isEmptyBreeds, isFalse);
    });

    test('returns correct values for RandomCatStatus.isFailure', () {
      const status = RandomCatStatus.failure;
      expect(status.isInitial, isFalse);
      expect(status.isLoading, isFalse);
      expect(status.isSuccess, isFalse);
      expect(status.isFailure, isTrue);
      expect(status.isEmptyBreeds, isFalse);
    });

    test('returns correct values for RandomCatStatus.isEmptyBreeds', () {
      const status = RandomCatStatus.emptyBreeds;
      expect(status.isInitial, isFalse);
      expect(status.isLoading, isFalse);
      expect(status.isSuccess, isFalse);
      expect(status.isFailure, isFalse);
      expect(status.isEmptyBreeds, isTrue);
    });
  });
}
```

这里是random_cat_event_test.dart。

```dart
import 'package:catsapp/ui/random_cat/random_cat.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('RandomCatEvent', () {
    test('supports comparisons', () {
      expect(RandomCatEvent(), RandomCatEvent());
    });

    group('SearchRandomCat', () {
      test('supports comparisons', () {
        expect(SearchRandomCat(), SearchRandomCat());
      });
    });
  });
}
```

所有测试都通过了!

{% asset_img 11.png 示意图 width="400" %}

现在我们已经完成了单元测试和模块测试的部分，是时候开始进行Widget测试了!

### Widget测试
这种测试是超级酷的，你可以测试用户与应用程序的可能互动，所以你可以完全控制你的应用程序的所有状态。
在这种情况下，Widget测试将是很容易的。✌️
让我们开始吧!
首先，我们需要测试我们的random_cat_page。

```dart
import 'package:catsapp/ui/random_cat/pages/bloc/random_cat_bloc.dart';
import 'package:catsapp/ui/random_cat/pages/random_cat_layout.dart';
import 'package:catsapp/repository/cat_repository.dart';
import 'package:catsapp/repository/service.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class RandomCatPage extends StatelessWidget {
  const RandomCatPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return RepositoryProvider(
      create: (context) => CatRepository(service: CatService()),
      child: BlocProvider(
        create: (context) => RandomCatBloc(
          catRepository: context.read<CatRepository>(),
        )..add(SearchRandomCat()),
        child: const RandomCatLayout(),
      ),
    );
  }
}
```

同样，我们需要创建假的类。

```dart
class FakeRandomCatState extends Fake implements RandomCatState {}

class FakeRandomCatEvent extends Fake implements RandomCatEvent {}
```
然后我们需要在setUpAll方法中注册。

```dart
setUpAll(() {
  registerFallbackValue(RandomCatEvent());
  registerFallbackValue(FakeRandomCatState());
});
```

在这种情况下，我们只需要测试页面的渲染是否正常。

```dart
group('RandomCatPage ', () {
  testWidgets('renders RandomCatPage', (tester) async {
    await mockNetworkImagesFor(
      () => tester.pumpWidget(
        const MaterialApp(
          home: RandomCatPage(),
        ),
      ),
    );
    expect(find.byType(RandomCatPage), findsOneWidget);
  });
});
```

来看下完整的程序

```dart
import 'package:catsapp/ui/random_cat/random_cat.dart';
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:network_image_mock/network_image_mock.dart';

class FakeRandomCatState extends Fake implements RandomCatState {}

class FakeRandomCatEvent extends Fake implements RandomCatEvent {}

void main() {
  setUpAll(() {
    registerFallbackValue(RandomCatEvent());
    registerFallbackValue(FakeRandomCatState());
  });

  group('RandomCatPage ', () {
    testWidgets('renders RandomCatPage', (tester) async {
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          const MaterialApp(
            home: RandomCatPage(),
          ),
        ),
      );
      expect(find.byType(RandomCatPage), findsOneWidget);
    });
  });
}
```
所有测试都通过了!

{% asset_img 12.png 示意图 width="400" %}

下一个我们需要测试的类是：Random_cat_layout_test.dart。

```dart
import '../random_cat.dart';
import 'bloc/random_cat_bloc.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class RandomCatLayout extends StatelessWidget {
  const RandomCatLayout({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return BlocConsumer<RandomCatBloc, RandomCatState>(
      listener: (context, state) {
        if (state.status.isEmptyBreeds) {
          context.read<RandomCatBloc>().add(SearchRandomCat());
        }
      },
      builder: (context, state) {
        if (state.status.isFailure) {
          return const FailureRandomCatView();
        } else if (state.status.isInitial) {
          return const InitialRandomCatView();
        } else if (state.status.isLoading) {
          return const LoadingView();
        } else if (state.status.isSuccess) {
          return SuccessRandomCatView(
            cat: state.cat,
          );
        } else {
          return const SizedBox();
        }
      },
    );
  }
}
```
这里我根据视图的状态创建了不同类型的视图。要测试这些视图，最重要的部分是Key。
按状态划分的视图。

```dart
import '../../../utils/const_keys_app.dart';
import 'package:flutter/material.dart';

// Initial

class InitialRandomCatView extends StatelessWidget {
  const InitialRandomCatView();

  @override
  Widget build(BuildContext context) {
    return const Center(
      key: Key(ConstWidgetKeysApp.RandomCatInitial),
      child: Text('No information available'),
    );
  }
}

// Loading

class LoadingView extends StatelessWidget {
  const LoadingView();

  @override
  Widget build(BuildContext context) {
    return const Center(
      key: Key(ConstWidgetKeysApp.CatLoading),
      child: CircularProgressIndicator(color: Colors.green),
    );
  }
}

// Failure

class FailureRandomCatView extends StatelessWidget {
  const FailureRandomCatView();

  @override
  Widget build(BuildContext context) {
    return const Center(
      key: Key(ConstWidgetKeysApp.RandomCatFailure),
      child: Text('Something was wrong'),
    );
  }
}

// Success

class SuccessRandomCatView extends StatelessWidget {
  const SuccessRandomCatView({
    required this.cat,
  });

  final Cat cat;
  @override
  Widget build(BuildContext context) {
    return Column(
      key: const Key(ConstWidgetKeysApp.RandomCatSuccess),
      children: [
        Padding(
          padding: const EdgeInsets.only(top: 28.0),
          child: CatCard(
            key: const Key(ConstWidgetKeysApp.RandomCatCardKey),
            cat: cat,
          ),
        ),
        const Spacer(),
        Padding(
          padding: const EdgeInsets.only(right: 18.0, bottom: 8.0),
          child: Align(
            alignment: Alignment.bottomRight,
            child: FloatingActionButton(
              key: const Key(ConstWidgetKeysApp.RandomCatFabButtonKey),
              onPressed: () async {
                context.read<RandomCatBloc>().add(SearchRandomCat());
              },
              child: const Icon(Icons.refresh),
            ),
          ),
        ),
      ],
    );
  }
}
```

这个类有更多的测试要做。这里我们有不同的状态和一个按钮的交互，所以让我们开始吧。
首先，我们需要模拟和假的类。

```dart
class MockRandomCatBloc extends MockBloc<RandomCatEvent, RandomCatState>
    implements RandomCatBloc {}

class FakeRandomCatState extends Fake implements RandomCatState {}

class FakeRandomCatEvent extends Fake implements RandomCatEvent {}
```

然后我们需要创建和初始化变量。

```dart
late MockRandomCatBloc blocCat;

setUpAll(() {
  // Register the event and the state
  registerFallbackValue(FakeRandomCatEvent());
  registerFallbackValue(FakeRandomCatState());
});

setUp(() {
  blocCat = MockRandomCatBloc();
});
```

我们要测试的第一件事是视图的可能状态，为了做到这一点，我们要创建一个特定的组，然后在里面，我们可以测试所有情况。

```dart
group('RandomCatPage states ', () {});
```

1. 当状态为RandomCatStatus.failure时，渲染视图FailureRandomCatView。

```dart
testWidgets(
        'render FailureRandomCatView when state is [RandomCatStatus.failure]',
        (tester) async {
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.failure),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );
      expect(find.byKey(const Key(ConstWidgetKeysApp.RandomCatFailure)),
          findsOneWidget);
    });
```

2. 当状态为RandomCatStatus.loading时，渲染视图LoadingView。

```dart
testWidgets(
        'render LoadingView when state is [RandomCatStatus.loading]',
        (tester) async {
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.loading),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );
      expect(
          find.byKey(const Key(ConstWidgetKeysApp.CatLoading)), findsOneWidget);
    });
```

3. 当状态为RandomCatStatus.success时，渲染视图SuccessRandomCatView。

```dart
testWidgets(
        'render SuccessRandomCatView when state is [RandomCatStatus.success]',
        (tester) async {
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.success),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );
      expect(find.byKey(const Key(ConstWidgetKeysApp.RandomCatSuccess)),
          findsOneWidget);
    });
```

4. 当状态为RandomCatStatus.initial时，渲染视图InitialRandomCatView。

```dart
testWidgets(
        'render InitialRandomCatView when state is [RandomCatStatus.initial]',
        (tester) async {
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.initial),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );
      expect(find.byKey(const Key(ConstWidgetKeysApp.RandomCatInitial)),
          findsOneWidget);
    });
```
酷😎! 现在我们已经测试了视图的所有可能状态，我们可以开始下一步了。测试按钮的点击!
当按钮被点击的时候，调用事件SearchRandomCat。

```dart
 group('Click on FAB ', () {
    testWidgets('call to bloc to get next cat', (tester) async {
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.success),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );
      expect(find.byType(FloatingActionButton), findsOneWidget);

      await tester.tap(find.byType(FloatingActionButton));
      await tester.pumpAndSettle();

      verify(() => blocCat.add(SearchRandomCat())).called(1);
    });
  });
```

最后，我们需要控制猫的品种为空的时候。
我们在BlocConsumer里面监听，所以如果发生这种情况，我们需要再次调用SearchRandomCat事件，为了测试，我们需要做以下工作。

```dart
group('Call to SearchRandomCat on BlocListener ', () {
    const cat = Cat(id: '1', breeds: [], width: 1, height: 1, url: 'www');
    testWidgets('Call to event when breeds is empty', (tester) async {
      whenListen(
        blocCat,
        Stream<RandomCatState>.fromIterable(
          [
            const RandomCatState(),
            const RandomCatState(status: RandomCatStatus.emptyBreeds, cat: cat)
          ],
        ),
      );
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.emptyBreeds, cat: cat),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );

      verify(() => blocCat.add(SearchRandomCat())).called(1);
    });
  });
```

来看完整的程序

```dart
import 'package:bloc_test/bloc_test.dart';
import 'package:catsapp/ui/random_cat/random_cat.dart';
import 'package:catsapp/utils/const_keys_app.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:network_image_mock/network_image_mock.dart';

class MockRandomCatBloc extends MockBloc<RandomCatEvent, RandomCatState>
    implements RandomCatBloc {}

class FakeRandomCatState extends Fake implements RandomCatState {}

class FakeRandomCatEvent extends Fake implements RandomCatEvent {}

void main() {
  late MockRandomCatBloc blocCat;

  setUpAll(() {
    // Register the event and the state
    registerFallbackValue(FakeRandomCatEvent());
    registerFallbackValue(FakeRandomCatState());
  });

  setUp(() {
    blocCat = MockRandomCatBloc();
  });

  group('RandomCatPage states ', () {
    testWidgets(
        'render FailureRandomCatView when state is [RandomCatStatus.failure]',
        (tester) async {
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.failure),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );
      expect(find.byKey(const Key(ConstWidgetKeysApp.RandomCatFailure)),
          findsOneWidget);
    });

    testWidgets('render LoadingView when state is [RandomCatStatus.loading]',
        (tester) async {
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.loading),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );
      expect(
          find.byKey(const Key(ConstWidgetKeysApp.CatLoading)), findsOneWidget);
    });

    testWidgets(
        'render SuccessRandomCatView when state is [RandomCatStatus.success]',
        (tester) async {
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.success),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );
      expect(find.byKey(const Key(ConstWidgetKeysApp.RandomCatSuccess)),
          findsOneWidget);
    });

    testWidgets(
        'render InitialRandomCatView when state is [RandomCatStatus.initial]',
        (tester) async {
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.initial),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );
      expect(find.byKey(const Key(ConstWidgetKeysApp.RandomCatInitial)),
          findsOneWidget);
    });
  });

  group('Click on FAB ', () {
    testWidgets('call to bloc to get next cat', (tester) async {
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.success),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );
      expect(find.byType(FloatingActionButton), findsOneWidget);

      await tester.tap(find.byType(FloatingActionButton));
      await tester.pumpAndSettle();

      verify(() => blocCat.add(SearchRandomCat())).called(1);
    });
  });

  group('Call to SearchRandomCat on BlocListener ', () {
    const cat = Cat(id: '1', breeds: [], width: 1, height: 1, url: 'www');
    testWidgets('Call to event when breeds is empty', (tester) async {
      whenListen(
        blocCat,
        Stream<RandomCatState>.fromIterable(
          [
            const RandomCatState(),
            const RandomCatState(status: RandomCatStatus.emptyBreeds, cat: cat)
          ],
        ),
      );
      when(() => blocCat.state).thenReturn(
        const RandomCatState(status: RandomCatStatus.emptyBreeds, cat: cat),
      );
      await mockNetworkImagesFor(
        () => tester.pumpWidget(
          BlocProvider<RandomCatBloc>(
            lazy: false,
            create: (_) => blocCat,
            child: const MaterialApp(
              home: RandomCatLayout(),
            ),
          ),
        ),
      );

      verify(() => blocCat.add(SearchRandomCat())).called(1);
    });
  });
}
```

所有测试都通过了!

{% asset_img 13.png 示意图 width="400" %}

我们要做的最后一步是检查保险。你有不同的选择。
 * 在终端上运行flutter测试--覆盖率。
 * 安装LCOV以实现HTML的可视化
我使用了第二个选项，所以我将告诉你如何安装和使用它。(mac的安装)。
 * brew install lcov
 * flutter test — coverage
 * genhtml coverage/lcov.info -o coverage/html
最后，你将会看到我们的应用程序100%的覆盖率🧪✌️。

{% asset_img 14.png 示意图 width="400" %}

### 结论
我认为一开始学习测试可能会很困难和令人沮丧，但你应该学习它，因为如果你想建立好的应用程序，你需要了解测试的工作原理。在我第一次接触它的时候，我解决了很多bug，如果我没有做测试，这些bug就会存在。
所以......请做测试!

以上所有源代码，您可以移步：[https://github.com/reasonpun/my_100_goals/tree/main/goals_01](https://github.com/reasonpun/my_100_goals/tree/main/goals_01)

