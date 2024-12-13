---
title: C++中的二分搜索
date: 2024-12-13 17:19:07
categories:
- 算法
tags:
- 算法 二分搜索
---

二分查找是一种在有序数组中查找特定元素的高效算法。它的基本思想是将数组分成两半，然后根据目标值与中间元素的大小关系来决定是继续在左半部分还是右半部分进行查找，这个过程会不断重复，直到找到目标值或者搜索范围为空。

<!--more-->

### 知识点讲解：

1. **有序数组**：二分查找算法的前提是数组必须是有序的，即元素是按照升序或降序排列的。

2. **查找过程**：算法会取数组的中间元素与目标值进行比较，如果相等，则查找成功；如果目标值小于中间元素，则在左半部分继续查找；如果目标值大于中间元素，则在右半部分继续查找。

3. **递归与迭代**：二分查找可以用递归或迭代的方式来实现。

4. **时间复杂度**：二分查找的时间复杂度是O(log n)，其中n是数组中元素的数量。这是因为每次比较都会将搜索范围缩小一半。

5. **边界条件**：在实现二分查找时，需要特别注意边界条件，比如数组为空、目标值在数组的两端等情况。

### 代码示例：

下面是一个使用迭代方式实现的二分查找算法的C++代码示例，以及详细的注释：

```cpp
// 二分查找的非递归实现
#include <iostream>
#include <vector>

using namespace std;

// 二分查找函数：在有序数组中查找 target，找到返回索引，找不到返回 -1
int binarySearch(const vector<int>& arr, int target) {
    int low = 0;              // 左边界初始化为 0
    int high = arr.size() - 1; // 右边界初始化为数组的最后一个索引

    while (low <= high) {
        // 计算中间索引，使用 (low + high) / 2 可能会溢出，因此用 low + (high - low) / 2
        int mid = low + (high - low) / 2;

        // 检查是否找到目标值
        if (arr[mid] == target) {
            return mid; // 返回目标值所在的索引
        } else if (arr[mid] > target) {
            high = mid - 1; // 目标值在左侧，更新右边界
        } else {
            low = mid + 1; // 目标值在右侧，更新左边界
        }
    }

    return -1; // 如果未找到，返回 -1
}

// 主函数，测试二分查找
int main() {
    vector<int> arr = {1, 3, 5, 7, 9, 11, 13, 15}; // 有序数组
    int target;

    cout << "请输入要查找的数字: ";
    cin >> target;

    int result = binarySearch(arr, target);

    if (result != -1) {
        cout << "找到目标值 " << target << "，索引为: " << result << endl;
    } else {
        cout << "未找到目标值 " << target << endl;
    }

    return 0;
}

```

```cpp
// 二分查找的递归实现
#include <iostream>
#include <vector>

using namespace std;

// 递归二分查找函数
int binarySearchRecursive(const vector<int>& arr, int low, int high, int target) {
    if (low > high) {
        return -1; // 基准情况：搜索范围为空，返回 -1
    }

    int mid = low + (high - low) / 2;

    if (arr[mid] == target) {
        return mid; // 找到目标值，返回索引
    } else if (arr[mid] > target) {
        return binarySearchRecursive(arr, low, mid - 1, target); // 目标值在左侧
    } else {
        return binarySearchRecursive(arr, mid + 1, high, target); // 目标值在右侧
    }
}

// 主函数，测试递归二分查找
int main() {
    vector<int> arr = {1, 3, 5, 7, 9, 11, 13, 15}; // 有序数组
    int target;

    cout << "请输入要查找的数字: ";
    cin >> target;

    int result = binarySearchRecursive(arr, 0, arr.size() - 1, target);

    if (result != -1) {
        cout << "找到目标值 " << target << "，索引为: " << result << endl;
    } else {
        cout << "未找到目标值 " << target << endl;
    }

    return 0;
}

```

### 注释解释：

- `#include <iostream>` 和 `#include <vector>`：引入标准输入输出流和向量容器的头文件。
- `binarySearch` 函数：定义了一个二分查找的函数，接受一个整数向量、目标值和数组大小作为参数。
- `left` 和 `right` 变量：分别代表搜索范围的左边界和右边界。
- `while` 循环：只要左边界不大于右边界，就继续查找。
- `mid` 变量：计算中间位置的索引，使用 `(left + right) / 2` 可能会造成整数溢出，因此使用 `left + (right - left) / 2`。
- `if-else if-else` 语句：根据中间元素与目标值的比较结果，调整搜索范围。
- `main` 函数：定义了一个有序数组、目标值和数组大小，调用 `binarySearch` 函数，并根据返回结果输出相应的信息。

希望这个详细的讲解和代码示例能帮助孩子们理解二分查找算法的工作原理和实现方式。

二分查找的基本原理和实现步骤如下：

### 基本原理：

1. **有序性**：二分查找算法只能在有序数组中进行，因为只有有序数组才能保证通过比较中间元素来缩小搜索范围。

