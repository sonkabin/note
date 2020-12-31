# 状态压缩DP

分为两类：

1. 棋盘式
2. ？

## 棋盘式

### [小国王](https://www.acwing.com/problem/content/1066/)

```c++
#include <iostream>
#include <vector>
using namespace std;
typedef long long LL;

const int N = 11, M = 1 << N, K = 110;
int n, m;
/*
两个国王不攻击的条件有(假设当前行状态是a, 上一行状态是b):
1. 每行都没有相邻的1
2. 同一列没有同时放国王, a & b == 0
3. 某列放了国王，上一行的左右两列不能放国王, a | b没有相邻的1
*/
LL f[N][K][M]; // 第i行，放k个国王，当前行状态为m
vector<int> state; // 存放所有合法状态
vector<int> h[M]; // 存放合法的可转移状态
int cnt[M]; // 存放某个合法状态的一行有多少个国王

bool check(int state){
    for(int i = 0; i < n; i++)
        if((state >> i & 1) && (state >> i + 1 & 1))
            return false;
    return true;
}

int count(int state){
    int res = 0;
    for(int i = 0; i < n; i++)
        res += state >> i & 1;
    return res;
}

void lazy(){ // 偷懒写法
    f[0][0][0] = 1;
    for(int i = 1; i <= n + 1; i++) // 枚举到n + 1行
        for(int j = 0; j <= m; j++)
            for(int k = 0; k < state.size(); k++){
                int a = state[k]; // 取出下标对应的状态
                for(int t : h[k]){
                    int b = state[t], c = cnt[a];
                    if(j >= c) // 国王数不能小于当前行要放的国王数
                        f[i][j][a] += f[i - 1][j - c][b];
                }
            }

    cout << f[n + 1][m][0] << endl; // 放到第n + 1行，放了m个棋子，第n + 1行状态等于0，表示第n + 1行不放棋子，和原问题等价
}

void naive(){ // 朴素的枚举
    f[0][0][0] = 1;
    for(int i = 1; i <= n; i++)
        for(int j = 0; j <= m; j++)
            for(int k = 0; k < state.size(); k++){
                int a = state[k]; // 取出下标对应的状态
                for(int t : h[k]){
                    int b = state[t], c = cnt[a];
                    if(j >= c)
                        f[i][j][a] += f[i - 1][j - c][b];
                }
            }
        
    LL res = 0;
    for(int i = 0; i < state.size(); i++) // 统计第n行的每个状态下的方案数
        res += f[n][m][state[i]];
    cout << res << endl;
}

int main(){
    cin >> n >> m;
    for(int i = 0; i < 1 << n; i++)
        if(check(i)){
            state.push_back(i);
            cnt[i] = count(i);
        }
    
    for(int i = 0; i < state.size(); i++)
        for(int j = 0; j < state.size(); j++){
            int a = state[i], b = state[j]; // a, b是合法状态
            if((a & b) == 0 && (check(a | b)))
                h[i].push_back(j); // 这里存的是下标
        }
    
    // naive();
    lazy();
    return 0;
}
```

### [玉米田](https://www.acwing.com/problem/content/329/)

```c++
#include <iostream>
#include <vector>
using namespace std;

/*
种玉米的条件有(假设当前行状态是a, 上一行状态是b):
1. 每行没有相邻的1
2. 同一列没有同时种玉米, 即 a & b == 0
3. 土地是肥沃的, 即a & g[i] == 0，因为g[i]为1的位表示不育，若a对应的位也是1(表示种),那么a & g[i] != 0，表示非法
*/
const int N = 14, M = 1 << N, mod = 1e8;
int g[N];
int n, m;
vector<int> state;
vector<int> h[M];
int f[N][M];

bool check(int state){
    for(int i = 0; i < m; i++)
        if((state >> i & 1) && (state >> i + 1 & 1))
            return false;
    return true;
}

int main(){
    cin >> n >> m;
    for(int i = 1; i <= n; i++)
        for(int j = 0; j < m; j++){
            int t;
            cin >> t;
            if(!t)  g[i] |= 1 << j;
        }
    
    for(int i = 0; i < 1 << m; i++)
        if(check(i))
            state.push_back(i);
            
    for(int i = 0; i < state.size(); i++)
        for(int j = 0; j < state.size(); j++){
            int a = state[i], b = state[j];
            if((a & b) == 0)
                h[i].push_back(j);
        }
    
    f[0][0] = 1;
    for(int i = 1; i <= n + 1; i++)
        for(int j = 0; j < state.size(); j++){
            int a = state[j];
            if(a & g[i]) continue;
            for(int k : h[j]){
                int b = state[k];
                f[i][a] = (f[i][a] + f[i - 1][b]) % mod;
            }
        }
        
    cout << f[n + 1][0] << endl;
    return 0;
}
```

### [炮兵阵地](https://www.acwing.com/problem/content/294/)

