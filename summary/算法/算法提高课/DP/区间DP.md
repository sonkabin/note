# 区间DP

## 实现方式

1. 迭代式（常用于一维）

   ```c++
   for (int len = 1; len <= n; len++)
   	for(int l = 1; l + len - 1 <= n; l++) // r是l + len - 1是因为长度为len且已知左端点, r - l + 1 == len
   		r = l + len - 1;
   ```

   

2. 记忆化搜索

   比较好写嘻嘻

## 模板

### [石子合并](https://www.acwing.com/problem/content/284/)

思想：

状态表示：f[l, r]

- 集合：所有合并区间[l, r]的代价的方案
- 属性：min

状态计算：

找到最后一个不同的状态，假设最后一步是合并区间[l, k]与[k + 1, r]，那么`f[l, r] = f[l, k] + f[k + 1, r] + l~r的代价`。因此枚举k即可

时间复杂度：O(N^3)

代码：

```c++
#include <iostream>
#include <cstring>
using namespace std;

const int N = 310;
int s[N], w[N];
int n;
int f[N][N];

int main(){
    cin >> n;
    memset(f, 0x3f, sizeof f);
    for(int i = 1; i <= n; i++){
        cin >> w[i];
        s[i] = s[i - 1] + w[i];
        f[i][i] = 0;
    }
    for(int len = 2; len <= n; len++) // 一堆石子不用合并，已在上面置代价为0
        for(int l = 1; l + len - 1 <= n; l++){
            int r = l + len - 1;
            for(int k = l; k < r; k++)
                f[l][r] = min(f[l][r], f[l][k] + f[k + 1][r] + s[r] - s[l - 1]);
        }
    cout << f[1][n] << endl;
    return 0;
}
```



## 剖环成链

### [环形石子合并](https://www.acwing.com/problem/content/1070/)

朴素思想：枚举每一个缺口，进行区间dp，复杂度O(N^4)

**经典做法**：本质上是对n条长度为n的链进行区间dp，因此可以在首尾相连处加上一条新的链，时间复杂度O(2N^3)

代码：

```c++
#include <iostream>
#include <cstring>
using namespace std;

const int N = 410, inf = 0x3f3f3f3f;
int n;
int w[N], s[N];
int f[N][N], g[N][N];

int main(){
    memset(f, 0x3f, sizeof f);
    memset(g, -0x3f, sizeof g);
    cin >> n;
    for(int i = 1; i <= n; i++){
        int t;
        cin >> t;
        w[i + n] = w[i] = t;
    }
    for(int i = 1; i <= n * 2; i++){
        s[i] = s[i - 1] + w[i];
        f[i][i] = 0;
        g[i][i] = 0;
    }
    for(int len = 2; len <= n; len++)
        for(int l = 1; l + len - 1 < 2 * n; l++){ // 不需要枚举到2×n（因为从n开始长度为n的是2n-1处），不过枚举到也不会错
            int r = l + len - 1;
            for(int k = l; k < r; k++){
                f[l][r] = min(f[l][r], f[l][k] + f[k + 1][r] + s[r] - s[l - 1]);
                g[l][r] = max(g[l][r], g[l][k] + g[k + 1][r] + s[r] - s[l - 1]);
            }
        }
    int minv = inf, maxv = -inf;
    for(int i = 1; i <= n; i++){
        minv = min(minv, f[i][i + n - 1]);
        maxv = max(maxv, g[i][i + n - 1]);
    }
    cout << minv << endl << maxv << endl;
    return 0;
}
```



### [能量项链](https://www.acwing.com/problem/content/322/)



```c++
#include <iostream>
using namespace std;

const int N = 210;
int n;
int w[N], f[N][N];

int main(){
    cin >> n;
    for(int i = 1; i <= n; i++){ // 将(2,3),(3,5)改成2,3,5,2的形式,最终len = n + 1
        int t;
        cin >> t;
        w[i + n] = w[i] = t;
    }
    
    for(int len = 3; len <= n + 1; len++) // len至少为3才可合并(比如2,3,2才可以合并，而2,2不能合并)
        for(int l = 1; l + len - 1 <= 2 * n; l++){ // 从n开始到2*n，长度是n + 1
            int r = l + len - 1;
            for(int k = l + 1; k < r; k++)
                f[l][r] = max(f[l][r], f[l][k] + f[k][r] + w[l] * w[r] *w[k]);
        }
    
    int res = 0;
    for(int i = 1; i <= n; i++)
        res = max(res, f[i][i + n]);
    cout << res << endl;
    return 0;
}
```





