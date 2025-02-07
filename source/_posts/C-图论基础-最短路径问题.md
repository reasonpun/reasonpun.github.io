---
title: C++图论基础 - 最短路径问题
date: 2025-02-07 17:19:07
categories:
- 算法
tags:
- 算法 最短路径
---

## 课程目标

- 了解最短路径问题的基本概念。
- 掌握最常用的最短路径算法：Dijkstra算法、Bellman-Ford算法和Floyd-Warshall算法。
- 理解每种算法的应用场景、优缺点及其实现方式。
- 通过代码示例帮助学生深入理解算法的实际应用。

<!--more-->

## 一、最短路径问题概述

### 1.1 什么是最短路径问题？
在图论中，最短路径问题是指：在给定图中的某一个源点出发，找到到其他各个顶点的最短路径长度。在加权图中，每一条边都有一个权重，表示从一个顶点到另一个顶点的代价。

### 1.2 最短路径问题的实际应用
- **GPS导航**：计算最短的驾驶路径。
- **网络路由**：选择网络中最优的传输路径。
- **城市规划**：最短通路规划。


## 二、常见的最短路径算法

### 2.1 Dijkstra算法

#### 2.1.1 算法简介
Dijkstra算法用于在加权图中，找到从一个源点到所有其他顶点的最短路径。该算法基于贪心算法，每次选择当前距离最短的顶点，更新它的邻接顶点的最短路径。

#### 2.1.2 算法步骤：
1. 初始化：设定源点到所有其他顶点的距离为∞，源点到自己的距离为0。
2. 每次从未访问的顶点中选择一个距离源点最近的顶点，标记为已访问。
3. 更新该顶点的邻接点的距离：如果通过该顶点到达某个邻接点的距离比原来的距离更短，则更新距离。
4. 重复2-3步骤，直到所有顶点都被访问过。

#### 2.1.3 适用场景
适用于没有负权边的图。  
图中的边权都为正值。

#### 2.1.4 C++代码实现
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <climits>
using namespace std;

#define INF INT_MAX
typedef pair<int, int> pii;                                     /* 用于表示 {权重, 顶点} */

/* Dijkstra 算法 */
void dijkstra(int n, vector<vector<pii>>& adj, int source) {
    vector<int> dist(n, INF);                                   /* 存储最短路径 */
    dist[source] = 0;                                           /* 源点到自己是0 */
    priority_queue<pii, vector<pii>, greater<pii>> pq;          /* 最小堆（优先队列） */
    pq.push({0, source});
    while (!pq.empty()) {
        int u = pq.top().second;                                /* 当前顶点 */
        int d = pq.top().first;                                 /* 当前顶点的最短路径 */
        pq.pop();
        if (d > dist[u]) continue;                              /* 如果当前路径不是最短的，跳过 */
        for (auto& edge : adj[u]) {                             /* 遍历所有邻接点 */
            int v = edge.first;
            int weight = edge.second;
            if (dist[u] + weight < dist[v]) {                   /* 发现更短路径 */
                dist[v] = dist[u] + weight;
                pq.push({dist[v], v});
            }
        }
    }

    /* 输出最短路径 */
    for (int i = 0; i < n; ++i) {
        if (dist[i] == INF) {
            cout << "从源点到顶点 " << i << " 没有路径\n";
        } else {
            cout << "源点到顶点 " << i << " 的最短路径为: " << dist[i] << "\n";
        }
    }
}

