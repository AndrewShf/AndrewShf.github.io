---
layout: default
title: LeetCode-CN-2020春季个人赛
date: 2020-04-24 11:23
post-link:
---

### [contest location(spring solo contest )][problem]



#### [Problem 2: 传递信息][p2]

#### solution

​	用了类似dp的思想解决此问题

#### Code

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

#### solution

​	前缀和+二分搜索

#### Code

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

#### Solution 1

​	利用dp，核心代码如下：

```c++
dp[i] = i+jump[i]>=len ? 1:dp[i+jump[i]]+1;
for (int j = i+1; j<len && dp[j]>=dp[i]+1; ++j)
	dp[j] = dp[i]+1;
/*dp[j]>=dp[i]+1, to ensure the value at the right of i, is always smaller than dp[i]+2*/
```

#### Code 1

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

#### Solution 1

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

#### Solution 2 — 树形dp

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

#### Code 1 & 2

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

通过统计字符来判断嵌套数

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



[problem]: <https://leetcode-cn.com/contest/season/2020-spring/ranking/solo/>
[p2]: <https://leetcode-cn.com/contest/season/2020-spring/problems/chuan-di-xin-xi/>
[p3]: <https://leetcode-cn.com/contest/season/2020-spring/problems/ju-qing-hong-fa-shi-jian/>
[p4]:<https://leetcode-cn.com/contest/season/2020-spring/problems/zui-xiao-tiao-yue-ci-shu/>
[p5]: <https://leetcode-cn.com/contest/season/2020-spring/problems/er-cha-shu-ren-wu-diao-du/>
[wp3]:<https://leetcode-cn.com/problems/minimum-number-of-frogs-croaking/>