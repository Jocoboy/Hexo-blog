---
title: 强连通分量
date: 2019-05-27 23:31:01
categories:
- C++
- Algorithm
    - Graph Theory
tags:
- Tarjan
- Kosaraju
- Gabow
---

## 特殊的图存储结构

### 前向星

前向星是一种特殊的边集数组，其中存储着每一条边的起点、终点、权值。
借助head数组构造前向星之前，必须将每一条边按起点从小到大排序(如果起点相同就按终点从小到大排序)，时间复杂度较高。

### 链式前向星

在前向星的基础上，向边集数组中引入next属性，代表与当前边同起点的下一条边的存储位置，以避免排序。
链式前向星与邻接表类似，不同之处在于，邻接表越先输入的边离结点越近而越早遍历，而链式前向星越先输入的边离结点越远而越晚遍历。
<!--more-->

## Tarjan算法

### 基本思想

基于深度优先遍历思想。

1. 将有向图中寻找强连通分量转化为搜索树中寻找搜索子树
2. 引入数组dfn，作为某点的时间戳(搜索次序编号)
3. 引入数组low，作为某点在搜索树中最小搜索子树的根结点
4. 遍历每一个顶点，借助栈记录顶点访问顺序
5. 遍历链式前向星每一条边，记边的起点为u，终点为v
6. 如果顶点v未被访问，继续搜索，搜索结束后，low[u]=min(low[u],low[v])
7. 如果顶点v已被访问并且顶点u还在栈内，low[u]=min(low[u],dfn[v])
8. 如果dfn[u]=low[u]，则顶点u为强连通分量的根结点，将v退栈直到u=v

### 应用场景

用于求解有向图中的所有强连通分量。

