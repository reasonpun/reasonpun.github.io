---
title: C++图论基础-概念和遍历
date: 2025-01-17 15:52:23
categories:
- 算法
tags:
- 算法 图论
---

### 1. 图的定义和基本概念

在图论中，图（Graph）是由顶点（Vertex）和边（Edge）构成的集合。我们可以通过图来表示各种关系，比如社交网络中的人际关系、城市交通中的路线等。图的基本元素包括顶点和边。

<!--more-->

#### 1.1 图的基本组成
- **顶点（Vertex）：** 图中的基本单位，表示对象或元素。例如，社交网络中的“人”、城市图中的“城市”等。
- **边（Edge）：** 连接两个顶点的线，表示这些顶点之间的关系。例如，社交网络中的“朋友关系”、城市图中的“公路”。
  
图可以用不同的方式分类，主要包括以下几类：
- **有向图（Directed Graph）：** 边有方向。即边从一个顶点指向另一个顶点。例如，一个网页的链接。
- **无向图（Undirected Graph）：** 边没有方向，表示两个顶点之间的双向关系。例如，社交网络中的朋友关系。
- **加权图（Weighted Graph）：** 边有权重，表示边的“大小”或“成本”，例如地图中的路程长度。
- **非加权图（Unweighted Graph）：** 边没有权重，边只表示是否存在连接。
- **简单图（Simple Graph）：** 不包含自环和多重边，即每对顶点之间至多有一条边。

### 2. 图的存储结构

在计算机中，图的存储方式有多种，最常用的存储方式是 **邻接矩阵** 和 **邻接表**。

#### 2.1 邻接矩阵（Adjacency Matrix）
邻接矩阵是一个二维数组，表示图中顶点之间的连接关系。如果图有 `n` 个顶点，那么邻接矩阵是一个 `n x n` 的矩阵。矩阵中的元素表示顶点之间是否有边，如果存在边，则值为 `1` 或 `权重`；如果不存在边，则值为 `0` 或无穷大。

- **优点**：
  - 判断两顶点之间是否有边的时间复杂度是 O(1)。
  - 实现简单，适用于边数较多的图。
- **缺点**：
  - 空间复杂度为 O(n^2)，对于稀疏图，空间浪费较大。
  - 边的遍历时间复杂度较高，遍历所有边的时间是 O(n^2)。

#### 2.2 邻接表（Adjacency List）
邻接表使用一个数组来存储每个顶点的邻接点，通常使用链表或动态数组来存储每个顶点的邻接点。这种存储方式适合稀疏图。

- **优点**：
  - 空间效率高，适合稀疏图，空间复杂度为 O(n + m)，其中 `n` 是顶点数，`m` 是边数。
  - 遍历顶点的所有邻接点时间复杂度为 O(d)，其中 d 是顶点的度。
- **缺点**：
  - 判断两顶点之间是否有边的时间复杂度是 O(d)，其中 d 是顶点的度。

### 3. 数组模拟临界表存储

在图的实现中，通常用邻接表来存储图，因为它比邻接矩阵更节省空间。数组模拟邻接表存储时，我们用一个数组，每个数组的元素保存一个链表，链表中的节点表示与该顶点相连的其他顶点。

#### 3.1 实现思路
1. 定义一个结构体 `Edge`，表示图中的边，存储边的终点。
2. 用一个 `vector<vector<int>>` 或 `vector<list<int>>` 来表示邻接表。

#### 3.2 示例代码
以下是使用邻接表来存储图并进行边的添加的代码：

