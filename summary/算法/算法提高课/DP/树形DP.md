# 树形DP

[简单的树形DP](../../算法竞赛进阶指南/5.4树形DP.md)

## [树的最长路径](https://www.acwing.com/problem/content/1074/)

![树的最长路径](树形DP1.jpg)

思路：f[u]定义为以u为根的，只往一个方向走到底的最长链的长度(即绿色链)。红色链是最长路径，由以u为根的绿色链+一条第二长的链



```c++
#include <iostream>
#include <cstring>
using namespace std;

const int N = 10010, M = N * 2;
int n, ans;
int h[N], e[M], ne[M], w[M], idx;

void add(int a, int b, int c){
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}
// 只返回绿色链
int dfs(int x, int parent){
    int dist = 0;
    int d1 = 0, d2 = 0;
    for(int i = h[x]; ~i; i = ne[i]){
        int j = e[i];
        if(j == parent) continue;
        int d = dfs(j, x) + w[i];
        if(d >= d1)  d2 = d1, d1 = d;
        else if(d > d2) d2 = d;
        dist = max(dist, d);
    }
    ans = max(ans, d1 + d2);
    return dist;
}

int main(){
    cin >> n;
    memset(h, -1, sizeof h);
    for(int i = 1; i < n; i++){
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c), add(b, a, c);
    }
    dfs(1, -1);
    cout << ans << endl;
    return 0;
}
```



## [树的中心](https://www.acwing.com/problem/content/1075/)

思路：对于某个节点来说，该点到最远的点是两个中的最大值：往子节点方向走的最大值、往父节点方向走的最大值

1. 往子节点方向的最大值求法与树的最长直径的最长链求法相同，因此需要**维护最长链**
2. 往父节点方向的最大值求法：有两种情况，父节点继续向上，或者父节点向下。其中向下时，父节点的最长链可能通过该点，而这样是不正确的，若通过该点应选择第二长的链，因此需要**维护第二长链**
3. 分析后可知，一个节点需要三个信息：最长链、次长链、父节点方向的最长链

```c++
#include <iostream>
#include <cstring>
using namespace std;

const int N = 10010, M = N * 2, INF = 0x3f3f3f3f;
int n;
int h[N], w[M], e[M], ne[M], idx;
int d1[N], d2[N], up[N], pointing[N];

void add(int a, int b, int c){
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}

int dfs_down(int u, int parent){
    for(int i = h[u]; ~i; i = ne[i]){
        int j = e[i];
        if(j == parent) continue;
        // 子节点更新父节点信息，所以需要先求子节点信息
        int d = dfs_down(j, u) + w[i];
        // pointing让父节点u指向子节点j，表示u向下的最长链经过j
        if(d >= d1[u]) d2[u] = d1[u], d1[u] = d, pointing[u] = j;
        else if(d > d2[u])   d2[u] = d;
    }
    return d1[u];
}

void dfs_up(int u, int parent){
    for(int i = h[u]; ~i; i = ne[i]){
        int j = e[i];
        if(j == parent) continue;
        // 用父节点u的信息更新子节点j的信息，所以需要先求父节点信息
        // 如果u向下的最长链经过j，应选次长链来取最大值
        if(pointing[u] == j)    up[j] = max(up[u], d2[u]) + w[i];
        else    up[j] = max(up[u], d1[u]) + w[i];
        dfs_up(j, u);
    }
}

int main(){
    memset(h, -1, sizeof h);
    scanf("%d", &n);
    for(int i = 1; i < n; i++){
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c), add(b, a, c);
    }
    
    dfs_down(1, -1);
    dfs_up(1, -1);
    int res = INF;
    for(int i = 1; i <= n; i++)
        res = min(res, max(d1[i], up[i]));
    printf("%d\n", res);
    return 0;
}
```





## [数字转换](https://www.acwing.com/problem/content/1077/)

