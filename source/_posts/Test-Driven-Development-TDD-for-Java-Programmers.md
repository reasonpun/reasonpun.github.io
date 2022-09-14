---
title: Java程序员的测试驱动开发(TDD)
date: 2022-06-27 10:28:08
categories:
- Java
- TDD
tags:
- SprJavaing
- TDD
---

最常见但又被低估的做法之一是写代码而不实施测试，甚至大多数专业人士也是如此。
尽管一个明显的事实是，如果你理解了业务逻辑，你就可以直接深入到执行中去，但这并不意味着你遵循了被编程专家长期证明的最佳实践。
实现你的程序的最好方法之一是遵循TDD或测试驱动的开发。
在这篇文章中，我们将看看什么是TDD，以及如何使用TDD进行更好的编程。

什么是TDD？
TDD仅仅意味着我们使用测试来驱动代码的实现。但实际上，它是一个从红色到绿色到重构的工作流程。

{% asset_img 1.png 示意图 width="400" %}

<!--more-->

最初，一个新的测试将从红色状态开始，意味着它是一个失败的状态。
在初始状态下失败后，我们将纠正测试和逻辑，使其通过或达到绿色状态。一旦我们的测试通过了，我们就可以重构测试和业务逻辑的实现，使它们更有效率。如果重构导致测试回到红色状态，我们将修正测试以进入绿色状态，然后再次重构，从而形成一个循环。

#### ISBN验证

为了更详细地了解TDD，我们将看看我们如何在一个项目中使用TDD。在这个项目中，我们将研究如何验证一本书的ISBN号码。ISBN是国际标准书号的缩写，这些数字是你在任何书的条形码上面找到的。一个ISBN号码的有效性是基于以下简单的逻辑。

如果ISBN号码包含10个数字，并且它们都是数字，那么ISBN数字与数值10到1的乘积之和除以11应该是一个整数值。

{% asset_img 2.png 示意图 width="400" %}

在上面的例子中，总和是132，当它被11除以时，我们得到12，一个整数值。因此，这是一个有效的ISBN号码。

如果ISBN号码包含9个数字，最后一个数字是字母'X'，那么ISBN数字的乘积的总和是10到1，考虑到'X'是10，除以11应该得到一个整数值。

{% asset_img 3.png 示意图 width="400" %}

这里，产品的总和是209，当它被11除以时，我们得到19，这是一个整数值。

最后，如果ISBN号码包含13位数字，那么这些数字应该交替地乘以1和3。
之后，总和除以10应该得到一个整数值。

{% asset_img 4.png 示意图 width="400" %}

这里，产品的总和是100，当它被10除以时，我们得到10，这是一个整数值。

#### 在IntelliJ设置项目

为此，我们将创建一个新的maven项目。
我们将使用IntelliJ IDEA创建一个新的maven项目，请进入文件→新建→项目，选择maven。然后将com.book作为组ID，将tdd作为工件ID。生成项目后，进入pom.xml文件，添加以下内容，将JUnit作为一个依赖项。

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

接下来，到 src → main → java，创建一个新的包 com.book.tdd，并在其中添加一个名为 ValidateISBN.java 的新类。然后，转到 src → test → java，创建一个新的包 com.book.tdd，并在其中添加一个名为 ValidateISBNTests.class 的新类。加入上述配置后，你的项目应该是这样的。

{% asset_img 5.png 示意图 width="400" %}

由于我们使用的是TDD，首先我们将进入ValidateISBNTest.java文件并创建我们的第一个测试。这是因为，在TDD中，代码的实现是由测试驱动的。

```java
@Test
public void checkValidateISBN(){
    fail("Not implemented");
}
```

正如我们前面提到的，我们要从红色到绿色。
因此，最初我们的测试是失败的，得到了红色状态。
为了运行这个测试，我们将使用import org.junit.Test导入测试；通过import org.junit.Assert.*导入断言；这里，我们使用了静态方法导入，因为我们不想在代码中使用断言，如Assert.assertTrue()。而当你使用沟渠运行图标运行这个测试时，你会得到以下输出：

{% asset_img 6.png 示意图 width="400" %}

现在我们需要进入绿色状态，要做到这一点，我们可以删除第10行，fail("Not Implemented")；并添加我们期望的方法来检查有效性。为此，我们可以像这样调整我们的测试。

