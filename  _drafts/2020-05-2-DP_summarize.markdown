---
layout: default
title: DP problems
date: 2020-04-26 11:23
post-link:
---

[TOC]



## Bitmask DP



### [hats wearing ][lc5387]

***Problem Description:***

​	There are `n` people and 40 types of hats labeled from 1 to 40.		Given a list of list of integers `hats`, where `hats[i]` is a list of all hats preferred by the `i-th` person.Return the number of ways that the n people wear different hats to each other.Since the answer may be too large, return it modulo `10^9 + 7`.



***Solution:***

​	To get the maximum times of match in a Bipartite graph. We can represent the status like this,

​	 DP(the i-th people)(the status). We use bit mask to represent the status, like 10101...01. 1 means the hat is already chosen, 0 means the hat is not yet chosen. 

​	However, the maximum number of hats is 40, the space complexity can be up to O(2^40) . The number of people is less than 15. We change the DP equivalence to this,

```c++
if (j & (1<<val)) {
	    // i th of hat, j ranges from 0~2^10
        dp[i][j] = (dp[i-1][j-(1<<val)]+dp[i][j])%(MOD); 
}
```



***Code:***

```c++
const int MOD = 1e9+7;
class Solution {
public:
    int numberWays(vector<vector<int>>& people) {
        vector<vector<int>> hats;
        map<int, vector<int>> mp;
        for (int i = 0; i < people.size(); ++i) {
            for (int j:people[i])
                mp[j].push_back(i);
        }
        for (int i = 1; i <= 40; ++i) {
            if (mp.count(i)) {
                sort(mp[i].begin(), mp[i].end());
                hats.push_back(mp[i]);
            }
        }
        int h = hats.size();
        int p = people.size();
        long long dp[42][(1<<10)+5];
        memset(dp, 0, sizeof(dp));
        dp[0][0] = 1;
        for (int i = 1; i <= h; ++i) {
            for (int j = 0; j < (1<<p); ++j) {
                for (int k = 0; k < hats[i-1].size(); ++k) {
                    int val = hats[i-1][k];
                    if (j & (1<<val)) {
                       		/* j-(1<<val) == j^(1<<val)*/
                            dp[i][j] = (dp[i-1][j^(1<<val)]+dp[i][j])%(MOD);
                    }
                }
                dp[i][j] = (dp[i][j]+dp[i-1][j]) % MOD;
            }
        }
        return dp[h][(1<<p)-1];
    }
};
```



## Digit DP



### [不要62][hdu2089]

***Problem Description:***

​	4 and 62, the 2 numbers are recognized as ominous numbers in some culture. With a given interval **[n, m]**, please identify how many numbers are **not** ominous.

​	For example, 62315, 73418, 88914, the 3 numbers are all ominous.

​	n <= m <= 1e6

***Solution***

1. **Simple solution**: brutal force, we can use c++ library to help us identify which number contains '4' or '62'.
2. **Fast solution**: **digit dp** (illustrate the process by an example)
   + To get the answer from this interval [0, 456]
   + we use dp(i)(j) to record calculated interval, like 0~99, 0~999, 0~9999, 0~99999..., which can dramatically save the time consumption.  The first one (i) means the total digits of the upper bound, the second (j) one means whether the digit before the highest digit of this number is six. (if it is six, then this digit can not be '2').
   + Now we calculate [0,456]. the highest digit of this number is 4. there is no digit before; and this interval is not recorded in dp, so we need to divide this to a smaller problem. we divide it to interval [0, 099], [100, 199], [200, 299], [300, 399], [400, 456].
   + We move to the lower digit from the highest digit '4', we can see the first 4 intervals have the the same answers, since they all contain the interval [0, 99].
   + For the fifth interval [0, 56] (move to  a lower digit). We can use dfs to deal with this subproblem. 
   + See the comment in code-2 for more details.

***Code***

