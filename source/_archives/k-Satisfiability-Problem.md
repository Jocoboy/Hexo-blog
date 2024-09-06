---
title: k-适定性问题
date: 2019-06-04 23:32:29
categories:
- C++
- Algorithm
    - Graph Theory
tags:
- 2-sat
- Tarjan
---

## k-sat问题

### 基本概念

给定若干集合，每个集合有若干元素，每个集合元素个数不超过k个。
给出从每个集合中选取元素的约束条件以及目标方案，判可行性。若可行，求可行解。

当k=1时，每个集合中元素的个数至多为1，问题的答案显而易见。
当k=2时，假定每个集合中元素有且仅有2个，且2个元素不允许同时被取出，即为2-sat问题的一般情景。
当k>=3时，k-sat问题是NP-Complete问题，无法在多项式时间内解决，不作讨论。
<!--more-->

### 应用场景

#### 2-sat判可行性

[src: HDU](http://acm.hdu.edu.cn/showproblem.php?pid=3062)

**题意**：给定n个集合，每个集合中有2个元素，给定集合间的m对矛盾关系。判从2n个元素中取n个元素的可行性。

**题解**：2-sat裸题。将元素之间的一对矛盾关系转化为顶点之间的一条无向连边，建图后Tarjan算法判图连通性即可。注意此题卡cin。

**实现代码**：
```cpp
#include <bits\stdc++.h>
using namespace std;
const int N = 2001;

class Graph
{
private:
    int vertexNum;
    int edgeNum;
    struct node {
        int to;
        int next;
    };
    node edge[N * 500];
    int head[N];

    stack<int> st;
    int tot;
    int low[N];
    int dfn[N];
    bool vis[N];

    int SCCNum;
    int color[N];
    //vector<int> SCC[N];

public:
    Graph(int n, int m) : vertexNum(n << 1), edgeNum(m << 1) {}
    void init();
    void addEdge();
    bool findSCC();
    void Tarjan(int);
    //void print();
};

void Graph::init()
{
    tot = 0;
    SCCNum = 0;
    for (int i = 0; i < vertexNum; i++) {
        head[i] = -1;
        low[i] = 0;
        dfn[i] = 0;
        vis[i] = 0;
    }
}

void Graph::addEdge()
{
    for (int i = 0; i < edgeNum; i++) {
        int u, v, a1, a2, c1, c2;
        cin >> a1 >> a2 >> c1 >> c2;
        u = (a1 << 1) + c1;
        v = (a2 << 1) + c2;

        edge[i].to = v ^ 1;
        edge[i].next = head[u];
        head[u] = i++;

        edge[i].to = u ^ 1;
        edge[i].next = head[v];
        head[v] = i;
    }
}

bool Graph::findSCC()
{
    for (int i = 0; i < vertexNum; i++) {
        if (!dfn[i]) {
            Tarjan(i);
        }
    }
    for (int i = 0; i < vertexNum; i += 2) {
        if (color[i] == color[i + 1]) {
            return false;
        }
    }
    return true;
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
        SCCNum++;
        do {
            v = st.top();
            st.pop();
            vis[v] = false;
            //SCC[SCCNum].push_back(v);
            color[v] = SCCNum;
        } while (u != v);
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
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    int n, m;
    while (cin >> n >> m) {
        Graph G(n, m);
        G.init();
        G.addEdge();
        cout << (G.findSCC() ? "YES" : "NO") << endl;
        //G.print();
    }
    return 0;
}
```

#### 2-sat求可行解

[src: codeforces](http://codeforces.com/contest/876/problem/E)

**题意**：给定n串数字字符，所有数字不超过m。给出一种标记操作，带上标记的数字字典序将小于一般数字。判若干次操作后可使字符串升序的可行性。若可行，输出可行解。

**题解**：对于相同位置的字符，标记时存在若干对矛盾关系。依次建边后Tarjan算法判图连通性的同时，记录各连通分支中顶点元素的拓扑序列，依据相邻顶点拓扑序输出标记方案。

**实现代码**：
```cpp
#include <bits\stdc++.h>
using namespace std;
const int N = 200001;

vector<int> let[N];

class Graph
{
private:
    int vertexNum;
    //int edgeNum;
    struct node {
        int to;
        int next;
    };
    node edge[N];
    int head[N];
    int cnt;

    stack<int> st;
    int tot;
    int low[N];
    int dfn[N];
    bool vis[N];

    int sum;
    int topo[N];
    int SCCNum;
    int color[N];
    //vector<int> SCC[N];

public:
    Graph(int n) : vertexNum(n << 1) {}
    void init();
    void addEdge(int, int);
    void findSCC();
    void Tarjan(int);
    //void print();
};

void Graph::init()
{
    sum = 0;
    cnt = 0;
    tot = 0;
    SCCNum = 0;
    for (int i = 0; i < vertexNum; i++) {
        head[i] = -1;
        low[i] = 0;
        dfn[i] = 0;
        vis[i] = 0;
    }
}

void Graph::addEdge(int u, int v)
{
    edge[cnt].to = v;
    edge[cnt].next = head[u];
    head[u] = cnt++;
}

void Graph::findSCC()
{
    for (int i = 0; i < vertexNum; i++) {
        if (!dfn[i]) {
            Tarjan(i);
        }
    }
    bool flag = true;
    for (int i = 0; i < vertexNum && flag; i++) {
        if (color[i] == color[i + 1]) {
            flag = false;
        }
    }
    if (flag) {
        puts("Yes");
        vector<int> ans;
        for (int i = 0; i < vertexNum; i += 2) {
            if (topo[i] < topo[i +1]) {
                ans.push_back((i >> 1) + 1);
            }
        }
        cout << ans.size() << endl;
        for (auto v : ans) {
            cout << v << " ";
        }
    } else {
        puts("No");
    }
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
        SCCNum++;
        do {
            v = st.top();
            st.pop();
            vis[v] = false;
            color[v] = SCCNum;
            topo[v] = ++sum;
            //SCC[SCCNum].push_back(v);
        } while (u != v);
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
    for (int i = 1; i <= n; i++) {
        int len;
        cin >> len;
        for (int j = 1; j <= len; j++) {
            int x;
            cin >> x;
            let[i].push_back(x);
        }
    }
    Graph G(m);
    G.init();
    for (int i = 1; i <= n - 1; i++) {
        bool flag = false;
        int mlen = min(let[i].size(), let[i + 1].size());
        for (int j = 0; j < mlen; j++) {
            int u = let[i][j] - 1;
            int v = let[i + 1][j] - 1;
            if (u != v) {
                if (u < v) {
                    G.addEdge(u << 1 ^ 1, v << 1 ^ 1);
                    G.addEdge(v << 1, u << 1);
                } else {
                    G.addEdge(u << 1 ^ 1, u << 1);
                    G.addEdge(v << 1, v << 1 ^ 1);
                }
                flag = true;
                break;
            }
        }
        if (!flag && let[i].size() > let[i + 1].size()) {
            return 0 * puts("No");
        }
    }
    G.findSCC();
    //G.print();
    return 0;
}
```