```java
@Test
public void checkValidateISBN(){
    ValidateISBN validateISBN = new ValidateISBN();
    boolean result = validateISBN.checkISBN(0140441926);
}
```

你可以注意到，checkISBN(int isbn)方法还没有实现，我们不一定说参数类型应该是字符串或其他什么。我们假设它是int，因为它是一个数字。如果这个数字不是这个类型，我们可以在以后通过测试结果来验证它。

当我们运行这个程序时，我们得到以下错误，说整数太大。

{% asset_img 7.png 示意图 width="400" %}

由于这个原因，我把变量改成了字符串，并再次检查。现在我得到了以下错误:

{% asset_img 8.png 示意图 width="400" %}

我知道这是因为我没有这个方法。
因此，我点击发红的方法，在Windows中按alt+enter或者在Mac中按option+enter来选择我所拥有的解决这个问题的选项，从下拉列表中我选择，在'ValidateISBN'中创建方法'checkISBN'，在我们的ValidateISBN.java文件中创建一个新方法。

{% asset_img 9.png 示意图 width="400" %}

在ValidateISBN.java文件中创建checkISBN()方法，并返回false作为默认值后，我们可以再次运行测试。

{% asset_img 10.png 示意图 width="400" %}

在这里，我们得到一个AssertonError，因为我们对一个假值断言为真。如果我们把第12行改为assertFalse(result)；测试将通过，因为我们断言的是假值。

现在让我们进入ValidateISBN.java文件，编写我们的实现。在这里，要记住的最重要的事情之一是编写只需要通过测试的代码。
在这里，为了通过我们的测试，我们可以把return false;改为return true;现在我们的代码处于绿色状态。

现在让我们说，我想验证一个无效的ISBN号码。那么我可以在ValidateISBNTests.java文件中写下以下测试。

```java
@Test
public void checkInvalidISBN(){
    ValidateISBN validateISBN = new ValidateISBN();
    boolean result = validateISBN.checkISBN("0140441927");
    assertFalse(result);
}
```

现在，当我们运行这个方法时，它失败了，因为我们总是从checkISBN()方法返回true。
现在，由于我们处于红色状态，我们需要写一些代码来进入绿色状态。我们将首先写出我们需要为ISBN号码验证所做的第一个检查，我们在前面讨论过；如果数字的数量等于10，并且都是数字。

```java
public boolean checkISBN(String isbn) {

    int total = 0;
    for (int i = 0; i < 10; i++) {
        total += isbn.charAt(i) * (10 - i);
    }
    if (total % 11 == 0) {
        return true;
    } else {
        return false;
    }
}
```

现在，如果我们运行我们所写的测试，我们可以看到checkInvalidISBN()没有错误，但checkValidateISBN()却有错误，这是由于我们所检查的ISBN号码是无效的。我们可以用我们用来讨论ISBN号码背后的逻辑的那个号码来代替它，0140449116。

为了给checkValidateISBN()起一个更有意义的名字，我们可以把它的名字重构为checkValidISBN() 现在我们可以再次运行测试。

{% asset_img 11.png 示意图 width="400" %}

而现在我们需要添加另一个测试来验证ISBN号码的长度。我们知道ISBN号码只能是长度为10或13的。所以我们将运行一个测试，证明ISBN的长度不能超过10或13。
要做到这一点，我们首先要写一个失败状态的测试。

```java
@Test
public void checkISBNLengthTenOrThirteen(){
    fail();
}
```

而如果我们将其调整为以下内容并运行测试，我们会得到一个StringIndexOutOfBoundsExceptions，因为这只包含9位数字。

```java
@Test
public void checkISBNLengthTenOrThirteen(){
    ValidateISBN validateISBN  = new ValidateISBN();
    boolean result = validateISBN.checkISBN("012345678");
}

```
虽然我们可以创建一个新的异常来处理这个问题，但我们可以使用Java的NumberFormatException来处理这个问题。由于我们知道这是不对的，而且我们在期待一个错误，所以我们可以将测试注解改为我们所期待的错误。

```java
@Test(expected = NumberFormatException.class)
public void checkISBNLengthTenOrThirteen(){
    ValidateISBN validateISBN  = new ValidateISBN();
    boolean result = validateISBN.checkISBN("012345678");
}
```
{% asset_img 12.png 示意图 width="400" %}