```cpp
#include <iostream>
#include <vector>
#include <list>
using namespace std;

const int MAXN = 100;  // 顶点最大数
vector<int> adjList[MAXN];  // 邻接表

// 添加一条边
void addEdge(int u, int v) {
    adjList[u].push_back(v);  // 有向图，u -> v
    adjList[v].push_back(u);  // 无向图，需要反向添加
}

int main() {
    int n, m;  // 顶点数n和边数m
    cin >> n >> m;

    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        addEdge(u, v);
    }

    // 输出邻接表
    for (int i = 1; i <= n; i++) {
        cout << "Vertex " << i << ": ";
        for (int v : adjList[i]) {
            cout << v << " ";
        }
        cout << endl;
    }

    return 0;
}
```

### 4. 图的遍历

图的遍历算法是图论中的一个基础问题，常见的遍历算法有 **深度优先搜索（DFS）** 和 **广度优先搜索（BFS）**。这两种算法分别使用递归和队列来遍历图中的顶点。

#### 4.1 深度优先搜索（DFS）

**DFS** 是一种深度优先的遍历方式，它会从一个起始点出发，沿着一条路径一直深入直到没有未访问的邻接点，再回溯到上一个顶点。DFS 的特点是通过递归或栈来实现。

- **时间复杂度：** O(V + E)，其中 V 是顶点数，E 是边数。
- **空间复杂度：** O(V)，栈空间最多存储 V 个顶点。

```cpp
#include <iostream>
#include <vector>
using namespace std;

vector<int> adjList[MAXN];  // 邻接表
bool visited[MAXN];  // 记录访问状态

// 深度优先搜索
void DFS(int u) {
    visited[u] = true;
    cout << u << " ";  // 输出当前访问的顶点

    // 递归访问与u相连的所有未访问的邻接点
    for (int v : adjList[u]) {
        if (!visited[v]) {
            DFS(v);
        }
    }
}

int main() {
    int n, m;
    cin >> n >> m;

    // 构建邻接表
    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        adjList[u].push_back(v);
        adjList[v].push_back(u);  // 无向图
    }

    // 从节点1开始DFS遍历
    DFS(1);

    return 0;
}
```

#### 4.2 广度优先搜索（BFS）

**BFS** 是一种层次优先的遍历方式，它会从起始点出发，首先访问所有邻接点，然后再访问每个邻接点的邻接点，以此类推。BFS 使用队列来实现。

- **时间复杂度：** O(V + E)，其中 V 是顶点数，E 是边数。
- **空间复杂度：** O(V)，队列最多存储 V 个顶点。

```cpp
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

vector<int> adjList[MAXN];  // 邻接表
bool visited[MAXN];  // 记录访问状态

// 广度优先搜索
void BFS(int start) {
    queue<int> q;
    visited[start] = true;
    q.push(start);

    while (!q.empty()) {
        int u = q.front();
        q.pop();
        cout << u << " ";  // 输出当前访问的顶点

        // 访问与u相连的所有未访问的邻接点
        for (int v : adjList[u]) {
            if (!visited[v]) {
                visited[v] = true;
                q.push(v);
            }
        }
    }
}

int main() {
    int n, m;
    cin >> n >> m;

    // 构建邻接表
    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        adjList[u].push_back(v);
        adjList[v].push_back(u);  // 无向图
    }

    // 从节点1开始BFS遍历
    BFS(1);

    return 0;
}
```

### 总结

- **图的定义**：图是由顶点和边组成的数据结构。图可以是有向的、无向的、加权的等。
- **图的存储结构**：主要有邻接矩阵和邻接表两种。邻接矩阵适用于边多的图，而邻接表适用于稀疏图。
- **数组模拟邻接表**：使用 `vector` 或 `list` 来存储每个顶点的邻接点。
-

 **图的遍历**：
  - **深度优先搜索（DFS）**：利用递归或者栈的方式从一个顶点出发，尽可能深入地访问每个顶点。
  - **广度优先搜索（BFS）**：利用队列的方式，从起始顶点开始，层层展开地访问图中的顶点。

这些基础知识和代码示例构成了理解和实现图论相关问题的基础。在实际应用中，掌握这些算法和数据结构将帮助你解决许多图相关的计算机问题。