# DFS

## 剪枝

策略：

1. 优化搜索顺序：大部分情况下优先搜索分支较少的节点
2. 排除等效冗余：即不要重复搜索组合数
3. 可行性剪枝：当前状态不合法可以提前退出
4. 最优性剪枝：前面某次搜索的结果比当前状态更优，可以剪枝
5. 记忆化搜索（DP）

做法：

1. 枚举每个数要分到哪一组，这种的话没有等效冗余，如[小猫爬山](#小猫爬山)、[分成互质组](https://www.acwing.com/problem/content/1120/)
2. 枚举哪一组应该放进哪些数，这种需要优化等效冗余（需要数组标记），如[数独](#数独)、[木棒](#木棒)

## [小猫爬山](https://www.acwing.com/problem/content/167/)

思路：可以用1、3、4进行优化。对于1来说，先放重的猫分支数会比较少

```c++
#include<iostream>
#include<algorithm>
using namespace std;

const int N = 20;
int n, m, res;
int sum[N], w[N];

void dfs(int u, int len){
    // 最优性剪枝
    if(len >= res)  return;
    if(u == n){
        res = len;
        return;
    }
    for(int i = 0; i < len; i++){
        // 可行性剪枝
        if(sum[i] + w[u] > m)   continue;
        sum[i] += w[u];
        dfs(u + 1, len);
        sum[i] -= w[u];
    }
    sum[len++] = w[u];
    dfs(u + 1, len);
    sum[len] = 0;
}

int main(){
    cin >> n >> m;
    for(int i = 0; i < n; i++)  cin >> w[i];
    // 优化搜索顺序：优先搜索节点少的分支
    sort(w, w + n);
    reverse(w, w + n);
    
    res = n;
    dfs(0, 0);
    cout << res << endl;
    return 0;
}
```





## [数独](https://www.acwing.com/problem/content/168/)

思路：优化搜索顺序，先放备选最少的数

```c++
#include<iostream>
using namespace std;

const int N = 9, M = 1 << N;
char str[85];
int row[N], col[N], cell[3][3], ones[M], map[M];

void init(){
    for(int i = 0; i < N; i++)  row[i] = col[i] = M - 1;
    for(int i = 0; i < 3; i++)
        for(int j = 0; j < 3; j++)
            cell[i][j] = M - 1;
}

void flip(int x, int y, int t){
    int v = 1 << t;
    col[x] ^= v;
    row[y] ^= v;
    cell[x / 3][y / 3] ^= v;
}

int get(int x, int y){
    return col[x] & row[y] & cell[x / 3][y / 3];
}

int lowbit(int x){
    return x & -x;
}

bool dfs(int cnt){
    if(!cnt)    return true;
    int minv = 10;
    int x, y;
    for(int i = 0, k = 0; i < N; i++)
        for(int j = 0; j < N; j++, k++)
            if(str[k] == '.'){
                int state = get(i, j);
                // 优化搜索顺序：先放备选最少的数
                if(ones[state] < minv){
                    minv = ones[state];
                    x = i, y = j;
                }
            }
    
    int state = get(x, y);
    // 遍历在(x,y)位置上备选的数
    for(int i = state; i; i -= lowbit(i)){
        int t = map[lowbit(i)];
        flip(x, y, t);
        str[x * N + y] = t + '1';
        if(dfs(cnt - 1))    return true;
        flip(x, y, t);
        str[x * N + y] = '.';
    }
    return false;
}

int main(){
    // 映射每一位上的1代表实际的数字几
    for(int i = 0; i < N; i++)  map[1 << i] = i;
    // 记录每个数有多少个1
    for(int i = 0; i < M; i++)
        for(int j = 0; j < N; j++)
            ones[i] += i >> j & 1;
    
    while(cin >> str, str[0] != 'e'){
        init();
        int cnt = 0;
        for(int i = 0, k = 0; i < N; i++)
            for(int j = 0; j < N; j++, k++){
                if(str[k] == '.')   cnt++;
                else{
                    int t = str[k] - '1';
                    flip(i, j, t);
                }
            }
        dfs(cnt);
        puts(str);
    }
    return 0;
}
```



## [木棒](https://www.acwing.com/problem/content/169/)

思路：优化搜索顺序可以，先放长度较长的木棍。最优性剪枝，从小到大枚举木棒长度。可行性剪枝，如果所有木棍的长度和除以木棒长度后有余数，一定不能成功

排除等效冗余：

1. 如果安置一条木棍失败了，那么后面与它等长的木棍必然失败
2. 如果放置在木棒起始位置的木棒失败了，那么该状态下所有的后继状态必然失败
3. 如果放置在木棒末尾位置的木棒失败了，那么该状态下所有的后继状态必然失败

```c++
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 65;
int w[N];
int n, sum, len;
bool st[N];

bool dfs(int cnt, int cur, int start){
    if(cnt * len == sum)    return true;
    if(cur == len)  return dfs(cnt + 1, 0, 0);
    for(int i = start; i < n; i++){
        if(st[i] || cur + w[i] > len)    continue;
        st[i] = true;
        if(dfs(cnt, cur + w[i], i + 1)) return true;
        st[i] = false;
        // 排除等效冗余2,3
        if(!cur || cur + w[i] == len)    return false;
        // 排除等效冗余1
        int j = i;
        while(j + 1 < n && w[j + 1] == w[i]) j++;
        i = j;
    }
    return false;
}

int main(){
    while(cin >> n, n){
        memset(st, 0, sizeof st);
        sum = 0;
        for(int i = 0; i < n; i++){
            cin >> w[i];
            sum += w[i];
        }
        // 优化搜索顺序
        sort(w, w + n);
        reverse(w, w + n);
        
        len = 1;
        while(true){
            // 可行性剪枝
            if(sum % len == 0 && dfs(0, 0, 0)){
                cout << len << endl;
                break;
            }
            // 最优性剪枝
            len++;
        }
    }
    return 0;
}
```



# 迭代加深

适用：搜索层数很深，但是答案可能在浅层。

思路：设定一个搜索的最大深度，超过这个深度就不再搜索直接返回。一层一层搜索。和BFS不同的是，迭代加深只需要O(N)的空间

## [加成序列](https://www.acwing.com/problem/content/172/)

```c++
#include <iostream>
using namespace std;

const int N = 110;
int path[N];
int n;

bool dfs(int u, int depth){
    if(u == depth)    return path[u - 1] == n;
    
    bool st[N] = {0};
    for(int i = u - 1; i >= 0; i--)
        for(int j = i; j >= 0; j--){
            int s = path[i] + path[j];
            // 可行性剪枝
            if(s > n || s <= path[u - 1] || st[s])   continue;
            // 排除等效冗余
            st[s] = true;
            path[u] = s;
            if(dfs(u + 1, depth))   return true;
        }
    return false;
}

int main(){
    path[0] = 1;
    while(cin >> n, n){
        int depth = 1;
        while(!dfs(1, depth))   depth++;
        for(int i = 0; i < depth; i++)  cout << path[i] << " ";
        cout << endl;
    }
    return 0;
}
```



# 双向DFS

思路：直接DFS搜索空间太大，因此先搜索一半，记录下来。再搜索另一半的时候查表#

## [送礼物](https://www.acwing.com/problem/content/173/)

注意点：

1. 提前判断。因为可能溢出，所以提前判断可以让传入的参数类型是`int`而不需要开`long long`

```c++
#include <iostream>
#include <algorithm>
using namespace std;
typedef long long LL;

const int N = 48;
int n, m, k;
int w[N], cnt, sum[1 << 25], res;

int uniq(){
    int i = 0;
    for(int j = 1; j < cnt; j++){
        if(sum[i] == sum[j])    continue;
        sum[++i] = sum[j];
    }
    return i + 1;
}

void dfs1(int u, int s){
    if(u == k){
        sum[cnt++] = s;
        return;
    }
    dfs1(u + 1, s);
    if((LL)s + w[u] <= m)   dfs1(u + 1, s + w[u]);
}

void dfs2(int u, int s){
    if(u == n){
        int l = 0, r = cnt - 1;
        while(l < r){
            int mid = (l + r + 1) >> 1;
            if((LL)s + sum[mid] <= m)    l = mid;
            else    r = mid - 1;
        }
        res = max(res, s + sum[l]);
        return;
    }
    dfs2(u + 1, s);
    if((LL)s + w[u] <= m) dfs2(u + 1, s + w[u]);
}

int main(){
    cin >> m >> n;
    for(int i = 0; i < n; i++)  cin >> w[i];
    sort(w, w + n);
    reverse(w, w + n);
    
    cnt = 1;
    k = n / 2 + 2;
    dfs1(0, 0);
    sort(sum, sum + cnt);
    cnt = uniq();
    
    dfs2(k, 0);
    cout << res << endl;
    return 0;
}
```



# IDA\*（基于迭代加深的A\*算法）

思路：定义一个估价函数（小于等于真实值），表示从当前状态到终止状态还需要多少步。如果`估价函数+当前深度>预定义最大深度`，则进行剪枝。

## [排书](https://www.acwing.com/problem/content/182/)

思路：每次移动一段书到另一个位置，一共改变了三个位置的后缀。因此我们定义`估价函数=不正确的位置数之和除以3向上取整`。然后就可以愉快地使用IDA\*了

```c++
#include<iostream>
#include<cstring>
using namespace std;

const int N = 16;
int n;
int w[5][N], p[N];

int f(){
    int tot = 0;
    for(int i = 0; i < n - 1; i++)
        if(p[i + 1] != p[i] + 1)    tot++;
    return (tot + 2) / 3;
}

bool dfs(int depth, int maxDepth){
    if(depth + f() > maxDepth)  return false;
    if(f() == 0)    return true;
    
    for(int len = 1; len < n; len++) // 枚举连续一段书的长度
        for(int l = 0; l + len - 1 < n; l++){
            int r = l + len - 1;
            for(int k = r + 1; k < n; k++){
                memcpy(w[depth], p, sizeof p);
                int x = l;
                for(int i = r + 1; i <= k; i++, x++) p[x] = w[depth][i];
                for(int i = l; i <= r; i++, x++)    p[x] = w[depth][i];
                if(dfs(depth + 1, maxDepth))    return true;
                memcpy(p, w[depth], sizeof p);
            }
        }
    return false;
}

int main(){
    int T;
    cin >> T;
    while(T--){
        cin >> n;
        for(int i = 0; i < n; i++)  cin >> p[i];
        int depth = 0;
        while(depth < 5 && !dfs(0, depth))  depth++;
        if(depth >= 5)  puts("5 or more");
        else    cout << depth << endl;
    }
}
```



## [回转游戏](https://www.acwing.com/problem/content/description/183/)

思路：每次操作会使中间的8个数其中一个移除，另一个移入，因此一次操作最多减少一个不同的数。加入剪枝，即当前的操作不执行与上一步操作效果相反的操作

```c++
/*
      1     2
      3     4
5  6  7  8  9   10  11
      12    13
14 15 16 17 18  19  20
      21    22
      23    24
*/
#include<iostream>
#include<cstring>
using namespace std;

const int N = 25;
int a[N];
int op[8][7] = {
    {1, 3, 7, 12, 16, 21, 23},
    {2, 4, 9, 13, 18, 22, 24},
    {11, 10, 9, 8, 7, 6, 5},
    {20, 19, 18, 17, 16, 15, 14},
    {24, 22, 18, 13, 9, 4, 2},
    {23, 21, 16, 12, 7, 3, 1},
    {14, 15, 16, 17, 18, 19, 20},
    {5, 6, 7, 8, 9, 10, 11},
};
int oppsite[] = {5, 4, 7, 6, 1, 0, 3, 2};
int center[] = {7, 8, 9, 12, 13, 16, 17, 18};
char path[100];
int sum[4];

// 估价函数：8个数中最多的数的个数为res，那么至少还要经过8-res步操作才有可能达到目标状态
int f(){
    memset(sum, 0, sizeof sum);
    for(int i = 0; i < 8; i++)  sum[a[center[i]]]++;
    int res = 0;
    for(int i = 1; i < 4; i++)  res = max(res, sum[i]);
    return 8 - res;
}

void work(int o[]){
    int t = a[o[0]];
    for(int i = 0; i < 6; i++)  a[o[i]] = a[o[i + 1]];
    a[o[6]] = t;
}

bool dfs(int depth, int maxDepth, int last){
    if(depth + f() > maxDepth)  return false;
    if(f() == 0) return true;
    for(int i = 0; i < 8; i++){
        if(oppsite[i] == last)   continue;
        work(op[i]);
        path[depth] = 'A' + i;
        if(dfs(depth + 1, maxDepth, i)) return true;
        work(op[oppsite[i]]);
    }
    return false;
}

int main(){
    while(cin >> a[1], a[1]){
        for(int i = 2; i < N; i++)  cin >> a[i];
        memset(path, 0, sizeof path);
        int depth = 0;
        while(!dfs(0, depth, -1))   depth++;
        if(!depth)  puts("No moves needed");
        else{
            puts(path);
        }
        cout << a[center[0]] << endl;
    }
}

```