但是测试说它得到了一个意外的异常，因为它之前得到的是StringOutOfBoundsException。那么我们怎样才能解决这个问题呢？为了解决这个问题，我们需要更新我们的checkISBN()方法。

```java
public boolean checkISBN(String isbn) {

    if (isbn.length() == 10 || isbn.length() == 13) {
        int total = 0;
        for (int i = 0; i < 10; i++) {
            total += isbn.charAt(i) * (10 - i);
        }
        if (total % 11 == 0) {
            return true;
        } else {
            return false;
        }
    } else {
        throw new NumberFormatException("ISBN number should have length of 10 or 13");
    }
}

```

因此，现在当我们运行测试时，我们得到了预期的结果。

{% asset_img 13.png 示意图 width="400" %}

由于变量result是灰色的，这意味着，resultvariable是不必要的。因此，我们可以将这个方法重构为以下内容。

```java
@Test(expected = NumberFormatException.class)
public void checkISBNLengthTenOrThirteen(){
    ValidateISBN validateISBN  = new ValidateISBN();
    validateISBN.checkISBN("012345678");
}

```

现在我们需要测试我们传递给checkISBN()的ISBN是否是数字值（虽然我们讨论过ISBN数字中的字母X）。为了检查它，我们将实现以下测试。

```java
@Test
public void checkISBNNumeric(){
    fail();
}
```

之后，我们可以将其调整为以下代码。由于我们传递的是一个字符串，我们可以期待一个NumberFormatException

```java
@Test(expected = NumberFormatException.class)
public void checkISBNNumeric(){
    ValidateISBN validateISBN = new ValidateISBN();
    validateISBN.checkISBN("helloworld");
}
```

如果我们运行这个，我们会得到一个错误。这是因为我们应该看的是数字而不是字母。为了解决这个问题，我们对checkISBN()的代码做了一些调整。

```java
public boolean checkISBN(String isbn) {

    if (isbn.length() == 10 || isbn.length() == 13) {
        int total = 0;
        for (int i = 0; i < 10; i++) {
            if (!Character.isDigit(isbn.charAt(i))){
                throw new NumberFormatException("ISBN numbers can only have digits.");
            }
            total += isbn.charAt(i) * (10 - i);
        }
        if (total % 11 == 0) {
            return true;
        } else {
            return false;
        }
    } else {
        throw new NumberFormatException("ISBN number should have length of 10 or 13");
    }
```

而现在，当我们运行测试时，它通过了。
{% asset_img 14.png 示意图 width="400" %}

现在我们可以检查那些包含9位数字和字母'X'的ISBN号码。为了检查，我们可以创建一个新的方法，以fail();作为初始状态。

```java
@Test
public void checkContainsX(){
    fail();
}
```

现在我们可以在测试中加入以下内容，当我们运行它时，我们会得到一个NumberFormatException，这要感谢我们之前做的测试和代码修正。

```java
@Test
public void checkContainsX(){
    ValidateISBN validateISBN = new ValidateISBN();
    boolean result = validateISBN.checkISBN("080442957X");
    assertTrue(result);
}
```
{% asset_img 15.png 示意图 width="400" %}

现在，为了解决这个问题，我们可以改变我们的checkISBN()方法。

```java
public boolean checkISBN(String isbn) {

    if (isbn.length() == 10 || isbn.length() == 13) {
        int total = 0;
        for (int i = 0; i < 10; i++) {
            if (!Character.isDigit(isbn.charAt(i))) {
                if (i == 9 && isbn.charAt(i) == 'X') {
                    //do nothing
                } else {
                    throw new NumberFormatException("ISBN numbers can only have digits.");
                }
            }
            total += isbn.charAt(i) * (10 - i);
        }
        if (total % 11 == 0) {
            return true;
        } else {
            return false;
        }
    } else {
        throw new NumberFormatException("ISBN number should have length of 10 or 13");
    }
}
```

你可以看到，如果(i==9 && isbn.charAt(i)=='X')我们什么都不做，否则我们就抛出一个异常。而如果我们运行这个，我们会得到一个错误，说我们的断言是错误的。
这是因为我们需要将10乘以1加到总数中。所以我们可以通过在添加注释的地方添加total+=10;来解决这个问题（或者什么都不做），并在total += isbn.charAt(i) * (10 - i); 处添加一个else块来解决我们的正常流程。

