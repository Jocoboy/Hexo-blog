---
title: 图的最短路径
date: 2019-05-14 23:16:59
categories:
- C++
- Algorithm
    - Graph Theory
tags:
- Dijkstra
- Floyd
- Bellman-Ford
- Spfa
---

## Dijkstra算法

### 基本思想

基于广度优先遍历思想和贪心思想。

1. 给定G=<V,E,W>，指定源点s
2. 引入集合S和集合U，S用于记录已求出最短路径的顶点以及对应的最短路径长度，U用于记录未求出最短路径的顶点以及该顶点到源点s的距离
3. 最初S中只有源点s，U中只有除s之外的顶点
4. 从U中找出路径最短的顶点(U中顶点的路径指s到该顶点的路径)，将其加入到S中，并更新U中的顶点和顶点的路径
5. 重复步骤(4)，直到遍历完所有顶点
<!--more-->

### 应用场景

用于解决无负权边的单源最短路径问题。

[src: codeforces](http://codeforces.com/contest/601/problem/A)

**题意**：给定无向带权图G，G中无平行边或者自环，也没有负权边。将未连通的两个顶点用虚线相连(此操作后G中实线和虚线的条数之和应为n(n-1)/2)，火车只能走实线，汽车只能走虚线。问你从1开始，分别乘火车和汽车前往n，两种方式都能抵达的最短用时，无解输出-1。

**题解**：Dijkstra算法模板题，各边权值均为1，注意孤立点的情况。

**实现代码**：
```cpp
#include <bits/stdc++.h>
using namespace std;
const int INF = 0x3f3f3f3f;
const int N = 401;

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
    void reverse();
    int Dijistra();
};

void Graph::reverse()
{
    for (int i = 1; i <= vertexNum; i++) {
        for (int j = 1; j <= vertexNum; j++) {
            if (matrix[i][j] == 1) {
                matrix[i][j] = INF;
            } else {
                matrix[i][j] = 1;
            }
        }
    }
}

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
        int u, v;
        cin >> u >> v;
        matrix[u][v] = 1;
        matrix[v][u] = 1;
    }
}

int Graph::Dijistra()
{
    bool vis[N];
    int dis[N];

    for (int i = 1; i <= vertexNum; i++) {
        vis[i] = false;
        dis[i] = matrix[1][i];
    }
    vis[1] = true;
    dis[1] = 0;

    for (int i = 2; i <= vertexNum; i++) {
        int k, min_dis = INF;
        for (int j = 1; j <= vertexNum; j++) {
            if (!vis[j] && dis[j] < min_dis) {
                min_dis = dis[j];
                k = j;
            }
        }
        if (min_dis == INF)
            break;
        vis[k] = true;
        for (int j = 1; j <= vertexNum; j++) {
            if (!vis[j] && dis[j] > dis[k] + matrix[k][j]) {
                dis[j] = dis[k] + matrix[k][j];
            }
        }
    }
    return dis[vertexNum];
}

int main()
{
    int n, m;
    cin >> n >> m;
    Graph G(n, m);
    G.init();
    G.addEdge();
    int train = G.Dijistra();
    G.reverse();
    int car = G.Dijistra();

    if (train == INF || car == INF) {
        cout << -1;
    } else {
        cout << max(train, car);
    }
    return 0;
}
```

## Floyd算法

### 基本思想

基于动态规划思想。

1. 给定G=<V,E,W>，顶点个数为N
2. 引入矩阵M，M中的元素a[i][j]表示顶点i到顶点j的距离，若i和j不相邻，则a[i][j]=∞
3. 对矩阵M进行N次松弛操作(类似区间DP中添加分割点k)，第k次松弛后a[i][j]将更新为只经过1~k顶点时，顶点i到顶点j的最小距离
4. 完成N次松弛后，a[i][j]表示经过所有顶点时，顶点i到顶点j的最小距离

### 应用场景

用于解决多源最短路径问题。

[src: codeforces](http://codeforces.com/contest/295/problem/B)

**题意**：给定无向带权图G，G为连通图。按照一定顺序删除指定点及其所有连边。问你每次删除点之前图中现存任意两点间的距离和。

**题解**：Floyd算法模板题，各边权值任意，注意将删除点逆序存入后离线操作。

**实现代码**：
```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 501;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    int matrix[N][N];
    int del[N];
    long long ans[N];

public:
    Graph(int n, int m) : vertexNum(n), edgeNum(m) {}
    void init();
    void addEdge();
    void floyd();
};

void Graph::init()
{
    for (int i = 1; i <= vertexNum; i++) {
        ans[i] = 0;
    }
}

void Graph::addEdge()
{
    for (int i = 1; i <= vertexNum; i++) {
        for (int j = 1; j <= vertexNum; j++) {
            cin >> matrix[i][j];
        }
    }
}

void Graph::floyd()
{
    for (int i = vertexNum; i >= 1; i--) {
        cin >> del[i];
    }
    for (int k = 1; k <= vertexNum; k++) {
        for (int i = 1; i <= vertexNum; i++) {
            for (int j = 1; j <= vertexNum; j++) {
                matrix[del[i]][del[j]] = min(matrix[del[i]][del[j]], matrix[del[i]][del[k]] + matrix[del[k]][del[j]]);
                if (i <= k && j <= k) {
                    ans[k] += matrix[del[i]][del[j]];
                }
            }
        }
    }
    for (int i = vertexNum; i >= 1; i--) {
        cout << ans[i] << ' ';
    }
}

int main()
{
    int n, m;
    cin >> n;
    m = n * (n - 1) / 2;
    Graph G(n, m);
    G.init();
    G.addEdge();
    G.floyd();
    return 0;
}
```

## Bellman-Ford算法

### 基本思想

基于动态规划思想。

1. 给定G=<V,E,W>，顶点个数为N，指定源点s
2. 引入边集S，S中每个元素e有u、v、w三个属性，分别对应边的起点、终点、权值
3. 对边集S进行N-1次松弛操作后，S中的每个元素e的权值将最小化
4. 遍历S中的每个元素，若仍存在使得w减小的情况，则表明G中存在负权环

### 应用场景

用于解决有负权边的单源最短路径问题。

[src: POJ](http://poj.org/problem?id=3259)

**题意**：给定无向带权图G，向G中添加若干条有向负权边。问你G中是否存在负权环。

**题解**：Bellman-Ford算法模板题，注意有向边和无向边要分开讨论。

```cpp
// #include <bits/stdc++.h>
#include <iostream>
#include <vector>
using namespace std;
const int INF = 0x3f3f3f3f;
const int N = 501;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    struct node
    {
        int u;
        int v;
        int w;
        node(int u, int v, int w) : u(u), v(v), w(w) {}
        node();
    };
    vector<node> edge;

public:
    Graph(int n, int m) : vertexNum(n), edgeNum(m) {}
    void addEdge(int);
    bool Bellman_Ford();
};

void Graph::addEdge(int add)
{
    for (int i = 1; i <= edgeNum + add; i++)
    {
        int u, v, w;
        cin >> u >> v >> w;
        edge.push_back(node(u, v, w));
    }
}

bool Graph::Bellman_Ford()
{
    int dis[N];
    for (int i = 1; i <= vertexNum; i++)
    {
        dis[i] = INF;
    }

    for (int i = 1; i <= vertexNum - 1; i++)
    {
        int cnt = -edgeNum;
        for (vector<node>::iterator it = edge.begin(); it != edge.end(); it++ /*auto e : edge*/)
        {
            node e = *it;
            if (cnt++ < 0)
            {
                dis[e.u] = min(dis[e.u], dis[e.v] + e.w);
                dis[e.v] = min(dis[e.v], dis[e.u] + e.w);
            }
            else
            {
                dis[e.v] = min(dis[e.v], dis[e.u] - e.w);
            }
        }
    }

    int cnt = -edgeNum;
    for (vector<node>::iterator it = edge.begin(); it != edge.end(); it++)
    {
        node e = *it;
        if (cnt++ < 0)
        {
            if (dis[e.u] > dis[e.v] + e.w || dis[e.v] > dis[e.u] + e.w)
                return true;
        }
        else
        {
            if (dis[e.v] > dis[e.u] - e.w)
                return true;
        }
    }
    return false;
}

int main()
{
    int T;
    for (cin >> T; T > 0; T--)
    {
        int n, m, w;
        cin >> n >> m >> w;
        Graph G(n, m);
        G.addEdge(w);
        cout << (G.Bellman_Ford() ? "YES" : "NO") << endl;
    }
    return 0;
}
```

## Spfa算法

### 基本思想

基于广度优先遍历和动态规划思想，对Bellman-Ford算法进行优化。

1. 给定G=<V,E,W>，顶点个数为N，指定源点s
2. 引入队列Q，将源点s入队
3. 队首元素u出队，遍历u的所有出边u->v进行松弛操作，若此时v不在Q中，则将其入队
4. 当Q非空时，重复步骤(3)，同时判断元素v入队次数是否大于N-1，若是则表明G中存在负权环

### 应用场景

用于解决有负权边的单源最短路径问题。

[src: POJ](http://poj.org/problem?id=1860)

**题意**：给定有向带权图G，若G中某两个顶点之间有边，一定是一对方向相反的有向边，且权值可能不同。问你G中是否存在负权环。

**题解**：Spfa算法模板题，注意负权环和正权环是相对的概念。

```cpp
// #include <bits/stdc++.h>
#include <iostream>
#include <queue>
#include <vector>
using namespace std;
const int N = 101;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    struct node
    {
        int v;
        double r;
        double c;
        node(int v, double r, double c) : v(v), r(r), c(c) {}
        node();
    };
    vector<node> edge[N];

public:
    Graph(int n, int m) : vertexNum(n), edgeNum(m) {}
    void addEdge();
    bool Spfa(int, double);
};

void Graph::addEdge()
{
    for (int i = 1; i <= edgeNum; i++)
    {
        int u, v;
        double ruv, cuv, rvu, cvu;
        cin >> u >> v >> ruv >> cuv >> rvu >> cvu;
        edge[u].push_back(node(v, ruv, cuv));
        edge[v].push_back(node(u, rvu, cvu));
    }
}

bool Graph::Spfa(int s, double v)
{
    double dis[N];
    int cnt[N];
    bool vis[N];
    for (int i = 1; i <= vertexNum; i++)
    {
        dis[i] = 0;
        cnt[i] = 0;
        vis[i] = false;
    }
    dis[s] = v;
    cnt[s]++;
    vis[s] = true;
    queue<int> q;
    q.push(s);
    while (!q.empty())
    {
        int u = q.front();
        q.pop();
        vis[u] = false;
        for (vector<node>::iterator it = edge[u].begin(); it != edge[u].end(); it++ /*auto e : edge[u]*/)
        {
            node e = *it;
            if (dis[e.v] < (dis[u] - e.c) * e.r)
            {
                dis[e.v] = (dis[u] - e.c) * e.r;
                if (!vis[e.v])
                {
                    vis[e.v] = true;
                    q.push(e.v);
                    cnt[e.v]++;
                    if (cnt[e.v] > vertexNum - 1)
                        return true;
                }
            }
        }
    }
    return false;
}

int main()
{
    int n, m, s;
    double v;
    cin >> n >> m >> s >> v;
    Graph G(n, m);
    G.addEdge();
    cout << (G.Spfa(s, v) ? "YES" : "NO") << endl;
    return 0;
}
```