2. **中间元素**：算法通过比较目标值与数组中间元素的大小，来决定是继续在数组的左半部分还是右半部分进行查找。

3. **缩小范围**：每次比较后，算法都会将搜索范围缩小一半，这是二分查找效率的关键。

4. **递归或迭代**：二分查找可以通过递归或迭代的方式来实现。

### 实现步骤：

#### 迭代实现步骤：

1. **初始化**：设置两个指针，一个指向数组的开始（`left`），另一个指向数组的结束（`right`）。

2. **循环条件**：当`left`小于或等于`right`时，继续循环。

3. **计算中间索引**：计算中间索引`mid`，通常使用`mid = left + (right - left) / 2`来避免整数溢出。

4. **比较中间元素**：将数组中间位置的元素与目标值进行比较。

5. **调整搜索范围**：
   - 如果中间元素等于目标值，查找成功，返回中间索引。
   - 如果目标值小于中间元素，将`right`更新为`mid - 1`，因为目标值必定在左半部分。
   - 如果目标值大于中间元素，将`left`更新为`mid + 1`，因为目标值必定在右半部分。

6. **循环结束**：如果循环结束还没有找到目标值，返回一个特定的值（通常是-1），表示目标值不在数组中。

#### 递归实现步骤：

1. **基本情况**：如果数组的长度为0，或者`left`大于`right`，则返回-1，表示没有找到。

2. **计算中间索引**：与迭代方法相同，计算中间索引`mid`。

3. **比较中间元素**：与迭代方法相同，比较中间元素与目标值。

4. **递归调用**：
   - 如果中间元素等于目标值，返回中间索引。
   - 如果目标值小于中间元素，递归调用左半部分（`left`到`mid - 1`）。
   - 如果目标值大于中间元素，递归调用右半部分（`mid + 1`到`right`）。

5. **返回结果**：递归调用会返回目标值的索引或者-1。

二分查找的效率之所以高，是因为它每次操作都能将搜索空间减半，因此其时间复杂度为O(log n)，这使得它在处理大数据集时非常有效。

二分查找和线性查找在效率上有显著的差异，主要体现在它们的时间复杂度上：


## 二分查找和线性查找相比

### 线性查找（Linear Search）：
- **时间复杂度**：O(n)，其中n是数组中元素的数量。
- **工作原理**：从数组的第一个元素开始，逐个检查每个元素，直到找到目标值或检查完所有元素。
- **效率**：在最坏的情况下，如果目标值位于数组的末尾或根本不存在，线性查找需要检查所有元素，因此需要n次比较。
- **适用场景**：适用于小型数据集或无序数组，因为对于小规模数据，其简单性和实现的便捷性可能超过二分查找的优势。

### 二分查找（Binary Search）：
- **时间复杂度**：O(log n)，其中n是数组中元素的数量。
- **工作原理**：在有序数组中，通过比较中间元素与目标值，每次比较后将搜索范围缩小一半。
- **效率**：在最坏的情况下，二分查找需要进行log₂n次比较，这里的“log”是以2为底的对数。这意味着即使对于较大的数据集，所需的比较次数也远远少于线性查找。
- **适用场景**：适用于大型且已排序的数据集，因为对于大规模数据，二分查找的效率优势非常明显。

### 效率对比：
- **大规模数据集**：对于大规模数据集，二分查找的效率远高于线性查找。例如，在一个包含1000个元素的数组中，二分查找最多需要10次比较（log₂1000 ≈ 10），而线性查找在最坏情况下需要1000次比较。
- **小规模数据集**：对于小规模数据集，线性查找可能更快，因为它避免了二分查找中计算中间索引的开销，尤其是当数组非常小的时候。

总结来说，二分查找在处理大型有序数据集时效率更高，而线性查找在处理小型数据集或无序数据集时可能更简单、更直接。选择哪种查找方法取决于具体的应用场景和数据特点。

### 二分查找在以下情况下更有优势

1. **有序数据集**：二分查找算法要求数据集必须是有序的，无论是升序还是降序。如果数据集是无序的，那么二分查找无法应用，或者需要先对数据进行排序，这会增加额外的时间成本。

2. **大数据集**：对于包含大量元素的数据集，二分查找的优势尤为明显。由于其时间复杂度为O(log n)，随着数据集规模的增加，二分查找相比于线性查找的效率优势更加显著。

3. **重复查找操作**：如果需要在同一个有序数组中进行多次查找操作，二分查找更加高效。因为一旦数组被排序，后续的查找操作可以重复利用这个有序结构，而不需要每次都从头开始。

4. **内存限制**：在内存受限的情况下，二分查找可以减少比较次数，从而减少内存访问，提高效率。

5. **实时系统**：在需要快速响应的实时系统中，二分查找可以提供更快的查找速度，因为它避免了线性查找中可能的大量元素比较。

6. **优化存储访问**：在数据库和文件系统中，存储访问往往是性能瓶颈。二分查找可以减少存储访问次数，因此在这些场景下更有优势。

