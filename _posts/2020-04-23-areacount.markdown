---
layout: default
title: Atlantis(geometry+segment tree)
date: 2020-04-23 11:23
post-link:
---

### [problem description(hdu-1542)][problem]

#### Brief illustration

![](/images/sweep_sgt.png#center){:height="300px" width="300px"}

#### Solution

To solve this problem we use line-sweeping algorithm and segment tree.

We divide the area into many regular rectangle by several vertical lines (just like the dotted line above), so we can use a vertical line (parallel with y axis) to sweep from left to right. What we do now is to get the length information (several lines adding up together) which can be solved by using segment tree.

#### Core code

```c++
void pushup(int o) {
    if (tree[o].inv > 0) {
        tree[o].sum = tree[o].r-tree[o].l; // tree[o] is already fully occupied
    } else {
        tree[o].sum = tree[o<<1].sum+tree[o<<1|1].sum; // tree[o] is not fully occupied, calculate their sons' value
    }
}
```

#### Coding details (difference with regular process of building segment tree)

```c++
void build(int o, int l, int r) {  
    tree[o].inv = 0;
    tree[o].l = yval[l];
    tree[o].r = yval[r];
    tree[o].sum = 0;
    if (r-l == 1) // **** represent an interval
        return;
    int m = (l+r)>>1;
    build(o<<1, l, m); build(o<<1|1, m, r); // difference
}
```



#### Code

```c++

/*
http://acm.hdu.edu.cn/showproblem.php?pid=1542
Line sweep + Segment tree
Reference: https://oi-wiki.org/geometry/scanning/
Maintain the Y-value information by using segment tree
*/
#include <iostream>
#include <cstring>
#include <algorithm>
#include <cmath>
#include <cstdio>
#include <vector>
#include <queue>
#include <set>
#include <map>
#include <unordered_map>
#include <stack>
#include <deque>

#define ll long long
#define ull unsigned long long

using namespace std;

const int MOD = 1e9+7;
const int INF = 0x3f3f3f3f;
const int PI = acos(-1.0);

const int N = 20000 + 5;

int n;
struct Tree {
    double l, r;
    double sum;
    int inv;
} tree[N<<3];

struct Node {
    int flag;
    double x, y1, y2;
} node[N<<3];

double yval[N<<3];

bool cmp (Node n1, Node n2) {
    return n1.x < n2.x;
}

void build(int o, int l, int r) {  
    tree[o].inv = 0;
    tree[o].l = yval[l];
    tree[o].r = yval[r];
    tree[o].sum = 0;
    if (r-l == 1) // 表示一个区间，而不能是一个点
        return;
    int m = (l+r)>>1;
    build(o<<1, l, m); build(o<<1|1, m, r); // 注意不同
}

void pushup(int o) {
    if (tree[o].inv > 0) {
        tree[o].sum = tree[o].r-tree[o].l; // 区间有线段，算一次
    } else {
        tree[o].sum = tree[o<<1].sum+tree[o<<1|1].sum; // 区间无线段，计算子区间的和
    }
}

void add(int o, double l, double r, int c) { // 点修改
    if (l==tree[o].l && r==tree[o].r) {
        tree[o].inv += c; // 加入线段次数
        pushup(o); 
        return;
    }
    if (l < tree[o<<1].r) add(o<<1, l, min(tree[o<<1].r, r), c); // 区间add
    if (r > tree[o<<1|1].l) add(o<<1|1, max(l, tree[o<<1|1].l), r, c); // 区间add
    pushup(o);
}

int main() {
    int temp = 1;
    while ((scanf("%d", &n)!=EOF)) {
        double x1, x2, y1, y2;
        for (int i = 0; i < n; ++i) {
            scanf("%lf %lf %lf %lf", &x1, &y1, &x2, &y2);
            node[i].x = x1; node[i].y1 = y1; node[i].y2 = y2; node[i].flag = 1;
            node[i+n].flag = -1; node[i+n].x = x2; node[i+n].y1 = y1; node[i+n].y2 = y2;
            yval[i] = y1;
            yval[i+n] = y2;
        }
        sort(yval, yval+2*n); // 用来建树
        sort(node, node+2*n, cmp);
        build(1, 0, 2*n-1);
        add(1, node[0].y1, node[0].y2, node[0].flag);
        double ans = 0;
        for (int j = 1; j < 2*n; ++j) {
            ans += (node[j].x-node[j-1].x) * tree[1].sum;
            add(1, node[j].y1, node[j].y2, node[j].flag);
        }
        printf("%.0lf", ans);
    }
	return 0;
}

```



[problem]: http://acm.hdu.edu.cn/showproblem.php?pid=1542