[src: codeforces](http://codeforces.com/contest/427/problem/C)

**题意**：给定有向图，并给出每个顶点的权值。求通过顶点将所有强连通分量联系在一起的最小代价和对应方案数。

**题解**：Tarjan算法模板题。将所有强连通分量联系在一起，即寻找每个强连通分量中权值最小的顶点，并求它们的和。权值最小的顶点每重复出现一次，对应方案数就会加一，运用乘法计数原理可求得对应方案数。

**实现代码**：

```cpp
#include <bits\stdc++.h>
using namespace std;
const int INF = 0x3f3f3f3f;
const int mod = 1e9 + 7;
const int N = 100001;

long long w[N], sum, total = 1;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    struct node {
        int to;
        int next;
    };
    node edge[N * 3];
    int head[N];

    stack<int> st;
    int tot;
    int low[N];
    int dfn[N];
    bool vis[N];

    //int SCCNum;
    //vector<int> SCC[N];

public:
    Graph(int n, int m) : vertexNum(n), edgeNum(m) {}
    void init();
    void addEdge();
    void findSCC();
    void Tarjan(int);
    //void print();
};

void Graph::init()
{
    tot = 0;
    //SCCNum = 0;
    for (int i = 1; i <= vertexNum; i++) {
        head[i] = -1;
        low[i] = 0;
        dfn[i] = 0;
        vis[i] = 0;
    }
}

void Graph::addEdge()
{
    for (int i = 1; i <= edgeNum; i++) {
        int u, v;
        cin >> u >> v;
        edge[i].to = v;
        edge[i].next = head[u];
        head[u] = i;
    }
}

void Graph::findSCC()
{
    for (int i = 1; i <= vertexNum; i++) {
        if (!dfn[i]) {
            Tarjan(i);
        }
    }
    cout << sum << ' ' << total << endl;
}

void Graph::Tarjan(int u)
{
    low[u] = dfn[u] = ++tot;
    st.push(u);
    vis[u] = true;
    for (int i = head[u]; i != -1; i = edge[i].next) {
        int v = edge[i].to;
        if (!dfn[v]) {
            Tarjan(v);
            low[u] = min(low[u], low[v]);
        } else if (vis[v]) {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (low[u] == dfn[u]) {
        int v;
        //SCCNum++;
        int minn = INF, k = 0;
        do {
            v = st.top();
            st.pop();
            vis[v] = false;
            if (minn > w[v]) {
                k = 1;
                minn = w[v];
            } else if (minn == w[v]) {
                k++;
            }
            //SCC[SCCNum].push_back(v);
        } while (u != v);
        sum += minn;
        total = total * k % mod;
    }
}

/*void Graph::print()
{
    cout << SCCNum << endl;
    for (int i = 1; i <= SCCNum; i++) {
        for (int j = 0; j < SCC[i].size(); j++) {
            cout << SCC[i][j] << ' ';
        }
        cout << endl;
    }
}*/

int main()
{
    int n, m;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> w[i];
    }
    cin >> m;
    Graph G(n, m);
    G.init();
    G.addEdge();
    G.findSCC();
    //G.print();
    return 0;
}
```

## Kosaraju算法

### 基本思想

基于深度优先遍历思想。

1. 首先对原图进行dfs并将出栈顺序逆序得到拓扑序列
2. 然后将原图的每一条边反向得到反图，按照步骤(1)生成的拓扑序列顺序对反图进行dfs染色，染成同色的子图就是一个强连通分量
3. 如果把每个强连通分量缩点，并按每个强连通分量求得的顺序标记缩点，那么这个顺序就是强连通分量缩点后形成的DAG的拓扑序列

### 应用场景

用于求解有向图中的所有强连通分量。

[src: POJ](http://poj.org/problem?id=2186)

**题意**：给定有向图。问你有多少个点可以被其他所有点访问到。

**题解**：Kosaraju算法模板题。正反两次dfs找到所有强连通分量后，将每个强连通分量缩点染色，然后统计每个缩点的出度。若有且只有一个出度为0的缩点，则答案为该缩点所对应的强连通分量中顶点的个数，否则答案为0。

**实现代码**：

```cpp
//#include <bits\stdc++.h>
#include <iostream>
#include <stack>
#include <vector>
using namespace std;
const int N = 10001;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    struct node {
        int to;
        int next;
    };
    node edge[N * 5];
    node redge[N * 5];
    int head[N];
    int rhead[N];

    stack<int> topo;
    int color[N];
    int outDegree[N];
    bool vis[N];

    int SCCNum;
    //vector<int> SCC[N];

public:
    Graph(int n, int m) : vertexNum(n), edgeNum(m) {}
    void init();
    void addedge();
    void findSCC();
    void dfs(int);
    void rdfs(int);
    //void print();
};

void Graph::init()
{
    SCCNum = 0;
    for (int i = 1; i <= vertexNum; i++) {
        head[i] = -1;
        rhead[i] = -1;
        color[i] = 0;
        outDegree[i] = 0;
        vis[i] = false;
    }
}

void Graph::addedge()
{
    for (int i = 1; i <= edgeNum; i++) {
        int u, v;
        cin >> u >> v;
        edge[i].to = v;
        edge[i].next = head[u];
        head[u] = i;

        redge[i].to = u;
        redge[i].next = rhead[v];
        rhead[v] = i;
    }
}

void Graph::findSCC()
{
    for (int i = 1; i <= vertexNum; i++) {
        if (!vis[i]) {
            dfs(i);
        }
    }
    for (int i = 1; i <= vertexNum; i++) {
        vis[i] = false;
    }
    while (!topo.empty()) {
        int i = topo.top();
        topo.pop();
        if (!vis[i]) {
            SCCNum++;
            rdfs(i);
        }
    }
    for (int u = 1; u <= vertexNum; u++) {
        for (int i = head[u]; i != -1; i = edge[i].next) {
            int v = edge[i].to;
            if (color[u] != color[v]) {
                outDegree[color[u]]++;
            }
        }
    }
    int tot = 0, num = -1, ans = 0;
    for (int i = 1; i <= SCCNum; i++) {
        if (!outDegree[i]) {
            tot++;
            num = i;
        }
    }
    if (tot == 1) {
        for (int i = 1; i <= vertexNum; i++) {
            if (color[i] == num) {
                ans++;
            }
        }
    }
    cout << ans << endl;
}

void Graph::dfs(int u)
{
    vis[u] = true;
    for (int i = head[u]; i != -1; i = edge[i].next) {
        int v = edge[i].to;
        if (!vis[v]) {
            dfs(v);
        }
    }
    topo.push(u);
}

void Graph::rdfs(int u)
{
    vis[u] = true;
    color[u] = SCCNum;
    //SCC[SCCNum].push_back(u);
    for (int i = rhead[u]; i != -1; i = redge[i].next) {
        int v = redge[i].to;
        if (!vis[v]) {
            rdfs(v);
        }
    }
}

/*void Graph::print()
{
    cout << SCCNum << endl;
    for (int i = 1; i <= SCCNum; i++) {
        for (int j = 0; j < SCC[i].size(); j++) {
            cout << SCC[i][j] << ' ';
        }
        cout << endl;
    }
}*/

int main()
{
    int n, m;
    cin >> n >> m;
    Graph G(n, m);
    G.init();
    G.addedge();
    G.findSCC();
    //G.print();
    return 0;
}
```

## Gabow算法

### 基本思想

基于深度优先遍历思想。

1. 优化Tarjan算法，利用二号栈取代dfn和low数组求强连通分量的根结点
2. 遍历有向图每一个顶点，借助一号栈记录顶点访问顺序
3. 遍历链式前向星每一条边，记边的起点为u，终点为v
4. 如果顶点v未被访问，继续搜索
5. 如果顶点v已被访问并且没有被删除，维护二号栈未构成环的结点(即删除二号栈中构成环的结点)
6. 如果二号栈的顶元素等于u，则顶点u为强连通分量的根结点，将v退栈直到u=v

### 应用场景

用于求解有向图中的所有强连通分量。

[src: codeforces](http://codeforces.com/contest/427/problem/C)

**题意**：给定有向图，并给出每个顶点的权值。求通过顶点将所有强连通分量联系在一起的最小代价和对应方案数。

**题解**：Gabow算法模板题。将所有强连通分量联系在一起，即寻找每个强连通分量中权值最小的顶点，并求它们的和。权值最小的顶点每重复出现一次，对应方案数就会加一，运用乘法计数原理可求得对应方案数。

**实现代码**：
```cpp
#include <bits\stdc++.h>
using namespace std;
const int INF = 0x3f3f3f3f;
const int mod = 1e9 + 7;
const int N = 100001;

long long w[N], sum, total = 1;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    struct node {
        int to;
        int next;
    };
    node edge[N * 3];
    int head[N];

    stack<int> st;
    int tot;
    stack<int> st2;
    int dfn[N];
    bool vis[N];

    //int SCCNum;
    //vector<int> SCC[N];

public:
    Graph(int n, int m) : vertexNum(n), edgeNum(m) {}
    void init();
    void addEdge();
    void findSCC();
    void Gabow(int);
    //void print();
};

void Graph::init()
{
    tot = 0;
    //SCCNum = 0;
    for (int i = 1; i <= vertexNum; i++) {
        head[i] = -1;
        dfn[i] = 0;
        vis[i] = false;
    }
}

void Graph::addEdge()
{
    for (int i = 1; i <= edgeNum; i++) {
        int u, v;
        cin >> u >> v;
        edge[i].to = v;
        edge[i].next = head[u];
        head[u] = i;
    }
}

void Graph::findSCC()
{
    for (int i = 1; i <= vertexNum; i++) {
        if (!dfn[i]) {
            Gabow(i);
        }
    }
    cout << sum << ' ' << total << endl;
}

void Graph::Gabow(int u)
{
    st.push(u);
    dfn[u] = ++tot;
    st2.push(u);
    for (int i = head[u]; i != -1; i = edge[i].next) {
        int v = edge[i].to;
        if (!dfn[v]) {
            Gabow(v);
        } else if (!vis[v]) {
            while (dfn[st2.top()] > dfn[v]) {
                st2.pop();
            }
        }
    }

    if (st2.top() == u) {
        int v;
        st2.pop();
        //SCCNum++;
        int minn = INF, k = 0;
        do {
            v = st.top();
            st.pop();
            vis[v] = true;
            //SCC[SCCNum].push_back(v);
            if (minn > w[v]) {
                k = 1;
                minn = w[v];
            } else if (minn == w[v]) {
                k++;
            }
        } while (u != v);
        sum += minn;
        total = total * k % mod;
    }
}

/*void Graph::print()
{
    cout << SCCNum << endl;
    for (int i = 1; i <= SCCNum; i++) {
        for (int j = 0; j < SCC[i].size(); j++) {
            cout << SCC[i][j] << ' ';
        }
        cout << endl;
    }
}*/

int main()
{
    int n, m;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> w[i];
    }
    cin >> m;
    Graph G(n, m);
    G.init();
    G.addEdge();
    G.findSCC();
    //G.print();
    return 0;
}
```