int main() {
    int n = 5;                                                  /* 图中有5个顶点 */
    vector<vector<pii>> adj(n);                                 /* 邻接表表示图 */
    /* 构建图，使用邻接表表示 */
    adj[0].push_back({1, 10});
    adj[0].push_back({2, 5});
    adj[1].push_back({2, 2});
    adj[1].push_back({3, 1});
    adj[2].push_back({1, 3});
    adj[2].push_back({3, 9});
    adj[3].push_back({4, 4});
    adj[4].push_back({0, 7});
    int source = 0;
    dijkstra(n, adj, source);
    return 0;
}
```

**代码解释**：
- **邻接表**：`adj` 用来存储图中的边和权重。
- **优先队列**：用来存储当前访问的节点及其最短路径，确保每次都能选择到当前最短的节点进行更新。
- **时间复杂度**：O((V+E)logV)，其中 V 是顶点数，E 是边数。

**示例输出**：
```
源点到顶点 0 的最短路径为: 0
源点到顶点 1 的最短路径为: 8
源点到顶点 2 的最短路径为: 5
源点到顶点 3 的最短路径为: 9
源点到顶点 4 的最短路径为: 13
```


### 2.2 Bellman-Ford算法

#### 2.2.1 算法简介
Bellman-Ford算法是一种用于求解单源最短路径的算法，能够处理带负权边的图。与Dijkstra算法不同，Bellman-Ford算法可以检测图中是否有负权环。

#### 2.2.2 算法步骤：
1. 初始化：源点到自己的距离为0，其他点为∞。
2. 对所有的边进行 V−1 次松弛操作，每次更新通过该边可以得到的最短路径。
3. 再进行一次松弛操作，如果仍然可以更新，说明图中存在负权环。

#### 2.2.3 适用场景
适用于带负权边的图。  
可以检测负权环。

#### 2.2.4 C++代码实现
```cpp
#include <iostream>
#include <vector>
#include <climits>
using namespace std;

#define INF INT_MAX

// Bellman-Ford 算法
bool bellmanFord(int n, vector<vector<pair<int, int>>>& adj, int source) {
    vector<int> dist(n, INF);
    dist[source] = 0;

    // V-1 次松弛操作
    for (int i = 1; i < n; ++i) {
        for (int u = 0; u < n; ++u) {
            for (auto& edge : adj[u]) {
                int v = edge.first;
                int weight = edge.second;
                if (dist[u] != INF && dist[u] + weight < dist[v]) {
                    dist[v] = dist[u] + weight;
                }
            }
        }
    }

    // 检查是否有负权环
    for (int u = 0; u < n; ++u) {
        for (auto& edge : adj[u]) {
            int v = edge.first;
            int weight = edge.second;
            if (dist[u] != INF && dist[u] + weight < dist[v]) {
                cout << "图中存在负权环\n";
                return false;
            }
        }
    }

    // 输出最短路径
    for (int i = 0; i < n; ++i) {
        if (dist[i] == INF) {
            cout << "从源点到顶点 " << i << " 没有路径\n";
        } else {
            cout << "源点到顶点 " << i << " 的最短路径为: " << dist[i] << "\n";
        }
    }
    return true;
}

int main() {
    int n = 5;
    vector<vector<pair<int, int>>> adj(n);
    // 构建图，包含负权边
    adj[0].push_back({1, -1});
    adj[0].push_back({2, 4});
    adj[1].push_back({2, 3});
    adj[1].push_back({3, 2});
    adj[1].push_back({4, 2});
    adj[2].push_back({3, 5});
    adj[3].push_back({1, 1});
    adj[4].push_back({2, 5});
    int source = 0;
    bellmanFord(n, adj, source);
    return 0;
}
```

**代码解释**：
- **负权边的处理**：通过 V−1 次松弛操作，确保所有边都能被更新。
- **负权环检测**：在最后一次松

弛操作中，如果仍然能更新路径，则说明图中存在负权环。

**示例输出**：
```
源点到顶点 0 的最短路径为: 0
源点到顶点 1 的最短路径为: -1
源点到顶点 2 的最短路径为: 2
源点到顶点 3 的最短路径为: 3
源点到顶点 4 的最短路径为: 1
```

## 三、总结

### 3.1 Dijkstra与Bellman-Ford比较
- **时间复杂度**：Dijkstra算法O((V+E)logV)，适用于无负权边的图。Bellman-Ford算法O(VE)，可以处理负权边和负权环。
- **适用范围**：Dijkstra适合边权为正的图，而Bellman-Ford适合有负权边的图，且能检测负权环。

### 3.2 Floyd-Warshall算法
Floyd-Warshall算法是求解所有顶点对最短路径的算法，适用于较小规模的图，时间复杂度为O(V^3)，但其空间复杂度较高。

## 四、课堂练习
### 4.1 编写代码，使用Dijkstra算法求解给定图的最短路径。
### 4.2 使用Bellman-Ford算法检测给定图中是否存在负权环。
### 4.3 修改代码，实现Floyd-Warshall算法，计算图中所有顶点对的最短路径。
