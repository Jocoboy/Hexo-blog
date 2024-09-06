---
title: 矩阵快速幂
date: 2019-06-20 23:39:33
categories:
- C++
- Algorithm
    - Number Theory
tags:
mathjax: true
---

## 基本概念

- 单位矩阵：主对角线元素全为1，其余元素全为0的矩阵，充当数乘运算中的1
- 矩阵乘积：$C(c_{ij}) = A(a_{ik}) * B(b_{kj})$, 其中$c_{ij}= \displaystyle\sum_{k=1}^{n}a_{ik} * b_{kj}$
- 快速幂：计算$a^n$时，将指数n转化为二进制数，记第i位为$k_i$，则$n = \displaystyle\sum_{i=1}^{n}k_i*2^{i-1}$，以此将O(n)的复杂度降至O(logn)
<!--more-->

## 应用场景

### k步可达路径方案数

[src: HDU](http://acm.hdu.edu.cn/showproblem.php?pid=2157)

**题意**：给定有向图G，给出若干组(A,B,k)。问你从A到B恰好经过k个顶点的方案数。

**题解**：矩阵快速幂模板题。

**实现代码**：
```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int mod = 1000;

template <class T, int N>
class Matrix
{
private:
    int n;
    T value[N][N];

public:
    Matrix(bool isIdentityMatrix = false) : n(N)
    {
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < n; j++)
            {
                value[i][j] = T(0);
            }
        }

        if (isIdentityMatrix)
            for (int i = 0; i < n; i++)
            {
                value[i][i] = T(1);
            }
    }
    Matrix operator+(const Matrix &b) const
    {
        Matrix<T, N> ret;
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                ret.value[i][j] = (value[i][j] + b.value[i][j]) % mod;
        return ret;
    }
    Matrix operator*(const Matrix &b) const
    {
        Matrix<T, N> ret;
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                for (int k = 0; k < n; k++)
                    ret.value[i][j] = (ret.value[i][j] + value[i][k] * b.value[k][j]) % mod;
        return ret;
    }
    Matrix &operator%(const int mod)
    {
        return *this;
    }
    void setValue(int u, int v) { value[u][v] = 1; }
    T getValue(int u, int v) { return this->value[u][v]; }
};

template <class T>
T fpow(T a, LL n)
{
    T ret(true);
    while (n)
    {
        ret = ret * (n & 1 ? a : T(1)) % mod;
        a = a * a % mod;
        n >>= 1;
    }
    return ret;
}

int main()
{
    int n, m, t;
    while (cin >> n >> m && (n + m))
    {
        Matrix<LL, 25> a, b;
        for (int i = 0; i < m; i++)
        {
            int u, v;
            cin >> u >> v;
            a.setValue(u, v);
        }
        for (cin >> t; t > 0; t--)
        {
            int u, v, k;
            cin >> u >> v >> k;
            b = fpow(a, k);
            cout << b.getValue(u, v) << endl;
        }
    }
    return 0;
}
```

### Fibonacci数列第k项

[src: POJ](http://poj.org/problem?id=3070)

**题意**：求Fibonacci数列第k项的后四位(1≤k≤1000000000)。

<img src="Matrix-Quick-Power/matrix_quick_power.png">

**题解**：根据递推式$F_n=F_{n-1}+F_{n-2}$构造二阶矩阵作为底数。

**实现代码**：
```cpp
// #include <bits/stdc++.h>
#include <iostream>
using namespace std;
typedef long long LL;
const int mod = 10000;

template <class T, int N>
class Matrix
{
private:
    int n;
    T value[N][N];

public:
    Matrix(bool isIdentityMatrix = false) : n(N)
    {
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < n; j++)
            {
                value[i][j] = T(0);
            }
        }

        if (isIdentityMatrix)
            for (int i = 0; i < n; i++)
            {
                value[i][i] = T(1);
            }
    }
    Matrix operator+(const Matrix &b) const
    {
        Matrix<T, N> ret;
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                ret.value[i][j] = (value[i][j] + b.value[i][j]) % mod;
        return ret;
    }
    Matrix operator*(const Matrix &b) const
    {
        Matrix<T, N> ret;
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                for (int k = 0; k < n; k++)
                    ret.value[i][j] = (ret.value[i][j] + value[i][k] * b.value[k][j]) % mod;
        return ret;
    }
    Matrix &operator%(const int mod)
    {
        return *this;
    }
    void setValue(int u, int v) { value[u][v] = 1; }
    T getValue(int u, int v) { return this->value[u][v]; }
};

template <class T>
T fpow(T a, LL n)
{
    T ret(true);
    while (n)
    {
        ret = ret * (n & 1 ? a : T(1)) % mod;
        a = a * a % mod;
        n >>= 1;
    }
    return ret;
}

int main()
{
    int n;
    while (cin >> n && n != -1)
    {
        Matrix<LL, 2> a, b;
        a.setValue(0, 0);
        a.setValue(0, 1);
        a.setValue(1, 0);
        b = fpow(a, n);
        cout << b.getValue(0, 1) << endl;
    }
    return 0;
}
```