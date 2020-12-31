# 单调队列优化DP

## 数组模拟队列

```c++
const int N = 10010;
int q[N], hh = 0, tt = 0;
q[++tt] = x; // 队尾插入元素
q[hh]; // 队头
hh++; // 弹出队头
if(hh <= tt){ // 判断队列是否为空
}
// tt初始化有两种：1）初始化为-1，表示初始化一个空的队列；2）初始化为0，表示放一个哨兵，该值不影响结果
```



## 整理

有两种情况：

1. 一段区间里只取一个和当前位置处理

   直接单调队列，非常简单。[最大子序和](#最大子序和)、[烽火传递](#烽火传递)、[绿色通道](#绿色通道)、[跳跃游戏 VI](https://leetcode-cn.com/problems/jump-game-vi/)

2. 一段区间里取m-1个和当前位置处理

## [最大子序和](https://www.acwing.com/problem/content/137/)

思路：先求出前缀和，假设当前处于i，则sum[i]已经确定，要想结果最大，则维护一个区间的和的最小值即可。子序列的长度最多为m，但因为我们维护的是**前缀和**（sum[i] - sum[j - 1]才是区间[i, j]的和），因此维护的单调队列长度为m。

```c++
#include <iostream>
using namespace std;

const int N = 300010;
int n, m;
int q[N], sum[N];

int main(){
    scanf("%d%d", &n, &m);
    for(int i = 1; i <= n; i++){
        scanf("%d", &sum[i]);
        sum[i] += sum[i - 1];
    }
    int hh = 0, tt = 0, res = sum[1] - sum[0];
    for(int i = 1; i <= n; i++){
        if(i - q[hh] > m)   hh++;
        res = max(res, sum[i] - sum[q[hh]]);
        while(hh <= tt && sum[i] <= sum[q[tt]]) tt--;
        q[++tt] = i;
    }
    printf("%d\n", res);
    return 0;
}
```



## [烽火传递](https://www.acwing.com/problem/content/1091/)

思路：f[i]表示点燃第i个烽火台，且前i个烽火台满足条件的最少代价，则有f[i] = min(f[i - k]) + w[i]。k的取值需要仔细分析，现在第i个烽火台点燃了，就是说i前面的m个烽火台只要有一个点燃即可（不包括第i个烽火台），因此需要维护一个区间长度为m的单调队列。

```c++
#include <iostream>
using namespace std;

const int N = 200010;
int n, m;
int w[N], q[N], f[N];

int main(){
    scanf("%d%d", &n, &m);
    for(int i = 1; i <= n; i++) scanf("%d", &w[i]);
    
    int hh = 0, tt = 0;
    for(int i = 1; i <= n; i++){
        if(i - q[hh] > m)   hh++;
        f[i] = f[q[hh]] + w[i];
        while(hh <= tt && f[i] <= f[q[tt]])  tt--;
        q[++tt] = i;
    }
    int res = 1e9;
    for(int i = n - m + 1; i <= n; i++) // 最后连续m个烽火台中取最小值为结果
        res = min(res, f[i]);
    printf("%d\n", res);
    return 0;
}
```



## [绿色通道](https://www.acwing.com/problem/content/1092/)

补充：整理二分的模板和适用情况

二分有两种：

1. 判断某个值是否存在，其实可以统一到第二种情况中，不统一的话加A[mid] == target返回即可
2. 判断某个值的左边界或者右边界

模板

```java
int leftBound(int[] A, int target){
    int l = 0, r = A.length;
    while(l < r){
        int mid = (l + r) >>> 1;
        if(check(mid))	r = mid;
        else	l = mid + 1;
    }
    return l; // 如果没有找到target，则返回小于target的最大的数的索引；找到的话则也可以解读为小于target的数有多少个
    // return l == A.lenght ? -1 : A[l] == target ? l : -1;
}

int rightBound(int[] A, int target){ // 搜索右边界
    int l = 0, r = A.length;
    while(l < r){
        int mid = (l + r) >>> 1;
        if(check(mid))	l = mid + 1; // 找右边界
        else	r = mid;
    }
    return l - 1; // 如果没有找到target，则返回小于target的最大的数的索引
    // return l == 0 ? -1 : A[l] == target ? l - 1 : -1;
}
```

思路：二分+烽火传递。和烽火传递稍微不同的是，空题段表示的是长度为len的都没有写

```c++
#include <iostream>
#include <cstring>
using namespace std;

const int N = 50010;
int n, m;
int w[N], f[N], q[N];

bool check(int len){
    memset(f, 0, sizeof f);
    int hh = 0, tt = 0;
    for(int i = 1; i <= n; i++){
        if(i - q[hh] > len + 1) hh++; // len+1而不是len，因为空题段长度是len，所以只要len+1的区间里有一题做了都是满足要求的
        f[i] = f[q[hh]] + w[i];
        while(hh <= tt && f[i] <= f[q[tt]]) tt--;
        q[++tt] = i;
    }
    for(int i = n - len; i <= n; i++) // 空题段len，有效区间长度是len+1
        if(f[i] <= m)
            return true;
    return false;
}

int main(){
    scanf("%d%d", &n, &m);
    for(int i = 1; i <= n; i++) scanf("%d", &w[i]);
    int l = 0, r = n;
    while(l < r){
        int mid = (l + r) >> 1;
        if(check(mid))  r = mid;
        else    l = mid + 1;
    }
    printf("%d", r);
    return 0;
}
```



## [修剪草坪](https://www.acwing.com/problem/content/1089/)

思路：f[i]表示安排到第i头牛时的最大效率值。1）不选择第i头牛，f[i] = f[i - 1]；2）选择第i头牛，那么就有f[i] = sum[i] - sum[j] + f[j - 1]，包括i且区间长度为m的最大值，即[i - m + 1, i]，但是用了**前缀和**，j要取到i-m才能求出这段区间和，单调队列中将f[j - 1] - sum[j]打包在一起，因此维护区间长度为m的单调队列。

```c++
#include <iostream>
using namespace std;
typedef long long LL;

const int N = 100010;
int n, m;
LL sum[N], f[N];
int q[N];

LL get(int j){
    return f[j - 1] - sum[j];
} 

int main(){
    scanf("%d%d", &n, &m);
    for(int i = 1; i <= n; i++){
        scanf("%lld", &sum[i]);
        sum[i] += sum[i - 1];
    }
    
    int hh = 0, tt = 0;
    for(int i = 1; i <= n; i++){
        if(hh <= tt && i - q[hh] > m)   hh++;
        f[i] = max(f[i - 1], get(q[hh]) + sum[i]);
        while(hh <= tt && get(i) >= get(q[tt]))  tt--;
        q[++tt] = i;
    }
    printf("%lld\n", f[n]);
    return 0;
}
```



## [理想的正方形](https://www.acwing.com/problem/content/1093/)

思路：先按行滑动窗口，再按列在得到的窗口上再次滑动窗口。包括i且区间长度为k的最值，即[i - k + 1, i]，所以有判断条件`i - q[hh] >= k`（当然也可以写成`i - q[hh] > k - 1`）

```c++
#include <iostream>
using namespace std;

const int N = 1010;
int n, m, k;
int w[N][N], row_max[N][N], row_min[N][N], a[N], b[N], c[N];
int q[N];

void get_max(int a[], int w[], int n){
    int hh = 0, tt = 0;
    q[hh] = 1;
    a[1] = w[1];
    for(int i = 2; i <= n; i++){
        if(i - q[hh] >= k)   hh++;
        a[i] = max(w[q[hh]], w[i]);
        while(hh <= tt && w[i] >= w[q[tt]]) tt--;
        q[++tt] = i;
    }
}

void get_min(int a[], int w[], int n){
    int hh = 0, tt = 0;
    q[hh] = 1;
    a[1] = w[1];
    for(int i = 2; i <= n; i++){
        if(i - q[hh] >= k)   hh++;
        a[i] = min(w[q[hh]], w[i]);
        while(hh <= tt && w[i] <= w[q[tt]]) tt--;
        q[++tt] = i;
    }
}

int main(){
    scanf("%d%d%d", &n, &m, &k);
    for(int i = 1; i <= n; i++)
        for(int j = 1; j <= m; j++)
            scanf("%d", &w[i][j]);
    
    // 按行取最大值和最小值
    for(int i = 1; i <= n; i++){
        get_max(row_max[i], w[i], m);
        get_min(row_min[i], w[i], m);
    }
    int res = 1e9;
    // 按列取最大值和最小值
    for(int i = k; i <= m; i++){
        for(int j = 1; j <= n; j++) c[j] = row_max[j][i];
        get_max(a, c, n);
        for(int j = 1; j <= n; j++) c[j] = row_min[j][i];
        get_min(b, c, n);
        for(int j = k; j <= n; j++)
            res = min(res, a[j] - b[j]);
    }
    cout << res << endl;
    return 0;
}
```

