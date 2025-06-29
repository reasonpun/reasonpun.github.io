---
title: C++中的函数递归调用
date: 2024-11-08 16:31:43
categories:
- 算法
tags:
- 算法 递归
---

### 1. 递归的基本概念

**递归（Recursion）** 是指一个函数直接或间接调用自身的编程技术。递归是解决问题的一种常用方法，尤其在涉及分治法、树形结构、回溯等算法时，递归往往能简化问题的解决过程。

<!--more-->

递归函数通常包含两个主要部分：
1. **递归终止条件**：也称为基准情形（Base Case），用于防止无限递归调用。
2. **递归步骤**：函数在满足递归条件时调用自身，并缩小问题的规模。

### 2. 递归函数的构成

```cpp
// 递归函数结构
返回类型 函数名(参数) {
    if (终止条件) {
        // 终止条件处理
        return 特定值;
    } else {
        // 递归调用自身
        return 函数名(新的参数);
    }
}
```

### 3. 递归的优缺点

**优点**：
- 代码简洁，易于理解。
- 适合处理树结构、分治算法等问题。

**缺点**：
- 递归调用会占用栈内存，可能导致**栈溢出**。
- 递归效率较低，通常需要优化（如使用**记忆化搜索**、**尾递归优化**）。

### 4. 递归函数的例子

#### 示例 1：计算阶乘

**问题**：计算 n 的阶乘，定义为 `n! = n * (n-1) * (n-2) * ... * 1`，其中 `0! = 1`。

**递归公式**：
- `factorial(n) = n * factorial(n - 1)`，当 `n > 0`
- `factorial(0) = 1`

**代码**：

```cpp
#include <iostream>
using namespace std;

int factorial(int n) {
    if (n == 0) {
        return 1; // 递归终止条件
    } else {
        return n * factorial(n - 1); // 递归调用
    }
}

int main() {
    int n;
    cout << "Enter a number: ";
    cin >> n;
    cout << "Factorial of " << n << " is " << factorial(n) << endl;
    return 0;
}
```

**解释**：
- 当 `n = 0` 时，直接返回 `1`。
- 对于 `n > 0`，递归调用 `factorial(n - 1)` 直至 `n = 0`。
- 例如，`factorial(4)` 会调用 `factorial(3)`，依次调用直到 `factorial(0)`。

**执行过程**：
- `factorial(4)` → `4 * factorial(3)`
- `factorial(3)` → `3 * factorial(2)`
- `factorial(2)` → `2 * factorial(1)`
- `factorial(1)` → `1 * factorial(0)`
- `factorial(0)` → `1`
- 最终结果为：`4 * 3 * 2 * 1 = 24`

#### 示例 2：斐波那契数列

**问题**：计算斐波那契数列的第 n 项，定义为：
- `fib(0) = 0`
- `fib(1) = 1`
- `fib(n) = fib(n - 1) + fib(n - 2)`，当 `n > 1`

**代码**：

```cpp
#include <iostream>
using namespace std;

int fib(int n) {
    if (n == 0) {
        return 0;
    } else if (n == 1) {
        return 1;
    } else {
        return fib(n - 1) + fib(n - 2);
    }
}

int main() {
    int n;
    cout << "Enter the position: ";
    cin >> n;
    cout << "Fibonacci number at position " << n << " is " << fib(n) << endl;
    return 0;
}
```

**解释**：
- 使用递归公式 `fib(n) = fib(n - 1) + fib(n - 2)` 进行计算。
- 例如，`fib(5)` 需要计算 `fib(4)` 和 `fib(3)`，以此类推。

**执行过程**：
- `fib(5)` → `fib(4) + fib(3)`
- `fib(4)` → `fib(3) + fib(2)`
- `fib(3)` → `fib(2) + fib(1)`
- `fib(2)` → `fib(1) + fib(0)`
- 最终结果为：`5`

**优化**：
- 该递归实现效率低，可以使用**记忆化搜索**或**动态规划**进行优化。

### 5. CSP 竞赛中常见的递归题目

#### 题目：求组合数 C(n, k)

**描述**：
- 给定两个整数 `n` 和 `k`，求组合数 `C(n, k)`。
- 组合数公式：`C(n, k) = C(n - 1, k - 1) + C(n - 1, k)`，且 `C(n, 0) = C(n, n) = 1`。

**解法**：
- 使用递归公式进行计算。

**代码**：

```cpp
#include <iostream>
using namespace std;

int C(int n, int k) {
    if (k == 0 || k == n) {
        return 1;
    } else {
        return C(n - 1, k - 1) + C(n - 1, k);
    }
}

int main() {
    int n, k;
    cout << "Enter n and k: ";
    cin >> n >> k;
    cout << "C(" << n << ", " << k << ") = " << C(n, k) << endl;
    return 0;
}
```