```c++
#include <iostream>
#include <vector>
using namespace std;

/*两支炮兵不攻击的条件有(假设当前行状态是a, 上一行状态是b, 再上一行状态是c):
1. 同一行不相互攻击
2. 同一列不同时有炮兵, 即 a & b == 0且b & c == 0且a && c == 0
3. 不能在山地上, 即a & g[i] == 0
*/
const int N = 110, M = 1 << 11;
int n, m;
int g[N], cnt[M];
int f[2][M][M];
vector<int> state;

bool check(int state){
    for(int i = 0; i < m; i++)
        if((state >> i & 1) && ((state >> i + 1 & 1) || (state >> i + 2 & 1)))
            return false;
    return true;
}

int count(int state){
    int res = 0;
    for(int i = 0; i < m; i++)
        res += state >> i & 1;
    return res;
}

int main(){
    cin >> n >> m;
    for(int i = 1; i <= n; i++)
        for(int j = 0; j < m; j++){
            char c;
            cin >> c;
            if(c == 'H')    g[i] |= 1 << j;
        }
    
    for(int i = 0; i < 1 << m; i++)
        if(check(i)){
            state.push_back(i);
            cnt[i] = count(i);
        }
        
    for(int i = 1; i <= n + 2; i++)
        for(int j = 0; j < state.size(); j++){
            int a = state[j];
            if((a & g[i]) != 0) continue;
            for(int k = 0; k < state.size(); k++)
                for(int u = 0; u < state.size(); u++){
                    int b = state[k], c = state[u];
                    if((a & b) || (a & c) || (b & c))   continue;
                    f[i & 1][a][b] = max(f[i & 1][a][b], f[i - 1 & 1][b][c] + cnt[a]);
                }
        }
    
    cout << f[n + 2 & 1][0][0] << endl;
    return 0;
}
```

## 集合类型的状压DP

[愤怒的小鸟](https://www.acwing.com/problem/content/526/)

分析题意：

1. 是[重复覆盖问题](#重复覆盖问题)，至少多少只小鸟能消灭所有小猪

2. 轨迹是抛物线，因此一条抛物线上不会存在x相同的小猪

3. 给出两点，计算抛物线方程

   $y_1 = ax_1^2+bx_1, y_2 = ax_2^2 + bx_2$, 联立有
   $$
   a = \frac{ \frac{y_1}{x_1} - \frac{y_2}{x_2} }{x_1 - x_2} \\ 
   b = (y_1 - ax_1^2) / x_1^2
   $$

4. 暴搜的思想

   ```c++
   void dfs(int state, int cnt){
       if(state的每一位都是1) return;
       选取其中一位不是1的
       枚举过该位置的抛物线, new_state
       res = min(res, dfs(new_state) + 1)
   }
   ```

   

```c++
#include <iostream>
#include <cstring>
#include <cmath>
using namespace std;

typedef pair<double, double> PDD;

const int N = 18, M = 1 << N;
double eps = 1e-6;
int n, m;
PDD loc[N];
int path[N][N];
int f[M];

int cmp(double a, double b){
    if(fabs(a - b) < eps)   return 0;
    return a > b ? 1 : -1;
}

int main(){
    int T;
    cin >> T;
    while(T--){
        cin >> n >> m;
        for(int i = 0; i < n; i++)  cin >> loc[i].first >> loc[i].second;
        
        memset(path, 0, sizeof path);
        for(int i = 0; i < n; i++){
            path[i][i] = 1 << i; // 提前处理，经过第i个点的抛物线必然经过第i个点，因为后面判断相等会continue
            for(int j = 0; j < n; j++){
                double x1 = loc[i].first, y1 = loc[i].second;
                double x2 = loc[j].first, y2 = loc[j].second;
                if(!cmp(x1, x2))    continue;
                double a = (y1 / x1 - y2 / x2) / (x1 - x2);
                double b = y1 / x1 - a * x1;
                if(a >= 0)   continue;
                int state = 0;
                for(int k = 0; k < n; k++){ // 求所有经过 第i点, 第j点 这两个点的抛物线经过的点
                    double x = loc[k].first, y = loc[k].second;
                    if(!cmp(a * x * x + b * x, y))  state |= 1 << k;
                }
                path[i][j] = state;
            }
        }
        
        memset(f, 0x3f, sizeof f);
        f[0] = 0;
        for(int i = 0; i + 1 < 1 << n; i++){
            int x = 0;
            for(int j = 0; j < n; j++)
                if(!(i >> j & 1)){ // 选取其中一位不是1的状态，表示该点没有被覆盖
                    x = j;
                    break;
                }
                
            for(int j = 0; j < n; j++)
                f[i | path[x][j]] = min(f[i | path[x][j]], f[i] + 1);
        }
        
        cout << f[(1 << n) - 1] << endl;
    }
    return 0;
}
```



## 补充

$$
\begin{vmatrix}
0 & 0 & 1 & 0 & 1 & 1 & 0 \\
1 & 0 & 0 & 1 & 0 & 0 & 1 \\
0 & 1 & 1 & 0 & 0 & 1 & 0 \\
1 & 0 & 0 & 1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 1 & 1 & 0 & 1 \\
\end{vmatrix}
$$



### 重复覆盖问题

最少选择几行使得每一列至少存在一个1

### 精确覆盖问题

是否能找到一个行的集合，使得集合中每一列都恰好包含一个1。举例来说：八皇后，数独