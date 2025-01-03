---
title: C++Dijkstra算法
date: 2025-01-03 17:00:35
categories:
- 算法
tags:
- 算法 Dijkstra算法
---

# 📚 **Dijkstra 算法详解（C++）**

## **1. 什么是 Dijkstra 算法？**

**Dijkstra 算法**是一种用于**计算单源最短路径**的经典算法。它可以在**加权有向图或无向图**中找到**从起点到其他所有节点的最短路径**。

<!--more-->

### **1.1 适用范围**
- 图中**没有负权边**（有负权边时应使用 **Bellman-Ford 算法**）。  
- 可以处理**稠密图**（邻接矩阵）和**稀疏图**（邻接表）。  

### **1.2 基本思想**
- 使用**贪心策略**：每次选择**当前距离起点最近的节点**，并更新其相邻节点的最短路径。  
- 使用一个**优先队列**（小顶堆）来加速找到当前最短路径的节点。


## **2. Dijkstra 算法步骤**

### **2.1 初始化**

1. 设定一个**源点**（起始节点）。  
2. 将源点到自身的距离设为 `0`，其他所有节点的距离设为 **无穷大（∞）**。  
3. 使用**优先队列**（小顶堆）存储当前的节点和最短距离。  

### **2.2 过程**

1. 从优先队列中取出**当前距离最小的节点**。  
2. 遍历该节点的**所有相邻节点**，更新相邻节点的**最短路径**：  
   ```
   d[v] = min(d[v], d[u] + w(u, v))
   ```
   其中：
   - \( d[v] \)：当前到节点 \( v \) 的最短距离。  
   - \( d[u] \)：起始点到当前节点 \( u \) 的最短距离。  
   - \( w(u, v) \)：从节点 \( u \) 到 \( v \) 的边的权重。  
3. 将更新后的节点重新加入**优先队列**。  
4. 重复上述步骤，直到优先队列为空。  

### **2.3 结束**

- 当所有节点都被访问过后，算法结束。  
- 结果是**源点到其他所有节点的最短路径**。  


## **3. C++ 实现 Dijkstra 算法**

### **3.1 邻接表表示的 Dijkstra 算法**

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <climits>

using namespace std;

// 定义邻接表中的边
struct Edge {
    int to;     // 目标节点
    int weight; // 边的权重
};

// Dijkstra 算法函数
void dijkstra(int start, const vector<vector<Edge>>& graph) {
    int n = graph.size();
    vector<int> dist(n, INT_MAX); // 存储从起点到每个节点的最短距离
    vector<bool> visited(n, false); // 标记节点是否已经被访问

    // 小顶堆：按最短距离排序
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;

    // 起点到自身的距离为 0
    dist[start] = 0;
    pq.push({0, start});

    while (!pq.empty()) {
        // 取出当前距离最小的节点
        pair<int, int> current = pq.top();
        pq.pop();
        int currentDist = current.first; // 当前节点的最短距离
        int u = current.second;          // 当前节点的编号

        // 如果当前节点已访问，则跳过
        if (visited[u]) continue;
        visited[u] = true;

        // 遍历当前节点的所有邻接边
        for (const Edge& edge : graph[u]) {
            int v = edge.to;
            int weight = edge.weight;

            // 更新最短路径
            if (!visited[v] && dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
                pq.push({dist[v], v});
            }
        }
    }

    // 输出结果
    cout << "Node\tShortest Distance from Source" << endl;
    for (int i = 0; i < n; ++i) {
        cout << i << "\t" << dist[i] << endl;
    }
}

// 测试
int main() {
    int n = 5; // 节点数
    vector<vector<Edge>> graph(n);

    // 添加边 (节点编号从 0 开始)
    graph[0].push_back({1, 10});
    graph[0].push_back({4, 5});
    graph[1].push_back({2, 1});
    graph[1].push_back({4, 2});
    graph[2].push_back({3, 4});
    graph[3].push_back({0, 7});
    graph[3].push_back({2, 6});
    graph[4].push_back({1, 3});
    graph[4].push_back({2, 9});
    graph[4].push_back({3, 2});

    // 源点为节点 0
    dijkstra(0, graph);

    return 0;
}

```


## **4. 代码解析**

### **4.1 数据结构**
- `vector<vector<Edge>> graph`：邻接表存储图结构。  
- `vector<int> dist`：存储从起点到各个节点的最短距离。  
- `priority_queue`：存储当前节点及其最短距离，使用**小顶堆**确保每次访问的是**当前距离最小的节点**。

### **4.2 算法流程**
1. 初始化所有节点的最短距离为 `INT_MAX`，源点距离为 `0`。  
2. 使用小顶堆维护每次要访问的**最短距离节点**。  
3. 访问节点并更新其相邻节点的最短距离。  
4. 如果有更短的路径，将节点重新加入到优先队列中。  


## **5. 时间与空间复杂度**

- **时间复杂度**：\( O((V + E) \cdot \log V) \)  
  - \( V \)：节点数  
  - \( E \)：边数  
  - 每次从优先队列中取出节点需要 \( \log V \)，遍历所有边共计 \( O(E) \)。  
- **空间复杂度**：\( O(V + E) \)  
  - 存储邻接表和优先队列所需的空间。

## **6. Dijkstra 算法的应用**

1. **地图导航**：计算最短路径。  
2. **网络路由**：最短路径选择。  
3. **资源分配**：在有限资源下的最优分配路径。  
4. **游戏开发**：角色移动路径规划。  


## **7. 注意事项**

- **不能处理负权边**：负权边会导致路径更新出错。  
- **多源最短路径**：需要对每个源点分别执行 Dijkstra 算法，或者使用 **Floyd-Warshall 算法**。  


## **8. 总结**

- **核心思想**：每次选择当前未访问节点中**距离源点最近的节点**，并更新其相邻节点的最短路径。  
- **关键数据结构**：**优先队列**（小顶堆）。  
- **限制**：不能有负权边。  