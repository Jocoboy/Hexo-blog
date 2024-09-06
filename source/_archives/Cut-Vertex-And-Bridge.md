---
title: 无向图的割点与桥
date: 2019-06-22 23:41:37
categories:
- C++
- Algorithm
    - Graph Theory
tags:
- Tarjan
---

## 无向图求割点

### 基本概念

- 割点：无向连通图中，如果删除某点后，图的连通性被破坏，则称该点为割点

### Tarjan算法应用

[src: luogu](https://www.luogu.org/problemnew/show/P3388)

**题意**：给定无向图G。求总割点数，并按字典序输出所有割点。

**题解**：利用Tarjan算法思想。
1. 当前节点为树根时，如果子树不止一棵，则该点为割点
2. 当前节点不是树根时，如果无向边(u, v)为树边(父子边)且low[v]>=dfn[u] (不成环)，则该点为割点
<!--more-->

**实现代码**：
```cpp
// #include <bits\stdc++.h>
#include <iostream>
using namespace std;
const int N = 20001;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    struct node
    {
        int to;
        int next;
    };
    node edge[N * 10];
    int head[N];

    int tot;
    int low[N];
    int dfn[N];
    bool vis[N];

    int cutPointNum;
    bool cutPoint[N];

public:
    Graph(int n, int m) : vertexNum(n), edgeNum(m) {}
    void init();
    void addEdge();
    void findSCC();
    void Tarjan(int, int);
    void printCutPoint();
};

void Graph::init()
{
    tot = 0;
    cutPointNum = 0;
    for (int i = 1; i <= vertexNum; i++)
    {
        head[i] = -1;
        low[i] = 0;
        dfn[i] = 0;
        vis[i] = 0;
        cutPoint[i] = false;
    }
}

void Graph::addEdge()
{
    for (int i = 1; i <= edgeNum * 2; i++)
    {
        int u, v;
        cin >> u >> v;
        edge[i].to = v;
        edge[i].next = head[u];
        head[u] = i;

        i++;
        edge[i].to = u;
        edge[i].next = head[v];
        head[v] = i;
    }
}

void Graph::findSCC()
{
    for (int i = 1; i <= vertexNum; i++)
    {
        if (!dfn[i])
        {
            Tarjan(i, i);
        }
    }
}

void Graph::Tarjan(int u, int root)
{
    int children = 0;
    low[u] = dfn[u] = ++tot;
    vis[u] = true;
    for (int i = head[u]; i != -1; i = edge[i].next)
    {
        int v = edge[i].to;
        if (!dfn[v])
        {
            Tarjan(v, root);
            low[u] = min(low[u], low[v]);
            if (u != root && low[v] >= dfn[u])
            {
                cutPoint[u] = true;
            }
            if (u == root)
            {
                children++;
            }
        }
        else if (vis[v])
        {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (u == root && children >= 2)
    {
        cutPoint[root] = true;
    }
}

void Graph::printCutPoint()
{
    for (int i = 1; i <= vertexNum; i++)
    {
        if (cutPoint[i])
        {
            cutPointNum++;
        }
    }
    cout << cutPointNum << endl;
    for (int i = 1; i <= vertexNum; i++)
    {
        if (cutPoint[i])
        {
            cout << i << ' ';
        }
    }
}

int main()
{
    int n, m;
    cin >> n >> m;
    Graph G(n, m);
    G.init();
    G.addEdge();
    G.findSCC();
    G.printCutPoint();
    return 0;
}
```

## 无向图求桥

### 基本概念

- 桥：无向连通图中，如果删除某边后，图的连通性被破坏，则称该边为桥

### Tarjan算法应用

[src: uva](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=737)

**题意**：给定无向图G。求总桥数，并按字典序输出所有桥。

**题解**：利用Tarjan算法思想。
- 当且仅当无向边(u, v)为树边(父子边)且low[v]>dfn[u] (不成环)时，该边为桥

**实现代码**：
```cpp
// #include <bits\stdc++.h>
#include <iostream>
#include <algorithm>
using namespace std;
const int N = 200001;

class Graph
{
private:
    int vertexNum;
    struct node {
        int from;
        int to;
        int next;
    };
    node edge[N];
    int head[N];
    int cnt;

    int tot;
    int low[N];
    int dfn[N];
    bool vis[N];

    int cutEdgeNum;
    int cutEdge[N];

public:
    Graph(int n) : vertexNum(n) {}
    void init();
    void addEdge();
    void findSCC();
    void Tarjan(int, int);
    void printCutEdge();
};

void Graph::init()
{
    cnt = 0;
    tot = 0;
    cutEdgeNum = 0;
    for (int i = 0; i < vertexNum; i++) {
        head[i] = -1;
        low[i] = 0;
        dfn[i] = 0;
        vis[i] = 0;
        cutEdge[i] = false;
    }
}

void Graph::addEdge()
{
    for (int i = 0; i < vertexNum; i++) {
        int u, n, v;
        cin >> u;
        cin.get();
        cin.get();
        cin >> n;
        cin.get();
        for (int j = 0; j < n; j++) {
            cin >> v;
            edge[cnt].from = u;
            edge[cnt].to = v;
            edge[cnt].next = head[u];
            head[u] = cnt++;
        }
    }
}

void Graph::findSCC()
{
    for (int i = 0; i < vertexNum; i++) {
        if (!dfn[i]) {
            Tarjan(i, 0);
        }
    }
}

void Graph::Tarjan(int u, int parent)
{
    low[u] = dfn[u] = ++tot;
    vis[u] = true;
    for (int i = head[u]; i != -1; i = edge[i].next) {
        int v = edge[i].to;
        if (v == parent)
            continue;
        if (!dfn[v]) {
            Tarjan(v, u);
            low[u] = min(low[u], low[v]);
            if (low[v] > dfn[u]) {
                cutEdge[++cutEdgeNum] = i;
            }
        } else if (vis[v]) {
            low[u] = min(low[u], dfn[v]);
        }
    }
}

void Graph::printCutEdge()
{
    for (int i = 1; i <= cutEdgeNum; i++) {
        int j = cutEdge[i];
        if (edge[j].from > edge[j].to) {
            swap(edge[j].from, edge[j].to);
        }
    }
    sort(cutEdge + 1, cutEdge + 1 + cutEdgeNum, [&](int a, int b) {if(edge[a].from != edge[b].from) return edge[a].from < edge[b].from; return edge[a].to < edge[b].to; });
    cout << cutEdgeNum << " critical links" << endl;
    for (int i = 1; i <= cutEdgeNum; i++) {
        int j = cutEdge[i];
        cout << edge[j].from << " - " << edge[j].to << endl;
    }
    cout << endl;
}

int main()
{
    int n;
    while (cin >> n) {
        Graph G(n);
        G.init();
        G.addEdge();
        G.findSCC();
        G.printCutEdge();
    }
    return 0;
}
```