```c++
// Simple solution  time consumption: 432 ms
const int maxn=1e6+5;
char a[]="4",b[]="62";
int s[maxn];
char str[8];
void init(){
    for (int i=1; i<1e6; i++) {
        sprintf(str,"%d",i);
        if (strstr(str, a)!=NULL||strstr(str, b)!=NULL) s[i]=0;
        else s[i]=1;
    }
}
int main(){
    init();
    int m,n;
    while (scanf("%d%d",&m,&n)==2&&(m||n)) {
        int sum=0;
        for (int i=m; i<=n; i++) {
            sum+=s[i];
        }
        printf("%d\n",sum);
    }
    return 0;
}
```



```c++
// digit dp  
//time consumption: 15 ms
int num[10];
int d[10][2];
/* limit indicates whether this interval is like 0~99..9 */
/* cur marks which digit it is now processed */
int dfs(int cur,bool wassix,bool limit){
    int ans=0;
    if(cur==0) return 1;//cur=0 indicates the end of recursion
    /* have we calculated the interval before ? */
    if(!limit&&d[cur][wassix]!=-1) 
        return d[cur][wassix];
    int maxnum=limit==0?9:num[cur];
    /* divide to subproblems */
    for (int i=0; i<=maxnum; i++) {
        /* not-ok */
        if(i==4||(i==2&&wassix)) continue;
        /* move to a lower digit, was six ?,  a limit ?*/
        else ans += dfs(cur-1, i==6, i==maxnum&&limit);
    }
    if(limit==0) d[cur][wassix]=ans;
    return ans;
}
int cal(int n){
    int len=0;
    while (n) {
        num[++len]=n%10;
        n/=10;
    }
    return dfs(len,0,1);
}
int main(){
    int m,n;
    memset(d, -1, sizeof(d));
    while (scanf("%d%d",&m,&n)==2&&(m||n)) {
        printf("%d\n",cal(n)-cal(m-1));//注意是m-1
    }
    return 0;
}
```



### [XHXJ's LIS][hdu4352]



**Problem Description:**

​	The problem is: Determine how many numbers have the power value k in [L，R] in O(1)time. If you consider the number as a string and can get a longest strictly increasing subsequence the length of which is equal to k. We name k the power value of this number in this question.

​	First a integer T(T<=10000),then T lines follow, every line has three positive integer L,R,K.(
0<L<=R<2 63-1 and 1<=K<=10).	

**Solution:**

​	This problem can be solved by using 3-dimensional DP: dp(i)(j)(k), where i represents the digit, j represents the LIS status.

​	For me, this question is quite hard. If you don't know digit dp, please refer to the first problem in this section.

```c++
/*
http://acm.hdu.edu.cn/showproblem.php?pid=4352
XHXJ's LIS

LIS + bitmask dp + digit dp
*/
#define ll long long
#define ull unsigned long long

using namespace std;

const int MOD = 1e9+7;
// const int INF = 0x7fffffff;
const int INF = 0x3f3f3f3f;
const int PI = acos(-1.0);

ll L, R, K;
int num[20];
ll dp[20][(1<<10)+5][15];

int decode(int status) {
    int ans = 0;
    while (status) {
        if (status&1) ans++;
        status >>= 1;
    }
    return ans;
}

int encode(int x, int status) { // O(nlogn) LIS solution
    for (int i = x; i < 10; ++i) {
        if (status & (1<<i)) return (status^(1<<i)) | (1<<x); // replace i with x; the pos(i) >= pos(x)
    }
    return status | (1<<x);
}

ll dfs(int pos, int status, int isub, int iszero) {
    if (pos == 0)
        return decode(status)==K; // decode the status to see if it is satisfied
    if (!isub && dp[pos][status][K]!=-1)  // return stored dp value ?
        return dp[pos][status][K];
    int mx = isub==0 ? 9:num[pos];
    ll ans = 0;
    for (int i = 0; i <= mx; ++i) {
        // is the former digits are all zero ? does it have upper bound ? the status transformation ?
        ans += dfs(pos-1, (i==0&&iszero) ? 0:encode(i, status), isub&&i==mx, iszero&&i==0);
    }
    if (!isub) dp[pos][status][K] = ans;
    return ans;
}

ll cal(long long val) {
    int len = 0;
    while (val) {
        num[++len] = val%10;
        val /= 10;
    }
    return dfs(len, 0, 1, 1);
}

int main() {
    int t;
    scanf("%d", &t);
    int kase = 0;
    memset(dp, -1, sizeof(dp));
    while (t--) {
        scanf("%lld%lld%lld", &L, &R, &K);
        printf("Case #%d: %lld\n", ++kase, cal(R)-cal(L-1));
    }
	return 0;
}
```



