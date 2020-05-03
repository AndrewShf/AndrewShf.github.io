---
layout: default
title: Segment Tree Problems
date: 2020-5-1 11:23
post-link:
---

## SGT + DP

### [problem description(acwing-296)][problemdp]

#### Brief illustration

![](/images/dp_sgt.png#center){:height="300px" width="300px"}

#### Solution

1. Sort (by the end of each interval) before we can apply dynamic programming to this problem
2. dp[b[i]] = min(dp[b[i]], min(dp[a[i]-1]~dp[b[i]]-1)+c[i])
3. Build segment tree to record dp value in each interval so that we can query the wanted value at log(n) time complexity

#### Core code

```c++
int start = cow[i].start, end = cow[i].end, salary = cow[i].salary;
int tmp = dp[end];
dp[end] = min(dp[end], query(1, m-1, e, start-1, end-1)+salary);
```

#### Coding details (difference with regular process of building segment tree)

```c++
dp[m-1] = 0; // dp[m-1] not dp[0]!!!!
build(1, m-1, e); // m-1 ~ e not 0 ~ e !!!!
```



#### Code

```c++
#define ll long long
#define ull unsigned long long

using namespace std;

const int INF = 0x3f3f3f3f;

const int N = 1e4+10;
const int M = 1e5;
int n, m, e;
int dp[M], tree[M<<2];

struct Cow {
    int start, end;
    int salary;
} cow[N];

bool cmp (Cow c1, Cow c2) {
    return c1.end < c2.end;
}

void build (int o, int l, int r) {
    if (l == r) {
        tree[o] = dp[l];
        return;
    }
    int m = (l+r)>>1;
    build(o<<1, l, m);
    build(o<<1|1, m+1, r);
    tree[o] = min(tree[o<<1], tree[o<<1|1]);
}

void update(int o, int l, int r, int pos, int val) {
    if (l == r) {
        tree[o] = val;
        return;
    }
    int m = (l+r)>>1;
    if (pos <= m)
        update(o<<1, l, m, pos, val);
    else
        update(o<<1|1, m+1, r, pos, val);
    tree[o] = min(tree[o<<1], tree[o<<1|1]);
}

int query(int o, int l, int r, int ql, int qr) {
    if (ql<=l && qr>=r) {
        return tree[o];
    }
    int m = (l+r)>>1;
    int ans = INF;
    if (ql <= m)
        ans = min(ans, query(o<<1, l, m, ql, qr));
    if (qr > m)
        ans = min(ans, query(o<<1|1, m+1, r, ql, qr));
    return ans;
}

int main() {
    scanf("%d%d%d", &n, &m, &e);
    m++; e++; // 1 ~ 
    for (int i = 0; i < n; ++i) {
        scanf("%d%d%d", &cow[i].start, &cow[i].end, &cow[i].salary);
        cow[i].start++; cow[i].end++;
    }
    sort(cow, cow+n, cmp);
    memset(dp, 0x3f, sizeof(dp));
    dp[m-1] = 0; // dp[m-1] not dp[0]!!!!
    build(1, m-1, e); // m-1 ~ e not 0 ~ e !!!!
    for (int i = 0; i < n; ++i) {
        // query minimal
        int start = cow[i].start, end = cow[i].end, salary = cow[i].salary;
        int tmp = dp[end];
        dp[end] = min(dp[end], query(1, m-1, e, start-1, end-1)+salary);
        // update tree
        if (dp[end] != tmp)
            update(1, m-1, e, end, dp[end]);
    }
    if (dp[e] == INF)
        puts("-1");
    else
        printf("%d\n", dp[e]);
	return 0;
}
```



## SGT + Geometry

### [problem description(hdu-1542)][problemgeo]

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



[problemgeo]: http://acm.hdu.edu.cn/showproblem.php?pid=1542
[problemdp]: https://www.acwing.com/problem/content/298/

