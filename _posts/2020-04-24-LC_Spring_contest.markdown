---
layout: default
title: LeetCode-CN-2020 Spring 
date: 2020-04-24 11:23
post-link:
---


### [Team - Contest Location][team]

#### [Problem 4 — 切分数组][p24]

***Solution***

​	一维DP可做; dp方程式为dp[i] = min(dp[j]) arr[j]是与arr[i]不互质的数。如何找到呢？ 质因数分解。

​	从前往后递归，对每个数质因数分解，更新，某个质因子所能具备的最小划分。

***Code 1 & Code 2***

```c++
/* 比赛时过了，之后过不了。。。 在质因数分解那里耗时过多*/
const int N = 1e6+5;
vector<int> primes[N];
bool check[N];
int pos[N];
int dp[N];

class Solution {
public:
    int splitArray(vector<int>& nums) {
        int len = nums.size();
        // 拆分质因数
        for (int i = 2; i < N; ++i) {
            if (check[i] == 0) {
                for (int j = i; j < N; j+=i) {
                    primes[j].push_back(i);
                    check[j] = 1;
                }
            }
        }
        dp[1] = 1;
        memset(pos, 0x3f, sizeof(pos));
        for (int i = 1; i <= len; ++i) {
            for (int j : primes[nums[i-1]]) pos[j] = min(pos[j], dp[i-1]); 
            dp[i] = i;
            for (int j : primes[nums[i-1]]) dp[i] = min(dp[i], pos[j]+1);
        }
        return dp[len];
    }
};
```

```c++
/* 重写了一份楼教主的代码～ */
/* 先质因数分解 */
const int INF = 0x3f3f3f3f;
const int N = 1e6+5;
int check[N];

class Solution {
public:
    int splitArray(vector<int>& nums) {
        int len = nums.size();
        // 拆分质因数
        for (int i = 2; i < N; ++i) {
            if (check[i] == 0) {
                for (int j = i; j < N; j+=i) { /* 记录一份就可 */
                    check[j] = i;
                }
            }
        }
        int ans = 0;
        vector<int> primes(N, INF);
        for (int i = 0; i < len; ++i) {
            int current = INF; /*在 i 处拆分的答案*/
            int val = nums[i];
            for (; val > 1;) {
                int x = check[val];
                primes[x] = min(primes[x], ans); // 在 i处拆分最大值也是ans（本次单独成组）
                current = min(current, primes[x]+1); // 找最前面的质因数
                for (; val%x==0; val/=x) ; /* 继续拆分看也没有其他质因数 */
            }
            ans = current; // 更新解
        }
        return ans;
    }
};
```



#### [Problem5-游乐园的迷宫][p25]

***Solution 1:***

​	思路与该大佬的解答一致，摘抄[该大佬的分析][p25s1]。

1. 选取一个凸包顶点。
2. 在下图该凸包中，可以发现，从当前红点顺时针走到下一个点，之后无论走哪里都是向左拐； 同理，逆时针走到下一个点，无论之后走到哪里都是向右拐。