### Round Numbers

**[Problem Description][poj3252]**

​	They have thus resorted to "round number" matching. The first cow picks an integer less than two billion. The second cow does the same. If the numbers are both "round numbers", the first cow wins,
otherwise the second cow wins.

​	A positive integer *N* is said to be a "round number" if the binary representation of *N* has as many or more zeroes than it has ones. For example, the integer 9, when written in binary form, is 1001. 1001 has two zeroes and two ones; thus, 9 is a round number. The integer 26 is 11010 in binary; since it has two zeroes and three ones, it is not a round number.

​	Obviously, it takes cows a while to convert numbers to binary, so the winner takes a while to determine. Bessie wants to cheat and thinks she can do that if she knows how many "round numbers" are in a given range.

​	*Help her by writing a program that tells how many round numbers appear in the inclusive range given by the input (1 ≤ Start < Finish ≤ 2,000,000,000).*

**Solution**

​	Same with Decimal-digit dp problems. Use 3-dimensional dp to solve the problem, dp(i)(j)(k). The first one is the position of the digit, the second one is the count of (number of zeros - number of ones). the last one is to identify whether the digits before are all zero (attention!). For example 1001, 001 (the first one is ok, and the last one is not ok, even they share the same last 2 digits).

**Code**

```c++

/*
POJ 3252
Round Numbers
*/
#define ll long long
#define ull unsigned long long

using namespace std;

const int MOD = 1e9+7;
// const int INF = 0x7fffffff;
const int INF = 0x3f3f3f3f;
const int PI = acos(-1.0);

const int offset = 35;
ll L, R;
int num[35];
ll dp[35][70][2]; // the last dimension is to identify whether the digits before are all zero

ll dfs(int pos, bool waszero, bool isup, int zeros) {
    ll ans = 0;
    // special case for '0' and '1'
    if (pos==1 && waszero && (isup==0||num[pos]==1)) return 0;
    if (pos==0) return (zeros>=0) ? 1:0;
    if (!isup && dp[pos][zeros+offset][waszero]!=-1) return dp[pos][zeros+offset][waszero];
    int mx = isup ? num[pos]:1;
    for (int i = 0; i <= mx; ++i) {
        ans += dfs(pos-1, i==0&&waszero, i==mx&&isup, (i==0&&waszero)==1 ? 0:(zeros-(i==0 ? -1:1)));
    }
    if (!isup) dp[pos][zeros+offset][waszero] = ans;
    return ans;
}

ll cal(long long val) {
    if (val == 0) return 0;
    int len = 0;
    while (val) {
        num[++len] = val%2;
        val >>= 1;
    }
    return dfs(len, 1, 1, 0);
}

int main() {
    memset(dp, -1, sizeof(dp));
    while (scanf("%lld%lld", &L, &R)==2) {
        printf("%lld\n", cal(R)-cal(L-1));
    }
	return 0;
}
```



### Balanced number

**[Problem description][hdu3709]**

​	A balanced number is a non-negative integer that can be balanced if a pivot is placed at some digit. More specifically, imagine each digit as a box with weight indicated by the digit. When a pivot is placed at some digit of the number, the distance from a digit to the pivot is the offset between it and the pivot. Then the torques of left part and right part can be calculated. It is balanced if they are the same. A balanced number must be balanced with the pivot at some of its digits. For example, 4139 is a balanced number with pivot fixed at 3. The torqueses are 4\*2 + 1\*1 = 9 and 9\*1 = 9, for left part and right part, respectively. It's your job to calculate the number of balanced numbers in a given range [x, y].	



***Solution***

​	We use dp(pos)(pivot_pos)(pre_val) to represent the status. And select each position as the pivot when finding the answer.



***Code***

