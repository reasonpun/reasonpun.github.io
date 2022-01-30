---
layout: post
title: Python关键字:yield
---


注：以下代码实验环境均为

```
➜  ~  python
Python 2.7.10 (default, Oct 23 2015, 18:05:06)
[GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.0.59.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
```

### Yield关键字

理解yield关键字之前需要先明白什么是迭代。

#### 可迭代对象

建立一个列表之后，可以逐项的读取列表中的元素，这就是一个可迭代的对象：

```
>>> list = [1, 2, 3, 4]
>>> for i in list:
...     print i
...
1
2
3
4
>>>
```

使用列表生成器建立一个列表，同样也是创建了一个可迭代的对象：

```
>>> list = [x * 2 for x in range(3)]
>>> list
[0, 2, 4]
>>> for i in list:
...     print i
...
0
2
4
>>>
```

可以使用for .. in .. 的方式处理：链表，字符串等，这就叫做一个迭代器。但是这样会将数据存放到内存中，如果数据量过大的话，就非常不适合了。

#### 生成器

[生成器是可以迭代的，但是你 只可以读取它一次 ，因为它并不把所有的值放在内存中，它是实时地生成数据](http://pyzh.readthedocs.org/en/latest/the-python-yield-keyword-explained.html)

```
>>> g = (x * 2 for x in range(3))
>>> g
<generator object <genexpr> at 0x10978ca00>
>>> for i in g:
...     print i
...
0
2
4
>>> for i in g:
...     print i
...
>>>
>>>
```

#### yield关键字

yield 是一个类似return的关键字，但是这个函数返回的是生成器。

同时我们可以利用 isgeneratorfunction 判断一个特殊的 generator 函数

```
>>> mg = createGenerator()
>>> print (mg)
<generator object createGenerator at 0x10978c9b0>
>>> for i in mg:
...     print i
...
0
2
4
>>> from inspect import isgeneratorfunction
>>> isgeneratorfunction(mg)
False
>>> isgeneratorfunction(createGenerator)
True
>>>
```
需要注意的是：_当你调用这个函数的时候，函数内部的代码并不立马执行_ ，而是返回一个生成器对象。只有当使用for进行迭代的时候，函数内的代码才会执行。

#### 控制资源访问

```
>>> class Bank():
...     crisis = False
...     def createAtm(self):
...             while not self.crisis:
...                     yield "$100"
...
>>>
>>> bank = Bank()
>>> atm = bank.createAtm()
>>> print(atm.next())
$100
>>> print(atm.next())
$100
>>> print(atm.next())
$100
>>> print(atm.next())
$100
>>> atm.crisis = True
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'generator' object has no attribute 'crisis'
>>> bank.crisis = True
>>> print (atm.next())
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>>
```
如果不使用yield的话，这个类执行createAtm之后就会导致系统资源耗尽（或者说是无限循环）

#### 使用yield可以是程序非常优美，虽然python程序本身就会很优美

##### 生成斐波那契（Fibonacci）数列

许多初学者都会这么写

```
>>> def fab(max):
...     n, a, b = 0, 0, 1
...     while n < max:
...             print b
...             a, b = b, a + b
...             n = n + 1
...
>>> fab(5)
1
1
2
3
5
>>>
```
使用yield改写一个版本

```
>>> def fab(max):
...     n, a, b = 0, 0, 1
...     while n < max:
...             yield b
...             a, b = b, a + b
...             n = n + 1
...
>>>
>>> for n in fab(5):
...     print n
...
1
1
2
3
5
>>>
```
#### 简单描述下

for 语句在碰到生成器 generator 的时候，

调用

```
generator.__next__()
```

获取生成器的返回值。

```
__next__()
```

以Fibonacci为例：for每次调用，可以理解为执行了一次generator() 执行到 yield 的时候，生成器返回了n的值并停止。
这就像普通函数碰到 return 时一样，剩下的代码都被忽略了。

不同的地方在于，python 会记录这个停止的位置。 当再次执行generator()的时候，python 从这个停止位置开始执行而不是开头， 也就是说这次返回了1。再执行generator()，则返回2，当执行到返回5的时候，已经没有 yield 语句了， 就抛出了 StopIteration 。这和其他迭代器是类似的，当然在for中是不会抛出异常的。

#### 进阶

在[PEP 342](https://www.python.org/dev/peps/pep-0342/)中加入了将值传给生成器的支持。PEP 342加入了新的特性，能让生成器在单一语句中实现，生成一个值（像从前一样），接受一个值，或同时生成一个值并接受一个值。
我们用前面那个关于素数的函数来展示如何将一个值传给生成器。这一次，我们不再简单地生成比某个数大的素数，而是找出比某个数的等比级数大的最小素数（例如10， 我们要生成比10，100，1000，10000 ... 大的最小素数）。

```
>>> import math
>>> def getPrimes(number):
...     while True:
...             if isPrime(number):
>>>                     '''
...                     yield关键字返回number的值，
...                     而other = yield foo "返回foo的值，
...                     这个值返回给调用者的同时，将other的值也设置为那个值"。
...                     可以通过send方法来将一个值”发送“给生成器。
>>>                     '''
...                     number = yield number
...             number += 1
...
>>> def printSuccessivePrimes(iterations, base=10):
...     printGenerator = getPrimes(base)
>>>     '''
...     用send来“启动”一个生成器时（就是从生成器函数的第一行代码执行到第一个yield语句的位置），必须发送None。因为此时生成器还没有走到第一个yield语句，如果send一个真实的值，这时是没有人去“接收”它的。一旦生成器启动了，我们就可以像上面那样发送数据了
>>>     '''
...     printGenerator.send(None)
...     for power in range(iterations):
>>>             '''
...             打印的是generator.send的结果，send在发送数据给生成器的同时还返回生成器通过yield生成的值
>>>             '''
...             print(printGenerator.send(base ** power))
...
>>> def isPrime(number):
...     if number > 1:
...             if number == 2:
...                     return True
...             if number % 2 == 0:
...                     return False
...             for current in range(3, int(math.sqrt(number) + 1), 2):
...                     if number % current == 0:
...                             return False
...             return True
...     return False
...
>>>
>>> printSuccessivePrimes(10)
2
11
101
1009
10007
100003
1000003
10000019
100000007
1000000007
>>>
```
Python 使用yield的关键思想
  * generator是用来产生一系列值的
  * yield则像是generator函数的返回结果
  * yield唯一所做的另一件事就是保存一个generator函数的状态
  * generator就是一个特殊类型的迭代器（iterator）
  * 和迭代器相似，我们可以通过使用next()来从generator中获取下一个值
  * 通过隐式地调用next()来忽略一些值

### 参考文献
  * http://pyzh.readthedocs.org/en/latest/the-python-yield-keyword-explained.html
  * http://www.dabeaz.com/coroutines/index.html
  * https://docs.python.org/2/tutorial/classes.html
  * http://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do-in-python
  * http://www.pydanny.com/python-yields-are-fun.html
  * http://dhcmrlchtdj.github.io/sia/post/2012-11-20/python_yield.html
  * http://www.oschina.net/translate/improve-your-python-yield-and-generators-explained
  * http://www.pythonclub.org/python-basic/yield
  * https://www.ibm.com/developerworks/cn/opensource/os-cn-python-yield/
  * https://www.jeffknupp.com/blog/2013/04/07/improve-your-python-yield-and-generators-explained/
  * http://pythontips.com/2013/09/29/the-python-yield-keyword-explained/
  * http://blog.jobbole.com/28506/
