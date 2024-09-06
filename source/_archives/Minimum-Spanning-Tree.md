---
title: 最小生成树
date: 2019-05-17 23:19:34
categories:
- C++
- Algorithm
    - Graph Theory
tags:
- Kruskal
- Prim
---

## Kruskal算法

### 基本思想

基于贪心思想，借助并查集实现。

1. 给定G=<V,E,W>，顶点个数为N
2. 引入集合T，将G中非环边按权从小到大排序，取边e1加入T中
3. 检查e2与e1是否构成回路，若构成回路则舍弃，否则将e2也加入T中
4. 重复步骤(3)，直到生成树的边为N-1
<!--more-->

### 应用场景

用于稀疏图找最小生成树。

[src: POJ](http://poj.org/problem?id=1287)

**题意**：给定无向带权图G，G中可能存在平行边。问你最小生成树所有边的权值和。

**题解**：Kruskal算法模板题，注意平行边可能权值相同，不要使用set。

**实现代码**：
```cpp
// #include <bits/stdc++.h>
#include <iostream>
#include <set>
using namespace std;
const int N = 51;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    struct node {
        int u;
        int v;
        int w;
        node(int u, int v, int w) : u(u), v(v), w(w) {}
        node();
        friend bool operator<(node a, node b) { return a.w < b.w; }
    };
    multiset<node> edge;
    int pre[N];

public:
    Graph(int n, int m) : vertexNum(n), edgeNum(m) {}
    void init();
    int find(int);
    bool unite(int, int);
    void addEdge();
    int Kruskal();
};

void Graph::init()
{
    for (int i = 1; i <= vertexNum; i++) {
        pre[i] = i;
    }
}

int Graph::find(int x)
{
    return x == pre[x] ? x : pre[x] = find(pre[x]);
}

bool Graph::unite(int x, int y)
{
    int fx = find(x);
    int fy = find(y);
    if (fx != fy) {
        pre[fx] = pre[fy];
        return true;
    }
    return false;
}

void Graph::addEdge()
{
    for (int i = 1; i <= edgeNum; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        edge.insert(node(u, v, w));
    }
}

int Graph::Kruskal()
{
    int minWeight = 0;
    init();
    for (multiset<node>::iterator it = edge.begin(); it != edge.end(); it++ /*auto e : edge*/) {
        node e = *it;
        if (unite(e.u, e.v)) {
            minWeight += e.w;
        }
    }
    return minWeight;
}

int main()
{
    int n, m;
    while (cin >> n && n) {
        cin >> m;
        Graph G(n, m);
        G.addEdge();
        cout << G.Kruskal() << endl;
    }
    return 0;
}
```

## Prime算法

### 基本思想

基于贪心思想，与Dijkstra算法类似。

1. 给定G=<V,E,W>，顶点个数为N
2. 引入集合U，从顶点集V中任选一点u作为U中的元素
3. 从顶点u的邻接点中任选一点v使这两点之间权值最小且不构成回路，将v加入U中
4. 从顶点v出发，重复步骤(3)，直到U中元素个数为N

### 应用场景

用于稠密图找最小生成树。

[src: POJ](http://poj.org/problem?id=1287)

**题意**：给定无向带权图G，G中可能存在平行边。问你最小生成树所有边的权值和。

**题解**：Prime算法模板题，注意平行边可能权值相同，任选其一即可。

**代码实现**：
```cpp
//#include <bits/stdc++.h>
#include <iostream>
using namespace std;
const int INF = 0x3f3f3f3f;
const int N = 51;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    int matrix[N][N];

public:
    Graph(int n, int m) : vertexNum(n), edgeNum(m) {}
    void init();
    void addEdge();
    int Prim();
};

void Graph::init()
{
    for (int i = 1; i <= vertexNum; i++) {
        for (int j = 1; j <= vertexNum; j++) {
            matrix[i][j] = INF;
        }
    }
}

void Graph::addEdge()
{
    for (int i = 1; i <= edgeNum; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        if (matrix[u][v] > w) {
            matrix[u][v] = w;
            matrix[v][u] = w;
        }
    }
}

int Graph::Prim()
{
    bool vis[N];
    int dis[N];

    for (int i = 1; i <= vertexNum; i++) {
        vis[i] = false;
        dis[i] = matrix[1][i];
    }
    vis[1] = true;
    dis[1] = INF;

    int minWeight = 0;
    for (int i = 1; i <= vertexNum; i++) {
        int k = 1;
        for (int j = 1; j <= vertexNum; j++) {
            if (!vis[j] && dis[j] < dis[k]) {
                k = j;
            }
        }
        if (k == 1)
            break;
        vis[k] = true;
        minWeight += dis[k];
        for (int j = 1; j <= vertexNum; j++) {
            if (!vis[j] && dis[j] > matrix[k][j]) {
                dis[j] = matrix[k][j];
            }
        }
    }
    return minWeight;
}

int main()
{
    int n, m;
    while (cin >> n && n) {
        cin >> m;
        Graph G(n, m);
        G.init();
        G.addEdge();
        cout << G.Prim() << endl;
    }
    return 0;
}
```