```java
public boolean checkISBN(String isbn) {

    if (isbn.length() == 10 || isbn.length() == 13) {
        int total = 0;
        for (int i = 0; i < 10; i++) {
            if (!Character.isDigit(isbn.charAt(i))) {
                if (i == 9 && isbn.charAt(i) == 'X') {
                    total += 10;
                } else {
                    throw new NumberFormatException("ISBN numbers can only have digits.");
                }
            } else {
                total += isbn.charAt(i) * (10 - i);
            }
        }
        if (total % 11 == 0) {
            return true;
        } else {
            return false;
        }
    } else {
        throw new NumberFormatException("ISBN number should have length of 10 or 13");
    }
}
```

而现在，当我们运行这个测试时，我们得到的仍然是一个错误。

{% asset_img 16.png 示意图 width="400" %}

为了理解这一点，我们将在checkISBN()方法的第21行和第23行添加调试指针，并对测试进行调试。

{% asset_img 17.png 示意图 width="400" %}

而当我们调试时，我们得到的总数是2801，这是不可能的，因为我们知道总数是209（我们之前讨论的例子）。

{% asset_img 18.png 示意图 width="400" %}

那么，为什么会出现这种情况呢？这是因为第11行的isbn.charAt()方法返回了ASCII值，而这个值被乘以了。我们可以使用 Characters.getNumericValue() 方法来解决这个问题。
在这之后，checkISBN()方法看起来像这样:

```java
public boolean checkISBN(String isbn) {

    if (isbn.length() == 10 || isbn.length() == 13) {
        int total = 0;
        for (int i = 0; i < 10; i++) {
            if (!Character.isDigit(isbn.charAt(i))) {
                if (i == 9 && isbn.charAt(i) == 'X') {
                    total += 10;
                } else {
                    throw new NumberFormatException("ISBN numbers can only have digits.");
                }
            } else {
                total += Character.getNumericValue(isbn.charAt(i)) * (10 - i);
            }
        }
        if (total % 11 == 0) {
            return true;
        } else {
            return false;
        }
    } else {
        throw new NumberFormatException("ISBN number should have length of 10 or 13");
    }
}
```

而现在，当我们运行checkContainsX()测试时，它被通过了。

{% asset_img 19.png 示意图 width="400" %}

现在，我们已经实现了我们对10位数的ISBN号码和9位数的ISBN号码以及'X'的代码。所以现在我们唯一需要完成的部分是13位数的ISBN数字。为了实现这一点，我们将首先创建一个测试。

```java
@Test
public void checkValidThirteenDigitISBN(){
    fail();
}
```

现在我们可以调整我们的测试用例，对13位数的数字进行检查。

```java
@Test
public void checkValidThirteenDigitISBN(){
    ValidateISBN validateISBN = new ValidateISBN();
    boolean result = validateISBN.checkISBN("9780306406157");
    assertTrue(result);
}
```

我们知道这是一个ISBN号码，因为这是我们用来讨论业务逻辑的号码。但是当我们运行这个时，我们得到一个错误。

{% asset_img 20.png 示意图 width="400" %}

这是因为我们还没有实现这方面的业务逻辑。要做到这一点，我们可以将checkISBN()方法改为以下代码片断。

```java
public boolean checkISBN(String isbn) {

    if (isbn.length() == 10 || isbn.length() == 13) {
        if (isbn.length() == 13) {
            int total = 0;
            for (int i = 0; i < 13; i++) {
                if (Character.isDigit(isbn.charAt(i))) {
                    if (i % 2 == 0) {
                        total += Character.getNumericValue(isbn.charAt(i)) * 1;
                    } else {
                        total += Character.getNumericValue(isbn.charAt(i)) * 3;
                    }
                } else {
                    throw new NumberFormatException("ISBN numbers can only have digits.");
                }
            }
            if (total % 10 == 0) {
                return true;
            } else {
                return false;
            }
        } else {
            int total = 0;
            for (int i = 0; i < 10; i++) {
                if (!Character.isDigit(isbn.charAt(i))) {
                    if (i == 9 && isbn.charAt(i) == 'X') {
                        total += 10;
                    } else {
                        throw new NumberFormatException("ISBN numbers can only have digits.");
                    }
                } else {
                    total += Character.getNumericValue(isbn.charAt(i)) * (10 - i);
                }
            }
            if (total % 11 == 0) {
                return true;
            } else {
                return false;
            }
        }
    } else {
        throw new NumberFormatException("ISBN number should have length of 10 or 13");
    }
}
```

