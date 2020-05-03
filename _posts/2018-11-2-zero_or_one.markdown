---
layout: default
title: 0 或 1 (minimal path)
date: 2018-11-2 11:23
post-link:
---

### [Problem Description(hdu-4370)][problem]

### 解题思路

读懂题很考人。很容易就当成水题来做了。
借鉴网上的思路。转化成最短路来做。

由（1）得知，1结点只有一个出度
由（2）得知，n结点只有一个入度
由（3）得知，其余结点出度 == 入度

一下两种情况满足上述三种规则：
A） 1到n形成一条路--->求1到n的最短路
B） 1和n没有路---> 1和n各自形成非自环的闭环。
求两者的最小值，即为题目要求的∑(Cij * Xij)
细节：求非自环的闭环：不将dist[start]初始化为0，将其初始化为inf(也就是说到start到start的路不再是自环，需要另找一条路)，在队列q中放入与之相邻的点。dist[start] 则是满足要求的最小环。

### 代码

```c++
#include <iostream>
#include <cstring>
#include <algorithm>
#include <cmath>
#include <cstdio>
#include <vector>
#include <queue>
#include <set>
#include <map>
#include <stack>
#define ll long long
//#define local

using namespace std;

const int MOD = 1e9+7;
const int inf = 0x3f3f3f3f;
const int maxn = 305;
int N;
int c[maxn][maxn];
int inque[maxn];
int dist[maxn];

void spfa (int start, int end) {
    queue<int> q;
    inque[start] = 0;
    dist[start] = inf;
    for (int i = 1; i <= N; ++i) {
        if(i == start) continue;
        dist[i] = c[start][i];
        inque[i] = 1;
        q.push(i);
    }
    while (q.size()) {
        int u = q.front();
        q.pop();
        inque[u] = 0;
        for (int i = 1; i <= N; i++) {
            if(dist[i] > dist[u]+c[u][i]) {
                dist[i] = dist[u]+c[u][i];
                if(!inque[i]) {
                    inque[i] = 1;
                    q.push(i);
                }
            }
        }
    }
}

int main() {
#ifdef local
    if(freopen("/Users/Andrew/Desktop/data.txt", "r", stdin) == NULL) printf("can't open this file!\n");
#endif
    while (cin >> N) {
        for (int i = 1; i <= N; ++i)
            for (int j = 1; j <= N; ++j)
                scanf("%d", &c[i][j]);
        spfa(1, N);
        int ans = dist[N];
        int c1 = dist[1];
        spfa(N, N);
        int c2 = dist[N];
        printf("%d\n", min(ans, c1+c2));
    }
#ifdef local
    fclose(stdin);
#endif
    return 0;
}
```

[problem]: <http://acm.hdu.edu.cn/showproblem.php?pid=4370>