![](/images/convex_geometry.png#center){:height="300px" width="300px"}

3. 假设向左拐，即选择顺时针走到下一个点。红点访问过，去掉，此时转向步骤2，改点当作红点。
   1. 因为没有三点共线,横(纵)坐标最小(大)的点一定在凸包上,随便选一个即可。
   2. 利用向量叉积，并且不断更新找到顺时针走向上凸包的点（或者逆时针走向上凸包的点）（怎么找？ 不断更新即可）

***Solution 2:***

​	摘抄自[网上大佬的贪心构造][p25s2]

​	在这种余裕最大的情况下，要使得下一步仍是余裕最大，就应使得上述条件在下一步仍成立。考虑下一步的转向：

1. 如果是 L，那么将剩余未经过的点相对于 y 按照逆时针排序，选取第一个作为下一步到达的点。
2. 如果是 R，那么将剩余未经过的点相对于 y 按照顺时针排序，选取第一个作为下一步到达的点。
   通过画图可以看出，这样的贪心选择保证了下一步局面的最优性。下面以当前步为 L 为例画出了示意图。

![](/images/leetcode-playground-greedy.jpg#center){:height="300px" width="300px"}

​	其实本质效果和第一个方法是一致的。

***Code 1 & Code 2***

```c++
/* 按照上述思路重写的凸包+叉积 */
class Solution {
public:
    vector<int> visitOrder(vector<vector<int>>& points, string direction) {
        int cur = 0;
        int len = points.size();
        // find the first point on convex hull
        for (int i = 0; i < len; ++i) if (points[i][0] < points[cur][0]) cur = i;
        vector<int> ans;
        ans.push_back(cur);
        vector<int> vis(len+5, 0);
        vis[cur] = 1;
        for (int i = 0; i < len-2; ++i) {
            char dir = direction[i];
            int next = -1; // find next point on convext hull
            for (int j = 0; j < len; ++j) {
                if (vis[j]) continue;
                if (next == -1) next = j;
                else {
                    if (dir == 'L') {
                        // Cross product
                        if ((points[next][0]-points[cur][0])*(points[j][1]-points[cur][1])-(points[next][1]-points[cur][1])*(points[j][0]-points[cur][0]) < 0) {
                            next = j;
                        }
                    } else {
                        if ((points[next][0]-points[cur][0])*(points[j][1]-points[cur][1])-(points[next][1]-points[cur][1])*(points[j][0]-points[cur][0]) > 0) {
                            next = j;
                        }
                    }
                }
            }
            cur = next; // update cur point !
            vis[cur] = 1;
            ans.push_back(cur);
        }
        // put the ending point into ans
        for (int i = 0; i < len; ++i) if (vis[i] == 0) ans.push_back(i);
        return ans;
    }
};
```



```c++
// 听说暴搜可过，附上网上大佬的暴搜解法
class Solution:
    def visitOrder(self, points: List[List[int]], direction: str) -> List[int]:
        n = len(points)
        ans = []
        v = [1]*n
        def judge(s:int,m:int,e:int,tot:int)->bool:
            l1x, l1y = points[m][0]-points[s][0],points[m][1]-points[s][1]
            l2x, l2y = points[e][0]-points[m][0],points[e][1]-points[m][1]
            d = l1x*l2y-l1y*l2x
            if d >0 and direction[tot]=='L':
                return True
            if d<0 and direction[tot]=='R':
                return True
            return False
            
        def dfs(s:int,m:int,e:int,tot:int)->bool:
            if tot == n:
                return True
            if tot==0:
                for i in range(n):
                    v[i] = 0
                    if dfs(n,n,i,1):
                        ans.append(i)
                        return True
                    v[i] = 1
            if tot == 1:
                for i in range(n):
                    if v[i]:
                        v[i] = 0
                        if dfs(n,e,i,2):
                            ans.append(i)
                            return True
                        v[i] = 1
            if tot >= 2:
                flag = 0
                for i in range(n):
                    if v[i] and judge(m,e,i,tot-2):
                        flag = 1
                        v[i] = 0
                        if dfs(m,e,i,tot+1):
                            ans.append(i)
                            return True
                        v[i] = 1
                if not flag:
                    return False   
                    
        if dfs(n,n,n,0):
            ans.reverse()
            return ans

作者：longint1024
链接：https://leetcode-cn.com/problems/you-le-yuan-de-mi-gong/solution/bei-bi-de-wai-xiang-ren-bao-sou-by-longint1024/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

### [contest location(spring solo contest )][problem]

#### [Problem 2: 传递信息][p2]

***Solution***

​	用了类似dp的思想解决此问题

***Code***

```c++
class Solution {
public:
    int numWays(int n, vector<vector<int>>& relation, int k) {
        int dp[15][15][10];
        memset(dp, 0, sizeof(dp));
        for (int i = 0; i < relation.size(); ++i) {
            int m = relation[i][0];
            int n = relation[i][1];
            dp[m][n][1] = 1;
        }
        for (int iter = 2; iter <= k; ++iter) {
            for (int i = 0; i < n; ++i) {
                for (int j = 0; j < n; ++j) {
                    for (int r = 0; r < n; ++r) {
                        int m = 1;
                            dp[i][j][iter] += dp[i][r][m]*dp[r][j][iter-m];
                    }
                }
            }
        }
        return dp[0][n-1][k];
    }
};
```



#### [Problem 3: 剧情触发时间][p3]

***Solution***

​	前缀和+二分搜索

***Code***

```c++
class Solution {
public:
    bool ok (int x, int y, int z, int a, int b, int c) {
        return ((x<=a) && (y<=b) && (z<=c));
    }
    vector<int> getTriggerTime(vector<vector<int>>& increase, vector<vector<int>>& requirements) {
        int len = increase.size();
        for (int i = 1; i < increase.size(); ++i) {
            increase[i][0] += increase[i-1][0];
            increase[i][1] += increase[i-1][1];
            increase[i][2] += increase[i-1][2];
        }
        vector<int> ans;
        for (int i = 0; i < requirements.size(); ++i) {
            if (requirements[i][0]==0 && requirements[i][1]==0 && requirements[i][2]==0) {
                ans.push_back(0);
                continue;
            }
            int l = -1, r = len-1;
            // cout << r << endl;
            while (r-l > 1) {
                int m = (l+r) >> 1;
                if (ok(requirements[i][0], requirements[i][1], requirements[i][2], increase[m][0], increase[m][1], increase[m][2])) {
                    r = m;
                } else {
                    l = m;
                }
            }
            if (ok(requirements[i][0], requirements[i][1], requirements[i][2], increase[r][0], increase[r][1], increase[r][2])) {
                ans.push_back(r+1);   
            } else {
                ans.push_back(-1);
            }
        }
        return ans;
    }
};
```



#### [Problem4: 最小跳跃数][p4]

***Solution 1***

​	利用dp，核心代码如下：

```c++
dp[i] = i+jump[i]>=len ? 1:dp[i+jump[i]]+1;
for (int j = i+1; j<len && dp[j]>=dp[i]+1; ++j)
	dp[j] = dp[i]+1;
/*dp[j]>=dp[i]+1, to ensure the value at the right of i, is always smaller than dp[i]+2*/
```

***Code 1***

```c++
/*
https://leetcode-cn.com/problems/zui-xiao-tiao-yue-ci-shu/
Reverse-DP
*/
class Solution {
public:
    int minJump(vector<int>& jump) {
        int len = jump.size();
        int* dp = new int[len+5];
        dp[len-1] = 1;
        for (int i = len-2; i >= 0; --i) {
            dp[i] = i+jump[i]>=len ? 1:dp[i+jump[i]]+1;
            for (int j = i+1; j<len && dp[j]>=dp[i]+1; ++j)
                dp[j] = dp[i]+1;
        }
        return dp[0];
    }
};
```



#### [Problem5: 二叉树任务调度][p5]

***Solution 1***

最开始offset为0，即可用时间为0；如果要执行根节点任务，会额外消耗时间。

当offset>根节点所要消耗的时间时，无需额外消耗时间了。记得offset要减去相应的时间。

当左子树总时间和右子树总时间之差在offset时间之内，代表，两子树可以并行执行并且少执行offset/2的时间

```c++
ans += (R+L-offset)/2.0;
```

当不满足条件时，

```c++
else if (L > R) {
	ans += R;                   //一个cpu使用了R的时间
	work(root->left, R+offset); //，另一个cpu可并行使用的时间+R
} 
```

时间复杂度： O(N) x log(N)

***Solution 2 — 树形dp***

假设两个cpu都能一直工作（即一直并行）：time cost = TotalWeight / 2

一般化 = 可以并行的时间+只能串行的时间 = TotalWeight/2 + onlySerial/2

TotalWeight = 可以并行的时间+只能串行的时间/2

那么我们一直维护这两个值，就能找到答案了！

总时间(TotalWeight)好计算，如何计算只能串行的时间呢？

```c++
if(r.second > l.first) { // 更新
	ans.second = r.second - l.first; // 更新为右子树的串行时间（只有一个cpu在跑）-左子树的总时间（另一个cpu跑到这里来用）
    return ans;
}
if(l.second > r.first) { // 更新
	ans.second = l.second - r.first; // 更新为左子树的串行时间（只有一个cpu在跑）-右子树的总时间（另一个cpu跑到这里来用）
	return ans;
}
ans.second = 0; // 更新
```

回溯计算节点的串行时间时，需要更新，因为，单一子树的串行时间碰到另一子树就可能会抵消到一部分。

***Code 1 & 2***

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    double ans;
    double sum(TreeNode* root) {
        if (root == nullptr) return 0;
        double s = root->val;
        s += sum(root->left); s += sum(root->right);
        return s;
    }
    void work(TreeNode* root, double offset) {
        if (root == nullptr) return;
        ans += max(root->val-offset, 0.); // 总耗费时间
        offset = max(offset-root->val, 0.); // 可并行使用时间-刚消耗的时间
        double L = sum(root->left), R = sum(root->right);
        if (abs(R-L) <= offset) {  // R, L之差小于offset，offset是可以并行使用的cpu的时间
            ans += (R+L-offset)/2.0;
        } else if (L > R) {
            ans += R;                   //一个cpu使用了R的时间
            work(root->left, R+offset); //，另一个cpu可并行使用的时间+R
        } else if (R > L) {
            ans += L;                   // 一个cpu使用了L的时间
            work(root->right, L+offset);// 另一个cpu可并行使用的时间+L
        }
    }
    double minimalExecTime(TreeNode* root) {
        ans = 0; work(root, 0);
        return ans;
    }
};
```

```c++
#define pdd pair<double,double>
class Solution {
public:
    pdd calc(pdd l,pdd r) {
        pdd ans;
        ans.first = l.first + r.first; // 总和
        if(r.second > l.first) { // 更新
            ans.second = r.second - l.first;
            return ans;
        }
        if(l.second > r.first) { // 更新
            ans.second = l.second - r.first;
            return ans;
        }
        ans.second = 0; // 更新
        return ans;
    }

    pdd work(TreeNode *root) {
        if(root == NULL) return {0,0};
        pdd res = calc(work(root->left),work(root->right));
        res.first += root->val; //总和+
        res.second += root->val; // 串行+
        return res;
    }
    double minimalExecTime(TreeNode* root) {
        pdd ans = work(root);
        return 1.0 * ans.first/2 + 1.0 * ans.second/2;
    }
};
```



#### [Week Contest 185 Problem 3][wp3]

***Solution***

通过统计字符来判断嵌套数

***Code***
```c++
class Solution {
    int arr[10];
public:
    int minNumberOfFrogs(string croakOfFrogs) {
        memset(arr, 0, sizeof(arr));
        int cur = 0;
        for (char ch : croakOfFrogs) {
            if (ch == 'c') {
                cur = 0;
            } else if (ch == 'r') {
                cur = 1;
            } else if (ch == 'o') {
                cur = 2;
            } else if (ch == 'a') {
                cur = 3;
            } else if (ch == 'k') {
                cur = 4;
            }
            if (cur == 0) {
                if (arr[4] > 0) arr[4]--;
                arr[0]++;
            } else {
                if (arr[cur-1] == 0) return -1;
                else arr[cur-1]--;
                arr[cur]++;
            }
        }
        for (int i = 0; i < 4; ++i)
            if (arr[i] != 0) return -1;
        return arr[4];
    }
};
```


[p25s1]: <https://leetcode-cn.com/problems/you-le-yuan-de-mi-gong/solution/c-tu-bao-cha-ji-by-heltion/>
[p25s2]: <https://leetcode-cn.com/problems/you-le-yuan-de-mi-gong/solution/c-ji-jiao-pai-xu-by-zqy1018/>
[p25]: <https://leetcode-cn.com/problems/you-le-yuan-de-mi-gong/solution/>
[p24]:<https://leetcode-cn.com/contest/season/2020-spring/problems/qie-fen-shu-zu/>
[team]:<https://leetcode-cn.com/contest/season/2020-spring/ranking/team/>
[problem]: <https://leetcode-cn.com/contest/season/2020-spring/ranking/solo/>
[p2]: <https://leetcode-cn.com/contest/season/2020-spring/problems/chuan-di-xin-xi/>
[p3]: <https://leetcode-cn.com/contest/season/2020-spring/problems/ju-qing-hong-fa-shi-jian/>
[p4]:<https://leetcode-cn.com/contest/season/2020-spring/problems/zui-xiao-tiao-yue-ci-shu/>
[p5]: <https://leetcode-cn.com/contest/season/2020-spring/problems/er-cha-shu-ren-wu-diao-du/>
[wp3]:<https://leetcode-cn.com/problems/minimum-number-of-frogs-croaking/>