7. **算法竞赛和面试**：在算法竞赛和编程面试中，二分查找是一个常用的技术，因为它可以解决一类特定的问题，并且具有很好的理论时间复杂度。

8. **数学和逻辑问题**：在解决一些需要快速定位和比较的问题时，如查找最大值、最小值或者某个特定值的范围时，二分查找可以提供清晰的逻辑框架。

总的来说，二分查找在数据量大、查找操作频繁、对性能要求高的场景下更有优势。然而，它也有一定的局限性，比如需要数据预先排序，以及不适用于无序数据集。在实际应用中，需要根据具体情况选择最合适的查找算法。

### 历年真题

根据搜索结果，以下是NOIP和CSP竞赛中涉及到二分查找算法的题目及详细解答：

### 1. CSP-J 2021 矩形计数
**题目描述**：
平面上有n个关键点，求有多少个四条边都和x轴或者y轴平行的矩形，满足四个顶点都是关键点。给出的关键点可能有重复，但完全重合的矩形只计一次。

**解题思路**：
- 首先对关键点按照x和y坐标进行排序。
- 去重，确保每个点只被计算一次。
- 枚举两个点作为矩形的两个对角，然后通过二分查找确定另外两个点是否存在。

**代码实现**：
```c
#include<stdio.h>
struct point{//结构体
    int x,y,id;
};
int equals(struct point a,struct point b){//比较结构体内变量x y是否相等
    return a.x==b.x && a.y==b.y;
}
int cmp(struct point a,struct point b){//排序sort使用 选择都是判断a<b 从小到大排序
    return a.x!=b.x?a.x<b.x:a.y<b.y;
}
void sort(struct point A[],int n){//按照排序规则 A[j]<a[j-1] 交换 说明可以从小到大排序
    for(int i=0;i<n;i++)
        for(int j=1;j<n;j++)
            if(cmp(A[j],A[j-1])){
                struct point t=A[j];
                A[j]=A[j-1];
                A[j-1]=t;
            }
}
int unique(struct point A[],int n){//去重
    int t=0;
    for(int i=0;i<n;i++)
        if(i==0||!equals(A[i],A[t-1]))
            A[t++]=A[i];
    return t;
}
int main(){
    int n,cnt=0;
    scanf("%d",&n);
    struct point points[1005];
    for(int i=0;i<n;i++){
        scanf("%d %d",&points[i].x,&points[i].y);
        points[i].id=i;
    }
    sort(points,n);
    n=unique(points,n);
    for(int i=0; i<n; i++){
        for(int j=i+1; j<n; j++){
            int x1=points[i].x, y1=points[i].y;
            int x2=points[j].x, y2=points[j].y;
            int x3=x1, y3=y2;
            int x4=x2, y4=y1;
            if(x3==x4 || y3==y4) continue;
            int a=lower_bound(points, points+n,(struct point){x3,y3}) - points;
            int b=lower_bound(points, points+n,(struct point){x4,y4}) - points;
            cnt+=a>=0 && a<n && b>=0 && b<n && !equals(points[a],points[b]);
        }
    }
    printf("%d\n", cnt);
    return 0;
}
```

### 2. P1314 [NOIP2011 提高组] 聪明的质监员
**题目描述**：
给定一系列矿石，每个矿石有重量和价值，以及一个总价值s。需要确定一个阈值W，使得大于W的矿石的总价值之和不超过s。

**解题思路**：
- 使用二分查找确定阈值W。
- 计算大于W的矿石的数量和价值之和。
- 如果总价值之和超过s，则需要增加W；如果小于等于s，则尝试减少W。

**代码实现**：
```c
#include<bits/stdc++.h>
typedef long long ll;
using namespace std;
const int N=200010;
ll w[N],v[N];
ll L[N],R[N];
ll n,m,s;
ll pre[N],vpre[N];
ll check(int x){
    ll y=0;
    memset(pre,0,sizeof pre);
    memset(vpre,0,sizeof vpre);
    for(int i=1;i<=n;i++){
        if(w[i]>=x){
            pre[i]=pre[i-1]+1;
            vpre[i]=vpre[i-1]+v[i];
        }else {
            pre[i]=pre[i-1];
            vpre[i]=vpre[i-1];
        }
    }
    for(int i=1;i<=m;i++){
        y+=(pre[R[i]]-pre[L[i]]-1)* (vpre[R[i]]-vpre[L[i]-1]);
    }
    return y;
}
int main(){
    cin>>n>>m>>s;
    for (int i = 1; i <= n; i++) cin>>w[i]>>v[i];
    for (int i = 1; i <=m; i++) cin>>L[i]>>R[i];
    int l=0,r=1e6+1;
    while (l < r){
        int mid = (l + r + 1) >> 1;
        if (check(mid)>=s) l = mid;
        else r = mid - 1;
    }
    ll ans=min(abs(check(r)-s),abs(check(r+1)-s));
    cout<<ans;
    return 0;
}
```
