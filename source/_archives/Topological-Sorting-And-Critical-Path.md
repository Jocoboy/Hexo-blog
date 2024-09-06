---
title: 拓扑排序与关键路径
date: 2019-05-21 23:22:26
categories:
- C++
- Algorithm 
    - Graph Theory
tags:
- Toposort
- Critical Path
mathjax: true
---

## DAG拓扑排序

<img src="Topological-Sorting-And-Critical-Path/topological_sorting.png">

1. 给定G=<V,E,W>，G为无环有向图，顶点个数为N
1. 找到G中入度为0的顶点并输出
2. 删除该点的所有连边，重复步骤(1)，直到G中无入度为0的顶点
3. 若输出顶点个数小于N，则证明G中有回路，没有关键路径
<!--more-->

## AOE网关键路径

### 基本概念

**定义**：

AOE网中从起点到终点具有最大路径长度的路径。

**特征属性**：

- 事件——AOE网中的顶点
- 活动——AOE网中的边
- $ES(i)$和$LS(i)$——事件$i$最早和最晚开始时间
- $EF(i)$和$LF(i)$——事件$i$最早和最晚完成时间
- $ES(i,j)$和$LS(i,j)$——活动$(i,j)$最早和最晚开始时间
- $EF(i,j)$和$LF(i,j)$——活动$(i,j)$最早和最晚完成时间
- $SL(i,j)$——活动$(i,j)$最晚开始(完成)时间与最早开始(完成)时间的差

**公式**：

- $ES(1) = 0, ES(i) = max( ES(j) + w_{ji} | <i,j> ∈ E )$
- $LF(n) = ES(n), LF(i) = min( LF(j) - w_{ij} | <i,j> ∈ E )$

- $ES(i,j) = ES(i), EF(i,j) = ES(i) + w_{ij}$
- $LF(i,j) = LF(j), LS(i,j) = LF(j) - w_{ij}$

- $SL(i,j) = LS(i,j) - ES(i,j) = LF(i,j) - EF(i,j)$

### 基本思想

1. 借助链式前向星建立AOE网
2. 遍历链式前向星每个结点，找到AOE网的拓扑序列并将其存入栈内，同时更新VE[i]最大值，此过程借助队列实现
3. 逆拓扑序遍历链式前向星，同时更新VL[i]最小值，此过程借助栈实现
4. 遍历链式前向星每个结点，计算E[i]和L[i]，若二者相等，则为关键路径上的关键活动

### 应用

<img src="Topological-Sorting-And-Critical-Path/critical_path.png">

给定AOE网，求关键路径及其长度，注意关键路径可能不止一条(此处略去序列化操作)。

**测试样例**:
```
9 15
1 2 3
1 3 2
1 4 4
2 3 0
2 5 4
3 4 2
3 5 4
3 6 4
4 7 5
5 6 0
5 7 3
5 9 6
6 8 3
7 8 1
8 9 1
```

**输出结果**:
```
13
1 2
2 5
2 3
3 5
5 9
```

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
    struct node
    {
        int to;
        int next;
        int w;
    };
    node edge[N];
    int head[N];
    int inDegree[N];

    stack<int> topo;
    int VE[N];
    int VL[N];
    int E[N];
    int L[N];

public:
    Graph(int n, int m) : vertexNum(n), edgeNum(m) {}
    void init();
    void addEdge();
    void topoSort();
    void CriticalPath();
};

void Graph::init()
{
    for (int i = 1; i <= vertexNum; i++)
    {
        inDegree[i] = 0;
        head[i] = -1;
        VE[i] = 0;
        VL[i] = 0;
        E[i] = 0;
        L[i] = 0;
    }
}

void Graph::addEdge()
{
    for (int i = 1; i <= edgeNum; i++)
    {
        int u, v, w;
        cin >> u >> v >> w;
        inDegree[v]++;
        edge[i].to = v;
        edge[i].next = head[u];
        head[u] = i;
        edge[i].w = w;
    }
}

void Graph::topoSort()
{
    queue<int> q;
    for (int i = 1; i <= vertexNum; i++)
    {
        if (!inDegree[i])
        {
            q.push(i);
        }
    }
    while (!q.empty())
    {
        int u = q.front();
        q.pop();
        topo.push(u);
        for (int i = head[u]; i != -1; i = edge[i].next)
        {
            int v = edge[i].to;
            int w = edge[i].w;
            if (!--inDegree[v])
            {
                q.push(v);
            }
            VE[v] = max(VE[v], VE[u] + w);
        }
    }
}

void Graph::CriticalPath()
{
    topoSort();
    cout << VE[vertexNum] << endl;
    for (int i = 1; i <= vertexNum; i++)
    {
        VL[i] = VE[vertexNum];
    }
    while (!topo.empty())
    {
        int u = topo.top();
        topo.pop();
        for (int i = head[u]; i != -1; i = edge[i].next)
        {
            int v = edge[i].to;
            int w = edge[i].w;
            VL[u] = min(VL[u], VL[v] - w);
        }
    }
    for (int u = 1; u <= vertexNum; u++)
    {
        for (int i = head[u]; i != -1; i = edge[i].next)
        {
            int v = edge[i].to;
            int w = edge[i].w;
            E[i] = VE[u];
            L[i] = VL[v] - w;
            if (E[i] == L[i])
            {
                cout << u << ' ' << v << endl;
            }
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
    G.CriticalPath();
    return 0;
}
```