[树的最长直径](#树的最长直径)的应用

埃氏筛法，时间复杂度O(NlogN)

```c++
#include <iostream>
#include <cstring>
using namespace std;

const int N = 50010;
int n, res;
int h[N], e[N], ne[N], idx;
int sum[N];
bool st[N];

void add(int a, int b){
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

int dfs(int u){
    int d1 = 0, d2 = 0;
    for(int i = h[u]; ~i; i = ne[i]){
        int j = e[i];
        int d = dfs(j) + 1;
        if(d >= d1) d2 = d1, d1 = d;
        else if(d > d2) d2 = d;
    }
    res = max(res, d1 + d2);
    return d1;
}

int main(){
    cin >> n;
    memset(h, -1, sizeof h);
    for(int i = 1; i <= n; i++)
        for(int j = 2; j <= n / i; j++)
            sum[i * j] += i;
    for(int i = 2; i <= n; i++)
        if(sum[i] < i){
            add(sum[i], i);
            st[i] = true;
        }else   st[i] = true;
    
    for(int i = 1; i <= n; i++)
        if(!st[i])
            dfs(i);       
    cout << res << endl;
    return 0;
}
```



## [二叉苹果树](https://www.acwing.com/problem/content/1076/)

[有依赖的背包问题](https://www.acwing.com/problem/content/10/)的简化版

为什么能看成分组背包+树形DP呢？我们把每一棵子树son按照体积划分，有`f[son][0],f[son][1],...,f[son][m]`等选择方式，因此每一棵子树都可以看成一个组。

```c++
#include <iostream>
#include <cstring>
using namespace std;

const int N = 110, M = N * 2;
int n, m;
int h[N], e[M], ne[M], w[M], idx;
int f[N][N];

void add(int a, int b, int c){
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u, int parent){
    for(int i = h[u]; ~i; i = ne[i]){
        int son = e[i];
        if(son == parent)  continue;
        dfs(son, u);
        for(int j = m; j > 0; j--)
            for(int k = 0; k < j; k++)
                // 当前节点与子节点之间的边必选，要为该边留一个位置
                f[u][j] = max(f[u][j], f[u][j - k - 1] + f[son][k] + w[i]);
    }
}

int main(){
    cin >> n >> m;
    memset(h, -1, sizeof h);
    for(int i = 1; i < n; i++){
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c), add(b, a, c);
    }
    dfs(1, -1);
    cout << f[1][m] << endl;
    return 0;
}
```





下面的是状态机+树形DP，还算简单的。

## [战略游戏](https://www.acwing.com/problem/content/325/)

思路：一条边至少需要子节点或父节点其中之一选中它。`f[u][0]:以u为根节点，不放士兵；f[u][1]:放士兵`，注意这里是有向边！设s1,s2,...是u的子节点

1. f(u, 0) = f(s1, 1) + f(s2, 1) +...
2. f(u, 1) = min(f(s1, 0), f(s1, 1)) + min(f(s2, 0), f(s2, 1)) + ...

```c++
#include <iostream>
#include <cstring>
using namespace std;

const int N = 1510;
int n, root;
int h[N], e[N], ne[N], idx;
int f[N][2];
bool st[N];

void add(int a, int b){
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u){
    f[u][0] = 0;
    f[u][1] = 1;
    for(int i = h[u]; ~i; i = ne[i]){
        int j = e[i];
        dfs(j);
        f[u][0] += f[j][1];
        f[u][1] += min(f[j][0], f[j][1]);
    }
}

int main(){
    while(scanf("%d", &n) != -1){
        memset(h, -1, sizeof h);
        idx = 0;
        memset(st, 0, sizeof st);
        int u, cnt;
        for(int i = 0; i < n; i++){
            scanf("%d:(%d)", &u, &cnt);
            while(cnt--){
                int v;
                scanf("%d", &v);
                add(u, v); // 题意给的是有向边，因此会有根节点
                st[v] = true;
            }
        }
        for(int i = 0; i < n; i++)
            if(!st[i])  root = i;
        dfs(root);
        printf("%d\n", min(f[root][0], f[root][1]));
    }
    return 0;
}
```



## [皇宫看守](https://www.acwing.com/problem/content/1079/)

思路：和战略游戏有点像，战略游戏要求一条边上的两个节点至少选择其中之一，而本题要求的是每个节点的父节点、子节点、本节点至少选择其中之一。是[监控二叉树](../../算法竞赛进阶指南/5.4树形DP.md#监控二叉树)的强化版。假设i表示当前节点，`s1, s2, s3`表示i的子节点，设计3个状态：

1. f(i, 0)表示i节点上不放守卫且不被子节点看到，则`f(i, 0) = f(s1, 1) + f(s2, 1) + f(s3, 1)`
2. f(i, 1)表示i节点上不放守卫且被子节点看到，表示子节点中至少一个是放守卫的，则`f(i, 1) = min( f(s1, 2) + min(f(s2, 1), f(s2, 2)) + min(f(s3, 1), f(s3, 2)),  f(s2, 2) + min(f(s1, 1), f(s1, 2)) + min(f(s3, 1), f(s3, 2)), f(s3, 2) + min(f(s1, 1), f(s1, 2)) + min(f(s2, 1), f(s2, 2)) )`
3. f(i, 2)表示i节点上放守卫，则`f(i, 2) = min(f(s1, 0), f(s1, 1), f(s1, 2)) + min(f(s2, 0), f(s2, 1), f(s2, 2)) + min(f(s3, 0), f(s3, 1), f(s3, 2)) + w[i]`

最后就是需要考虑叶节点的处理，对于叶节点来说，`f(i, 0)=0，f(i, 1)=inf，f(i, 2) = w[i]`。第二步的实现和叶节点的处理具体看实现

```c++
#include <iostream>
#include <cstring>
using namespace std;
typedef long long LL;

const int N = 1510, INF = 0x3f3f3f3f;
int n;
int h[N], e[N], ne[N], idx, w[N];
LL f[N][3];
bool st[N];

void add(int a, int b){
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u){
    f[u][2] = w[u];
    LL diff = INF;
    for(int i = h[u]; ~i; i = ne[i]){
        int j = e[i];
        dfs(j);
        f[u][0] += f[j][1]; // u的叶节点过多会溢出，故开long long
        f[u][1] += min(f[j][1], f[j][2]); // 先将所有子节点的值取最小值加起来
        f[u][2] += min(f[j][0], min(f[j][1], f[j][2]));
        diff = min(diff, f[j][2] - min(f[j][1], f[j][2])); // 至少要有一个子节点放守卫，因此枚举每个子节点。如果子节点的f[j][1]小于f[j][2]，则表示f[u][1]中在第j个节点选的是f[j][1]，所以要把f[j][2] - f[j][1]的差值加上；否则在第j个节点选的是f[j][2]，差值为0，加不加效果一样
    }
    if(diff != INF) f[u][1] += diff; // 加上差值。diff等于inf表示位于叶节点
    if(f[u][1] == 0)    f[u][1] = INF; // 对于叶节点，f[u][1]不会更新故等于0，应置为非法状态即inf
}

int main(){
    scanf("%d", &n);
    memset(h, -1, sizeof h);
    for(int i = 1; i <= n; i++){
        int u, v, t, cnt;
        scanf("%d%d%d", &u, &t, &cnt);
        w[u] = t;
        while(cnt--){
            scanf("%d", &v);
            add(u, v);
            st[v] = true;
        }
    }
    int root = 1;
    while(st[root]) root++;
    dfs(root);
    printf("%lld\n", min(f[root][1], f[root][2]));
    return 0;
}
```



# [换根DP](https://www.luogu.com.cn/blog/sflsrick/note-how-to-change-root)

## [树的中心](#树的中心)

## [树中距离之和](https://leetcode-cn.com/problems/sum-of-distances-in-tree/)



