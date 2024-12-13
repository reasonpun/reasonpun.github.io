---
title: C++中的顺序搜索
date: 2024-12-13 17:20:53
categories:
- 算法
tags:
- 算法 二分搜索
---

顺序查找，也称为线性查找，是一种最基本的查找技术，用于在数据结构（如数组或列表）中搜索特定的值。

<!--more-->

### 知识点讲解：

1. **基本思想**：顺序查找从数据结构的一端开始，逐个检查每个元素，直到找到所需的值或到达数据结构的末尾。

2. **适用场景**：顺序查找适用于无序数据结构，或者数据量较小时的查找操作。

3. **时间复杂度**：顺序查找的时间复杂度为O(n)，其中n是数据结构中元素的数量。在最坏的情况下，可能需要检查所有元素。

4. **实现方式**：顺序查找可以通过循环结构（如for循环或while循环）来实现。

5. **返回值**：如果找到目标值，返回其在数据结构中的位置（索引）；如果未找到，通常返回-1或其他特定的值表示查找失败。

### 代码示例：

下面是一个使用顺序查找算法在C++数组中查找特定值的代码示例，以及详细的注释：

```cpp
#include <iostream>
#include <vector>

// 顺序查找函数，参数为整数向量、目标值
int sequentialSearch(const std::vector<int>& arr, int target) {
    for (int i = 0; i < arr.size(); ++i) { // 遍历数组中的每个元素
        if (arr[i] == target) { // 如果当前元素等于目标值
            return i; // 返回当前元素的索引
        }
    }
    return -1; // 如果遍历结束仍未找到目标值，返回-1
}

int main() {
    std::vector<int> arr = {5, 3, 7, 1, 9, 4}; // 待查找的数组
    int target = 7; // 要查找的目标值
    int result = sequentialSearch(arr, target); // 调用顺序查找函数

    if (result != -1) { // 如果找到了目标值
        std::cout << "Element found at index " << result << std::endl;
    } else { // 如果没有找到目标值
        std::cout << "Element not found in the array" << std::endl;
    }

    return 0;
}
```

### 注释解释：

- `#include <iostream>` 和 `#include <vector>`：引入标准输入输出流和向量容器的头文件。
- `sequentialSearch` 函数：定义了一个顺序查找的函数，接受一个整数向量和一个目标值作为参数。
- `for` 循环：遍历数组中的每个元素，从索引0开始，直到数组的末尾。
- `if` 语句：检查当前元素是否等于目标值。
- `return i`：如果找到目标值，返回其在数组中的索引。
- `return -1`：如果遍历结束仍未找到目标值，返回-1表示查找失败。
- `main` 函数：定义了一个待查找的数组和一个目标值，调用 `sequentialSearch` 函数，并根据返回结果输出相应的信息。

希望这个详细的讲解和代码示例能帮助孩子们理解顺序查找算法的工作原理和实现方式。

### 历年真题
### 1. CSP-J2019 第一题
**题目描述**：
题目要求计算所圈的矩阵和其他已有矩阵的交集面积。交集面积的右边界是两个相交矩阵的最右边的边界，即`min(a, points[i][2])`，左边界是两个相交矩阵的最左边的边界，即`max(0, points[i][0])`。上下边也是一样的道理。通过判断`x`和`y`是否大于零，可以判断出矩阵是否交叉，若存在则`sum += x * y`。

**解题思路**：
- 使用顺序查找（线性查找）来遍历所有矩阵的点。
- 对于每个点，计算交集的边界，然后计算交集面积。

**代码示例**：
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    int n; // 矩阵的数量
    cin >> n;
    vector<pair<int, int>> points; // 存储矩阵的左右边界或上下边界
    int sum = 0; // 存储总的交集面积

    for (int i = 0; i < n; ++i) {
        int a, b, c, d;
        cin >> a >> b >> c >> d; // 读取矩阵的左右边界和上下边界
        points.push_back({a, b}); // 存储左边界
        points.push_back({c, d}); // 存储右边界
    }

    sort(points.begin(), points.end()); // 对点进行排序

    // 遍历所有点，计算交集面积
    for (int i = 0; i < points.size(); i += 2) {
        int x = points[i+1].first - points[i].second; // 计算交集的宽度
        int y = points[i+3].second - points[i+2].first; // 计算交集的高度
        if (x > 0 && y > 0) { // 如果交集存在
            sum += x * y; // 累加交集面积
        }
    }

    cout << sum << endl; // 输出总的交集面积
    return 0;
}
```

以上是CSP-J2019普及组初赛中的一道题目，涉及到顺序查找算法的应用。通过线性遍历所有矩阵的边界点，计算出所有矩阵的交集面积。