**解释**：
- 当 `k == 0` 或 `k == n` 时，返回 `1`。
- 否则通过公式 `C(n, k) = C(n-1, k-1) + C(n-1, k)` 计算。

**执行过程**：
- `C(5, 2)` → `C(4, 1) + C(4, 2)`
- `C(4, 1)` → `C(3, 0) + C(3, 1)`，返回 `4`
- `C(4, 2)` → `C(3, 1) + C(3, 2)`，返回 `6`
- 最终结果为：`10`

### 6. 尾递归优化

尾递归是指递归调用出现在函数的最后一步时的一种特殊递归形式，可以通过编译器优化为迭代，避免栈溢出。

**示例**：计算阶乘的尾递归实现

```cpp
#include <iostream>
using namespace std;

int factorialTail(int n, int result) {
    if (n == 0) {
        return result;
    } else {
        return factorialTail(n - 1, n * result);
    }
}

int main() {
    int n;
    cout << "Enter a number: ";
    cin >> n;
    cout << "Factorial of " << n << " is " << factorialTail(n, 1) << endl;
    return 0;
}
```

**解释**：
- `factorialTail(n, result)` 中，`result` 保存当前计算的结果。
- 递归调用出现在函数的最后一步，因此为尾递归。

### 7. 常见 CSP 递归题目

#### 题目：汉诺塔问题

**描述**：
- 有三根柱子 `A`、`B`、`C`，将 `n` 个盘子从 `A` 移到 `C`，每次只能移动一个盘子且大盘子不能放在小盘子上。

**代码**：

```cpp
#include <iostream>
using namespace std;

void hanoi(int n, char from, char to, char aux) {
    if (n == 1) {
        cout << "Move disk 1 from " << from << " to " << to << endl;
    } else {
        hanoi(n - 1, from, aux, to);
        cout << "Move disk " << n << " from " << from << " to " << to << endl;
        hanoi(n - 1, aux, to, from);
    }
}

int main() {
    int n;
    cout << "Enter number of disks: ";
    cin >> n;
    hanoi(n, 'A', 'C', 'B');
    return 0;
}
```

**解释**：
- 分两步：将 `n-1` 个盘子从 `A` 移到 `B`，再将第 `n` 个盘子从 `A` 移到 `C`，最后将 `n-1` 个盘子从 `B` 移到 `C`

。
- 复杂度为 `O(2^n)`。

**输出**：
- 输入 `3` 时：
```
Move disk 1 from A to C
Move disk 2 from A to B
Move disk 1 from C to B
Move disk 3 from A to C
Move disk 1 from B to A
Move disk 2 from B to C
Move disk 1 from A to C
```

### 8. 角谷定理

角谷定理（也称为“3n+1”问题）是一个著名的数学猜想，它描述了一个简单的数学过程：对于任意一个正整数 n，如果 n 是偶数，则将其除以 2；如果 n 是奇数，则将其变为 3n+1。重复这个过程，最终 n 会变为 1。

```cpp
#include <iostream>
using namespace std;

// 递归函数，实现角谷定理的计算过程
void collatzConjecture(int n) {
    // 基本情况：如果 n 等于 1，则结束递归
    if (n == 1) {
        cout << n << endl;
        return;
    }

    // 输出当前的 n 值
    cout << n << " -> ";

    // 如果 n 是偶数，则除以 2
    if (n % 2 == 0) {
        collatzConjecture(n / 2);
    }
    // 如果 n 是奇数，则计算 3n + 1
    else {
        collatzConjecture(3 * n + 1);
    }
}

int main() {
    int n;
    cout << "请输入一个正整数：";
    cin >> n;

    // 检查输入是否为正整数
    if (n <= 0) {
        cout << "输入无效，请输入一个正整数！" << endl;
        return 1;
    }

    cout << "角谷定理的计算过程如下：" << endl;
    collatzConjecture(n);

    return 0;
}

```

#### 程序说明

 1. 递归函数 collatzConjecture：
   - 函数接收一个整数 n 作为参数。
   - 如果 n 等于 1，递归结束，并输出 1。
   - 如果 n 是偶数，递归调用自身，传入 n/2。
   - 如果 n 是奇数，递归调用自身，传入 3n+1。
 2. 主函数 main：
   - 提示用户输入一个正整数。
   - 检查输入是否合法（必须是正整数）。
   - 调用递归函数 collatzConjecture，并输出计算过程。

#### 示例运行

假设用户输入 6，程序的输出如下：

```
请输入一个正整数：6
角谷定理的计算过程如下：
6 -> 3 -> 10 -> 5 -> 16 -> 8 -> 4 -> 2 -> 1
```

### 9. 总结

- 递归是一种强大的编程思想，适用于处理复杂的分治、树形结构和动态规划问题。
- 使用递归时需要注意**终止条件**，防止无限递归导致栈溢出。
- 尾递归可以优化递归的空间复杂度，但并不是所有递归问题都适合转换为尾递归。