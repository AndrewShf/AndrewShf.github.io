---
layout: default
title: 清理班次2(dp+segment tree)
date: 2020-04-23 11:23
post-link:
---

### [problem description(acwing-296)][problem]

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



[problem]: <https://www.acwing.com/problem/content/298/>