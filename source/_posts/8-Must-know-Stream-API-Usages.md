---
title: 8个必须知道的Stream API使用方法
date: 2022-12-16 10:46:01
categories:
- Java
tags:
- Java
- StreamAPI
---

{% asset_img 1.webp 示意图 width="400" %}

作为一名java开发工程师，数组和集合在我的代码中随处可见。我经常使用集合或数组来处理数据，如过滤和对象转换。我相信大家对数组的数据处理已经非常熟悉了。

<!--more-->

但是当数据处理比较复杂的时候，我们的代码就会非常长，难以理解。这个时候，我通常会把代码拆成方法，并给出有意义的方法名称，这样可以解决一部分问题。然而，在java8之后，官方有了一个更有效、更方便的解决方案：Stream API。

Stream是Java8中处理集合的关键抽象。它可以指定你要对集合进行的操作，并且可以进行非常复杂的操作，如搜索、过滤和映射数据。
使用Stream API对集合数据进行操作，类似于使用SQL进行数据库查询。也可以使用Stream API来执行并行操作。简而言之，Stream API提供了一种高效且易于使用的数据处理方式。

Stream具有以下特点:

 * 它不是一个数据结构，不保存数据。
 * 懒惰评估，在流的中间处理过程中，只记录操作，不立即执行，直到执行终止操作才会进行实际计算。

今天我将分享一些工作中常用的、非常有用的API。

#### 列表到地图
在我们的工作中，经常会遇到列表或数组被转换为地图的场景。Collectors.toMap可以将List转换成Map。
代码如下:

```java
public class ListToMap {

    public static void main(String[] args) {

        List<User> users = new ArrayList<>();
        users.add(new User(1, "Bob", 10));
        users.add(new User(2, "David", 22));
        users.add(new User(3, "Lance", 28));

        /**
         *  list to map
         *  (k1, k2) ->k1 indicates that if there is a duplicate key, the first key is retained and the second key is discarded
         */
        Map<String, User> userMap = users.stream().collect(Collectors.toMap(User::getUsername, user -> user, (k1, k2) -> k1));
        userMap.values().forEach(user->System.out.println(user.getUsername()));
    }
}
```

同样，还有Collectors.toList(),Collectors.toSet()，这意味着将相应的流转换为一个List或Set。

#### 使用groupingBy进行分组
分组在执行聚合数据操作时经常使用，比如按国籍对用户进行分组。说到分组，相信大家都会想到SQL的group by。在Java8之前，我们可能需要这样做:

```java
List<User> users = Arrays.asList(new User(1, "Bob", 10,"America"),new User(2, "Daivd", 22,"America"),new User(3, "Lance", 28,"England"));

Map<String, List<User>> result = new HashMap<>();
for (User user : users) {
  String country = user.getCountry();
  List<User> userList = result.get(country);
  if (userList == null) {
      userList = new ArrayList<>();
      result.put(country, userList);
    }
  userList.add(user);
}
```

如果你使用groupingBy，只需要一行代码就可以了:

```java
Map<String, List<User>> result=users.stream.collect(Collectors.groupingBy(User::getCountry));
```

#### 使用限制和跳过的内存分页
有时我们在内存中聚合数据后，数据量可能太大，无法一次性将数据返回到前端。这个时候，就需要分页。Stream api提供了limit和skip来完美支持它。

```java
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9);
List<Integer> pageableList=list.stream.skip(3).limit(5).collect(Collectors.toList());
```

#### 使用filter()来过滤元素
从集合中过滤掉不符合标准的元素。

```java
//Get all students older than 18
List<Student> studentNames = students.stream().filter(student->student.getAge()>18).collect(Collectors.toList());
```

#### 使用 sorted+Comparator 进行排序
在集合操作中，排序绝对是一个高频的操作。虽然有现成的集合工具进行排序，但Stream api整合了这一功能，形成了自己的功能生态，进一步提高了api的便利性。

```java
List<User> users = Arrays.asList(new User(1, "Bob", 10,"America"),
new User(2, "Daivd", 22,"America"),new User(3, "Lance", 28,"England"));
List<User> sortedUsers = users.stream().sorted(Comparator.comparing(User::getAge))
.collect(Collectors.toList());
```

#### 使用reduce进行聚合评估
Stream api也有一个非常有用的方法reduce()。使用reduce可以聚合集合中的所有元素，得到一个单一的值。最典型的情况是数组的总和。

```java
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9);
Optional<Integer> sum=list.stream().reduce((a,b)->a+b);
```

当然，你也可以使用reduce来执行其他的聚合操作，比如寻找一个数组的平均值。

#### 使用map来转换类型
map操作是对流中的元素进行重新处理，形成一个新的流。这在开发中很有用，例如，我有一个用户的集合，但我只需要使用用户的名字和年龄，使用map方法就可以做到这点。

```java
List<User> users = Arrays.asList(new User(1, "Bob", 10,"America"),new User(2, "Daivd", 22,"America"),new User(3, "Lance", 28,"England"));
List<UserVo> userVoList = users.stream().map(user->{
	UserVo userVo = new UserVo();
	userVo.setUsername(user.getUsername());
	userVo.setAge(user.getAge());
	return userVo;
}).collect(Collectors.toList());
```

#### flatMap
flatMap操作可以将同一类型的多个流合并为一个流。

```java
List<User> users = Arrays.asList(new User(1, "Bob", 10,"America"),new User(2, "Daivd", 22,"America"),new User(3, "Lance", 28,"England"));
List<User> userList = Arrays.asList(new User(4,"James",23,"America"));
List<List<User>> list = Arrays.asList(users,userList);

List<User> result = list.stream().flatMap(s->s.stream()).collect(Collectors.toList());
```

#### 最后
流API以声明的方式处理集合数据，可以极大地提高Java程序员的工作效率，使我们能够写出高效、干净、简洁的代码。
我希望每个人都能掌握Stream Api，写出优雅的代码。谢谢你的阅读!

<!-- https://levelup.gitconnected.com/8-must-know-stream-api-usages-b72ebc50d447 -->