## 记录方案

### [加分二叉树](https://www.acwing.com/problem/content/481/)

1. f[L, R]：中序遍历是区间[L, R]的所有二叉树的得分的最大值。将f[L, R]表示的二叉树按照根节点分类，则根节点在k时有得分f[L, k -  1] × f[k + 1, R] + w[k]
2. 此外，在计算每个状态的过程中，记录每个区间的最大值对应的根节点编号，后续用dfs即可求出前序遍历
3. 时间复杂度：状态总数是 O(N^2)，计算每个状态需要 O(N) 的计算量，因此总时间复杂度是 O(N^3)

```c++
#include <iostream>
using namespace std;

const int N = 30;
int n;
int f[N][N], w[N], root[N][N];

void dfs(int l, int r){
    if(l > r)   return;
    int t = root[l][r];
    printf("%d ", t);
    dfs(l, t - 1);
    dfs(t + 1, r);
}

int main(){
    cin >> n;
    for(int i = 1; i <= n; i++){
        cin >> w[i];
        f[i][i] = w[i];
        root[i][i] = i;
    }
    for(int len = 2; len <= n; len++)
        for(int l = 1; l + len - 1 <= n; l++){
            int r = l + len - 1;
            for(int k = l; k <= r; k++){
                int left = k == l ? 1 : f[l][k - 1];
                int right = k == r ? 1 : f[k + 1][r];
                int score = left * right + w[k];
                if(f[l][r] < score){
                    f[l][r] = score;
                    root[l][r] = k;
                }
            }
        }
    cout << f[1][n] << endl;
    dfs(1, n);
    puts("");
    return 0;
}
```



## 二维区间DP

### [棋盘分割](https://www.acwing.com/problem/content/323/)

![棋盘](区间DP1.jpg)

**注意**：棋盘的边框条数是9，棋盘的格子列数是8，而代码中用棋盘格子的**右下角坐标表示格子**。因此我们枚举切割线的时候，是从左上角开始枚举的，而右下角不该枚举。切完之后分成两部分，其中一个的坐标等于切割线，而另一部分坐标是切割线+1（因为用右下角表示的格子）

```c++
#include <iostream>
#include <cstring>
#include <cmath>
using namespace std;

const int N = 15, M = 9;
int n, m = 8;
int s[9][9];
double f[M][M][M][M][N];
double X;
double inf = 1e9;

double get(int x1, int y1, int x2, int y2){
    double sum = s[x2][y2] - s[x1 - 1][y2] - s[x2][y1 - 1] + s[x1 - 1][y1 - 1] - X;
    return sum * sum / n;
}

double dfs(int x1, int y1, int x2, int y2, int k){
    double &v = f[x1][y1][x2][y2][k];
    if(v >= 0)  return v; // 有值表示已经搜索过
    if(k == 1)  return v = get(x1, y1, x2, y2);
    
    v = inf;
    // 横切
    for(int i = x1; i < x2; i++){ // 枚举的是分界线。用右下角表示一个格子
        v = min(v, dfs(x1, y1, i, y2, k - 1) + get(i + 1, y1, x2, y2));
        v = min(v, dfs(i + 1, y1, x2, y2, k - 1) + get(x1, y1, i, y2));
    }
    // 纵切
    for(int j = y1; j < y2; j++){
        v = min(v, dfs(x1, y1, x2, j, k - 1) + get(x1, j + 1, x2, y2));
        v = min(v, dfs(x1, j + 1, x2, y2, k - 1) + get(x1, y1, x2, j));
    }
    return v;
}

int main(){
    cin >> n;
    for(int i = 1; i <= m; i++)
        for(int j = 1; j <= m; j++){
            cin >> s[i][j];
            s[i][j] += s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1];
        }
    
    memset(f, -1, sizeof f); // 初始化double数组为NaN
    X = (double)s[m][m] / n;
    printf("%.3lf\n", sqrt(dfs(1, 1, m, m, n)));
    return 0;
}
```



