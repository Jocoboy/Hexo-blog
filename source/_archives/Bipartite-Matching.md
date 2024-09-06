---
title: 二分图匹配
date: 2019-06-10 23:33:42
categories:
- C++
- Algorithm
    - Graph Theory
tags:
- Hungary
- KM
---

## Hungary算法

<img src="Bipartite-Matching/bipartite_matching.png">

### 基本概念

1. 交替路：从一个未匹配点出发，依次经过非匹配边、匹配边、非匹配边...形成的路径。
2. 增广路：从一个未匹配点出发，沿着交替路途经另一个未匹配点的路径。图中，9->4->8->1->6->2就是一条增广路。增广路的非匹配边比匹配边多一条。

### 基本思想

通过寻找增广路，不断交换增广路中的匹配边与非匹配边的身份(相当于添加一条匹配边)，直到找不到增广路为止。
<!--more-->

### 应用场景

用于解决无权二分图最大匹配问题。

[src: HDU](http://acm.hdu.edu.cn/showproblem.php?pid=2063)

**题意**：给定无权二分图，求最大匹配数。

**题解**：Hungary算法模板题。

**实现代码**：
```cpp
#include <bits\stdc++.h>
using namespace std;
const int N = 501;

class Graph
{
private:
    int edgeNum;
    int lvertexNum;
    int rvertexNum;
    int maxMatch;
    int matrix[N][N];
    bool vis[N];
    int lvertex[N];
    int rvertex[N];

public:
    Graph(int m, int ln, int rn) : edgeNum(m), lvertexNum(ln), rvertexNum(rn) {}
    void init();
    void addEdge();
    void Hungary();
    bool find(int);
};

void Graph::init()
{
    maxMatch = 0;
    memset(matrix, 0, sizeof(matrix));
    memset(lvertex, 0, sizeof(lvertex));
    memset(rvertex, 0, sizeof(rvertex));
}

void Graph::addEdge()
{
    for (int i = 1; i <= edgeNum; i++)
    {
        int u, v;
        cin >> u >> v;
        matrix[u][v] = 1;
    }
}

void Graph::Hungary()
{
    for (int i = 1; i <= lvertexNum; i++)
    {
        if (!lvertex[i])
        {
            memset(vis, 0, sizeof(vis));
            if (find(i))
            {
                maxMatch++;
            }
        }
    }
    cout << maxMatch << endl;
}

bool Graph::find(int u)
{
    for (int v = 1; v <= rvertexNum; v++)
    {
        if (matrix[u][v] && !vis[v])
        {
            vis[v] = true;
            if (!rvertex[v] || find(rvertex[v]))
            {
                lvertex[u] = v;
                rvertex[v] = u;
                return true;
            }
        }
    }
    return false;
}

int main()
{
    int m, ln, rn;
    while (cin >> m && m)
    {
        cin >> ln >> rn;
        Graph G(m, ln, rn);
        G.init();
        G.addEdge();
        G.Hungary();
    }
    return 0;
}
```

## KM算法

### 基本思想

1. 初始化时，将左侧顶点赋值为最大权重，右侧顶点赋值为0。
2. 用Hungary算法为左侧第i个顶点匹配最大权重边。
3. 若匹配失败，遍历左右两侧顶点，在已被匹配的左侧顶点与未被匹配的右侧顶点之间寻找最小代价，并根据左右两侧顶点匹配情况降低左侧顶点权重或增加右侧顶点权重，重复步骤(2)，直到该点匹配成功。
4. 若匹配成功，记录匹配顶点，重复步骤(2)，直到左侧所有顶点都被匹配。

### 应用场景

用于解决带权二分图完美匹配下的最优匹配问题。

[src: HDU](http://acm.hdu.edu.cn/showproblem.php?pid=2255)

**题意**：给定带权二分图，求最大匹配权。

**题解**：KM算法模板题。注意此题卡cin。

**实现代码**：
```cpp
#include <bits\stdc++.h>
using namespace std;
const int INF = 0x3f3f3f3f;
const int N = 301;

class Graph
{
private:
    int vertexNum;
    int optMatch;
    int matrix[N][N];
    int rvertex[N];
    int lexp[N];
    int rexp[N];
    bool lvis[N];
    bool rvis[N];

public:
    Graph(int n) : vertexNum(n) {}
    void init();
    void addEdge();
    void KM();
    bool find(int);
};

void Graph::init()
{
    optMatch = 0;
    for (int i = 1; i <= vertexNum; i++) {
        rvertex[i] = 0;
        lexp[i] = INF;
        rexp[i] = 0;
        for (int j = 1; j <= vertexNum; j++) {
            lexp[i] = max(matrix[i][j], lexp[i]);
        }
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

void Graph::KM()
{
    init();
    for (int i = 1; i <= vertexNum; i++) {
        while (true) {
            int slack = INF;
            memset(lvis, 0, sizeof(lvis));
            memset(rvis, 0, sizeof(rvis));
            if (find(i))
                break;
            for (int i = 1; i <= vertexNum; i++) {
                if (lvis[i]) {
                    for (int j = 1; j <= vertexNum; j++) {
                        if (!rvis[j]) {
                            slack = min(slack, lexp[i] + rexp[j] - matrix[i][j]);
                        }
                    }
                }
            }

            for (int i = 1; i <= vertexNum; i++) {
                if (lvis[i]) {
                    lexp[i] -= slack;
                }

                if (rvis[i]) {
                    rexp[i] += slack;
                }
            }
        }
    }
    for (int i = 1; i <= vertexNum; i++) {
        optMatch += matrix[rvertex[i]][i];
    }
    cout << optMatch << endl;
}

bool Graph::find(int u)
{
    lvis[u] = true;
    for (int v = 1; v <= vertexNum; v++) {
        if (!rvis[v] && lexp[u] + rexp[v] == matrix[u][v]) {
            rvis[v] = true;
            if (!rvertex[v] || find(rvertex[v])) {
                rvertex[v] = u;
                return true;
            }
        }
    }
    return false;
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    int n;
    while (cin >> n) {
        Graph G(n);
        G.init();
        G.addEdge();
        G.KM();
    }
    return 0;
}
```
