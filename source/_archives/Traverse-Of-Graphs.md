---
title: 图的遍历
date: 2019-05-10 22:54:19
categories:
- C++
- Algorithm
    - Graph Theory
tags:
- DFS
- BFS
---

## 图的深度优先遍历

### 基本思想

1. 给定G=<V,E>，假定初始状态为G中所有顶点均未被访问，以一个未被访问过的顶点为起始点，依次访问各个未被访问的邻接点，直到G中所有和顶点v连通的顶点都被访问。
2. 若经过(1)的操作后G中尚有其他顶点未被访问，则另选一个未被访问的顶点作为起始点，重复步骤(1)，直到G中所有顶点均被访问。
<!--more-->

### 应用

#### 无向图找环

[src: codeforces](http://codeforces.com/contest/263/problem/D)

**题意**：给定无向图G，G中每个顶点度数不小于k。要你找到一个长度大于k的环(保证答案存在)。

<img src="Traverse-Of-Graphs/traverse_of_graphs.png">

**题解**：考虑深度优先遍历。遍历过程中，对于一条边u->v:

1. vis[v] = 0 表示v未被访问，u->v是一条树边
2. vis[v] = 1 表示v已被访问，但其子孙未被访问完，u->v是一条后向边(返祖边)
3. vis[v] = 2 表示v已被访问，且其子孙已被访问完，u->v是一条前向边或者横叉边

**代码实现**：
```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 100001;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    int minDegree;
    vector<int> edge[N];
    vector<int> path;
    int dis[N];
    bool vis[N];
    bool flag;

public:
    Graph(int n, int m, int k) : vertexNum(n), edgeNum(m), minDegree(k) {}
    void init();
    void addEdge();
    void findCircle(int, int);
};

void Graph::init()
{
    flag = false;
    for(int i = 1 ; i <= vertexNum ;i++){
        vis[i] = false;
        dis[i] = 0;
    }
}

void Graph::addEdge()
{
    for (int i = 1; i <= edgeNum; i++) {
        int u, v;
        cin >> u >> v;
        edge[u].push_back(v);
        edge[v].push_back(u);
    }
}

void Graph::findCircle(int u = 1, int d = 0)
{
    if (vis[u] == 1)
        return;
    vis[u] = true;
    dis[u] = d;
    path.push_back(u);
    for (auto v : edge[u]) {
        if (flag)
            return;
        if (vis[v] == true && d - dis[v] >= minDegree) {
            int pos = 0;
            while (path[pos] != v) {
                pos++;
            }
            cout << d - dis[v] + 1 << endl;
            for (int i = pos; i <= d; i++) {
                cout << path[i] << ' ';
            }
            flag = true;
        } else {
            findCircle(v, d + 1);
        }
    }
    vis[u] = 2;
    path.pop_back();
}

int main()
{
    int n, m, k;
    cin >> n >> m >> k;
    Graph G(n, m, k);
    G.init();
    G.addEdge();
    G.findCircle();
    return 0;
}
```

#### 无向图的k着色问题

[src: luogu](https://www.luogu.org/problemnew/show/P2819)

**题意**：给定无向图G和颜色种类k。问你用k种颜色对G中所有顶点着色，使得每条边连接的两个顶点着色不同的方案数。

**题解**：考虑深度优先遍历。遍历所有顶点，着色后判重，若重复则回溯，否则继续。

**实现代码**：
```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 101;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    int colorNum;
    int matrix[N][N];
    int color[N];
    int sum;

public:
    Graph(int n, int m, int k) : vertexNum(n), edgeNum(m), colorNum(k) { sum = 0; }
    void addEdge();
    void kColoring(int);
    bool checkColor(int);
    int getSum() { return sum; }
};

void Graph::addEdge()
{
    for (int i = 1; i <= edgeNum; i++) {
        int u, v;
        cin >> u >> v;
        matrix[u][v] = 1;
        matrix[v][u] = 1;
    }
}

bool Graph::checkColor(int i)
{
    for (int j = 1; j < i; j++) {
        if (matrix[i][j] == 1 && color[i] == color[j])
            return false;
    }
    return true;
}

void Graph::kColoring(int i = 1)
{
    if (i > vertexNum) {
        sum++;
        /*for (int j = 1; j <= vertexNum; j++) {
            cout << color[j] << ' ';
        }
        cout << endl;*/
        return;
    } else {
        for (int j = 1; j <= colorNum; j++) {
            color[i] = j;
            if (checkColor(i)) {
                kColoring(i + 1);
            }
    
        }
    }
}

int main()
{
    int n, m, k;
    cin >> n >> m >> k;
    Graph G(n, m, k);
    G.addEdge();
    G.kColoring();
    cout << G.getSum() << endl; 
    return 0;
}
```

## 图的广度优先遍历

### 基本思想

给定G=<V,E>，假定初始状态为G中所有顶点均未被访问，以一个未被访问过的顶点为起始点，分别访问各个邻接点的邻接点，直到G中所有和顶点v连通的顶点都被访问。

### 应用

#### 无向图找最小字典序

[src: codeforces](https://codeforces.com/contest/1106/problem/D)

**题意**：给定无向图G，G中可能存在平行边或者自环。要你找到遍历所有顶点的最小字典序。

**题解**：考虑广度优先遍历。直接使用最小堆，利用标记数组判重即可。

**代码实现**：
```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 100001;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    vector<int> edge[N];
    bool vis[N];

public:
    Graph(int n, int m) : vertexNum(n), edgeNum(m) {}
    void init();
    void addEdge();
    void findMinDicOrd();
};

void Graph::init()
{
    for (int i = 1; i <= vertexNum; i++)
    {
        vis[i] = false;
    }
}

void Graph::addEdge()
{
    for (int i = 1; i <= edgeNum; i++)
    {
        int u, v;
        cin >> u >> v;
        edge[u].push_back(v);
        edge[v].push_back(u);
    }
}

void Graph::findMinDicOrd()
{
    priority_queue<int, vector<int>, greater<int>> p;
    p.push(1);
    while (!p.empty())
    {
        int u = p.top();
        p.pop();
        if (vis[u])
            continue;
        vis[u] = true;
        cout << u << " ";
        for (auto v : edge[u])
        {
            if (!vis[v])
            {
                p.push(v);
            }
        }
    }
}

int main()
{
    int n, m;
    cin >> n >> m;
    Graph G(n, m);
    G.addEdge();
    G.init();
    G.findMinDicOrd();
    return 0;
}
```