在改变checkISBN()方法并运行测试后，它得到了通过。

{% asset_img 21.png 示意图 width="400" %}

虽然我们的代码checkISBN()工作正常，但看起来很乱。为了清理它，我们进行了重构。这是通过创建常量和方法来完成的。重构代码后，它看起来像这样。

```java
package com.book.tdd;

public class ValidateISBN {

    public static final int ISBN_SHORT = 10;
    public static final int ISBN_LONG = 13;
    public static final int ISBN_SHORT_VALIDATOR = 11;
    public static final int ISBN_LONG_VALIDATOR = 10;

    public boolean checkISBN(String isbn) {

        if (isbn.length() == ISBN_SHORT || isbn.length() == ISBN_LONG) {
            if (isbn.length() == ISBN_LONG) {
                return checkISBNThirteenDigits(isbn);
            } else {
                return checkISBNTenDigits(isbn);
            }
        } else {
            throw new NumberFormatException("ISBN number should have length of 10 or 13");
        }
    }

    private boolean checkISBNTenDigits(String isbn) {

        int total = 0;
        for (int i = 0; i < ISBN_SHORT; i++) {
            if (!Character.isDigit(isbn.charAt(i))) {
                if (i == 9 && isbn.charAt(i) == 'X') {
                    total += 10;
                } else {
                    throw new NumberFormatException("ISBN numbers can only have digits.");
                }
            } else {
                total += Character.getNumericValue(isbn.charAt(i)) * (ISBN_SHORT - i);
            }
        }
        if (total % ISBN_SHORT_VALIDATOR == 0) {
            return true;
        }
        return false;
    }

    private boolean checkISBNThirteenDigits(String isbn) {

        int total = 0;
        for (int i = 0; i < ISBN_LONG; i++) {
            if (Character.isDigit(isbn.charAt(i))) {
                if (i % 2 == 0) {
                    total += Character.getNumericValue(isbn.charAt(i)) * 1;
                } else {
                    total += Character.getNumericValue(isbn.charAt(i)) * 3;
                }
            } else {
                throw new NumberFormatException("ISBN numbers can only have digits.");
            }
        }
        if (total % ISBN_LONG_VALIDATOR == 0) {
            return true;
        }
        return false;
    }
}
```

现在，如果我们运行测试套件，我们得到以下结果:

{% asset_img 22.png 示意图 width="400" %}

但是看一下我们的测试文件，我们可以看到我们正在反复使用变量validateISBN。为了避免这一点，我们可以使用 @Before 注解。
在使用方法的注解后，我们得到以下的最终测试套件:

```java
package com.book.tdd;

import org.junit.Before;
import org.junit.Test;

import static org.junit.Assert.*;

public class ValidateISBNTests {

    ValidateISBN validateISBN;

    @Before
    public void setup() {

        validateISBN = new ValidateISBN();
    }

    @Test
    public void checkValidISBN() {

        boolean result = validateISBN.checkISBN("0140449116");
        assertTrue(result);
    }

    @Test
    public void checkInvalidISBN() {

        boolean result = validateISBN.checkISBN("0140441927");
        assertFalse(result);
    }

    @Test(expected = NumberFormatException.class)
    public void checkISBNLengthTenOrThirteen() {

        validateISBN.checkISBN("012345678");
    }

    @Test(expected = NumberFormatException.class)
    public void checkISBNNumeric() {

        validateISBN.checkISBN("helloworld");
    }

    @Test
    public void checkContainsX() {

        boolean result = validateISBN.checkISBN("080442957X");
        assertTrue(result);
    }

    @Test
    public void checkValidThirteenDigitISBN() {

        boolean result = validateISBN.checkISBN("9780306406157");
        assertTrue(result);
    }
}
```

#### 最后说明
你可以看到，通过使用TDD，我们可以构建无错误的、高效的代码。这也是大多数公司将TDD作为最佳实践的主要原因。

虽然我们在这里只使用了JUnit，但在做TDD时，几乎不可能只依赖JUnit。我们必须使用多个测试库来测试我们的代码，这取决于我们的实现。但所有这些都是为了在我们的实现过程中使用TDD。我希望你们已经学会了如何使用TDD进行高效的实现。

<!-- https://medium.com/javarevisited/test-driven-development-tdd-for-java-programmers-cb73878afdde -->