---
title: 最优二叉树
date: 2019-05-23 23:28:25
categories:
- C++
- Algorithm
    - Graph Theory
tags:
- Huffman
---

## Huffman算法

### 基本思想

基于贪心思想。

1. 以权值为w1,w2...wn的n个结点构成n棵二叉树，其中每棵二叉树仅有一个权值为wi的根结点
2. 选取两颗根结点权值最小的树作为左右子树构造一棵新二叉树，并置新二叉树根结点权值为左右子树上根结点的权值之和
3. 将合并的两棵二叉树从森林中删除，同时将新二叉树加入森林
4. 重复步骤(2)和(3)，直到森林中只剩一棵二叉树
<!--more-->

### 应用

<img src="Optimal-Binary-Search-Tree/optimal_binary_search_tree.png" style="height: 50%; width: 50%;">

给定n个结点的权值，求最优二叉树(注意贪心思想，n个结点权值的给出顺序不同，最终生成的最优二叉树也不同)。

**测试样例**:
```
8
3 5 7 8 11 14 23 29
```

**输出结果**:
```
0	3	8	-1	-1
1	5	8	-1	-1
2	7	9	-1	-1
3	8	9	-1	-1
4	11	10	-1	-1
5	14	11	-1	-1
6	23	12	-1	-1
7	29	13	-1	-1
8	8	10	0	1
9	15	11	2	3
10	19	12	4	8
11	29	13	5	9
12	41	14	6	10
13	58	14	7	11
14	100	-1	12	13
```

**实现代码**：

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 101;

class Graph
{
private:
    struct node {
        int weight;
        int parent;
        int l_son;
        int r_son;
    };
    node tree[N];

public:
    void init(int);
    void Huffman(int);
    void select(int, int &, int &);
    void update(int, int, int);
    void print(int);
};

void Graph::init(int n)
{
    for (int i = 0; i < n; i++) {
        cin >> tree[i].weight;
    }
    for (int i = 0; i < 2 * n - 1; i++) {
        tree[i].parent = -1;
        tree[i].l_son = -1;
        tree[i].r_son = -1;
    }
}

void Graph::update(int parent, int l_son, int r_son)
{
    tree[l_son].parent = parent;
    tree[r_son].parent = parent;
    tree[parent].l_son = l_son;
    tree[parent].r_son = r_son;
    tree[parent].weight = tree[l_son].weight + tree[r_son].weight;
}

void Graph::select(int n, int &l_son, int &r_son)
{
    for (int i = 0; i < n; i++) {
        if (tree[i].parent == -1) {
            l_son = i;
            break;
        }
    }
    for (int i = 0; i < n; i++) {
        if (tree[i].parent == -1 && tree[l_son].weight > tree[i].weight) {
            r_son = i;
        }
    }
    for (int j = 0; j < n; j++) {
        if (tree[j].parent == -1 && j != l_son) {
            r_son = j;
            break;
        }
    }
    for (int j = 0; j < n; j++) {
        if (tree[j].parent == -1 && j != l_son && tree[r_son].weight > tree[j].weight) {
            r_son = j;
        }
    }
}

void Graph::Huffman(int n)
{
    init(n);
    for (int i = n; i < 2 * n - 1; i++) {
        int l, r;
        select(i, l, r);
        update(i, l, r);
    }
}

void Graph::print(int n)
{
    cout << left;
    for (int i = 0; i < n * 2 - 1; i++) {
        cout << setw(10) << i;
        cout << setw(10) << tree[i].weight;
        cout << setw(10) << tree[i].parent;
        cout << setw(10) << tree[i].l_son;
        cout << setw(10) << tree[i].r_son << endl;
    }
}

int main()
{
    int n;
    cin >> n;
    Graph G;
    G.Huffman(n);
    G.print(n);
    return 0;
}
```