## 补充

**博弈论+简单的区间DP**:

1. [石子游戏 VII](#石子游戏 VII)
2. [预测赢家](#预测赢家)

### [石子游戏 VII](https://leetcode-cn.com/problems/stone-game-vii/)

dp[l, r]：移除区间[l, r]的石子后的相对差值

```java
class Solution {
    public int stoneGameVII(int[] stones) {
        int n = stones.length;
        int[][] dp = new int[n][n];
        int[] sum = new int[n + 1];
        for(int i = 1; i <= n; i++)
            sum[i] = sum[i - 1] + stones[i - 1];
        for(int len = 2; len <= n; len++)
            for(int l = 0; l + len - 1 < n; l++){
                int r = l + len - 1;
                dp[l][r] = Math.max(sum[r + 1] - sum[l + 1] - dp[l + 1][r], sum[r] - sum[l] - dp[l][r - 1]);
            }
        return dp[0][n - 1];
    }
    
    public int stoneGameVII(int[] stones) {
        int n = stones.length;
        int[] f = new int[n];
        int[] sum = new int[n + 1];
        for(int i = 1; i <= n; i++)
            sum[i] = sum[i - 1] + stones[i - 1];
        for(int i = n - 1; i >= 0; i--)
            for(int j = i + 1; j < n; j++)
                f[j] = Math.max(sum[j + 1] - sum[i + 1] - f[j], sum[j] - sum[i] - f[j - 1]);
        return f[n - 1];
    }
}
```

### [预测赢家](https://leetcode-cn.com/problems/predict-the-winner/)

类似于上一题

```java
class Solution {
    public boolean PredictTheWinner(int[] nums) { // 无空间优化，类似于区间DP
        int n = nums.length;
        int[][] f = new int[n][n];
        for(int i = 0; i < n; i++)  f[i][i] = nums[i];
        for(int len = 2; len <= n; len++)
            for(int l = 0; l + len - 1 < n; l++){
                int r = l + len - 1;
                f[l][r] = Math.max(nums[l] - f[l + 1][r], nums[r] - f[l][r - 1]);
            }
        return f[0][n - 1] >= 0;
    }
    
    // 根据状态转移表，将l替换为i，r替换为j，可以让i从大到小遍历，j从小到大遍历
    public boolean PredictTheWinner(int[] nums) {
        // 博弈问题有先手和后手之分，因此可以将两者整合在一起，表示先手对后手的相对
        int n = nums.length;
        int[] dp = new int[n];
        for(int i = 0; i < n; i++)  dp[i] = nums[i];
        for(int i = n - 1; i >= 0; i--){
            for(int j = i + 1; j < n; j++){ // len长度至少为1
                dp[j] = Math.max(nums[i] - dp[j], nums[j] - dp[j - 1]);
            }
        }
        return dp[n - 1] >= 0;
    }
}
```



### [戳气球](https://leetcode-cn.com/problems/burst-balloons/)

思路：

状态表示：f[L, R]

- 集合：所有戳破区间(L, R)的气球得到分数的方案
- 属性：max

状态计算：

最后一个戳的气球是第k个，则有f[L, R] = f[L, k] + f[k, R] + scores[k] × scores[L] × scores[R]

```java
class Solution {
    public int maxCoins(int[] nums) {
        int n = nums.length;
        int[] scores = new int[n + 2];
        scores[0] = scores[n + 1] = 1;
        for(int i = 1; i <= n; i++)
            scores[i] = nums[i - 1];
        int[][] dp = new int[n + 2][n + 2];
        for(int len = 3; len <= n + 2; len++)
            for(int l = 0; l + len - 1 <= n + 1; l++){
                int r = l + len - 1;
                for(int k = l + 1; k < r; k++)
                    dp[l][r] = Math.max(dp[l][r], dp[l][k] + dp[k][r] + scores[k] * scores[l] * scores[r]);
            }
        return dp[0][n + 1];
    }
}
```