```c++
ll L, R;
int num[30];
ll dp[20][20][2000];

ll dfs(int pos, int pre, int pivot, bool isup) {
    ll ans = 0;
    if (pre < 0) return 0; // cut this branch!
    if (pos == 0) return pre==0; // is this number balanced ?
    if (dp[pos][pivot][pre]!=-1 && !isup) return dp[pos][pivot][pre];
    int end = isup ? num[pos]:9;
    for (int i = 0; i <= end; ++i) {
        ans += dfs(pos-1, pre+(pos-pivot)*i, pivot, isup && i==end); // calculate weight!
    }
    if (!isup) dp[pos][pivot][pre] = ans;
    return ans;
}

ll cal(long long val) {
    if (val < 0) return 0;
    int len = 0;
    while (val) {
        num[++len] = val%10;
        val /= 10;
    }
    num[0] = 0;
    ll ans = 0;
    for (int i = len; i > 0; --i) { // to select each position as the pivot
        ans += dfs(len, 0, i, 1);
    }
    return ans-(len-1); // 0, 00, 000 this kind of value are calculated more than one times!
}

int main() {
    memset(dp, -1, sizeof(dp));
    int T;
    scanf("%d", &T);
    while (T--) {
        scanf("%lld%lld", &L, &R);
        printf("%lld\n", cal(R)-cal(L-1));
    }
	return 0;
}
```



### B-number

**[problem description][hdu3652]**

​	A wqb-number, or B-number for short, is a non-negative integer whose decimal form contains the sub- string "13" and can be divided by 13. For example, 130 and 2613 are wqb-numbers, but 143 and 2639 are not. Your task is to calculate how many wqb-numbers from 1 to n for a given integer n.



***Solution***

​	How to represent the status? To accurately represent the status, to give the previous status information to the rest digits, We should know which digit (the position) is now processing in the current dfs, is the previous digit contains '13' (contains or not will make a difference to the rest of the recursive procedure), was the previous digit '1'? And we should know the mod13 of the previous number.

***Code***

```c++
#define ll long long
#define ull unsigned long long

using namespace std;

const int MOD = 1e9+7;
// const int INF = 0x7fffffff;
const int INF = 0x3f3f3f3f;
const int PI = acos(-1.0);

ll L, R;
int len;
int num[20];
/* represent the status: which position, is the previous digit one ? 
   is the the digits before contain '13'? the mod of 13 of the previous number
*/
ll dp[15][2][2][15];

ll dfs(int pos, bool wasone, bool isup, bool ok, int m) {
    ll ans = 0;
    if (pos == 0 && ok) {
        return m==0;
    } else if (pos == 0) return 0;
    if (!isup && dp[pos][wasone][ok][m]!=-1) return dp[pos][wasone][ok][m];
    int end = isup ? num[pos]:9;
    // cout << "end: " << end << endl;
    for (int i = 0; i <= end; ++i) {
        ans += dfs(pos-1, i==1, isup&&i==end, (wasone&&i==3)||(ok), (m*10+i)%13);
    }
    if (!isup) dp[pos][wasone][ok][m] = ans;
    return ans;
}

ll cal(long long val) {
    len = 0;
    while (val) {
        num[++len] = val%10;
        val /= 10;
    }
    num[0] = 0;
    ll ans = 0;
    return dfs(len, 0, 1, 0, 0);
}

int main() {
    memset(dp, -1, sizeof(dp));
    while (scanf("%lld", &R) != EOF) {
        printf("%lld\n", cal(R));
    }
	return 0;
}
```



### Similar problems

[hdu3555 bomb][hdu3555]



[lc5387]: <https://leetcode-cn.com/contest/biweekly-contest-25/problems/number-of-ways-to-wear-different-hats-to-each-other/>
[hdu2089]:<https://vjudge.net/problem/HDU-2089>
[hdu4352]: <https://vjudge.net/problem/HDU-4352>
[hdu3555]:<https://vjudge.net/problem/HDU-3555>
[poj3252]:<https://vjudge.net/problem/POJ-3252>
[hdu3709]:<https://vjudge.net/problem/HDU-3709>
[hdu3652]:<https://vjudge.net/problem/HDU-3652>