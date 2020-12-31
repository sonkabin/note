# 数位DP

1. 区间[L, R]转化为求区间[1, R] - [1, L - 1]
2. 用树来思考
3. 难点在于预处理（如果用循环写的话，哭泣，用记忆化搜索是真滴简单（对我来说两个都难））

### [计数问题](https://www.acwing.com/problem/content/description/340/)

$$
无论现在的状态是 1\_ \ \_ 还是 12345\_ \ \_（\_表示还没填到的数字），只要前面只有一个1，填完后面两位数所产生的方案数就是120 \\
因此令n的状态state为n所包含的1的个数，dp_{i,state}表示填到第i位，前面填了state个1时的答案
$$



```c++
#include <iostream>
#include <cstring>
using namespace std;

const int N = 10;
int f[N][2000]; // f[dep][sum]: 从最高位到dep位，1的个数为sum的总共的数的个数
int l, r;
int a[10], b[10];
int len, c[N];

int dfs(int dep, int sum, int target, int lead, int limit){
    if(dep == 0)    return sum;
    if(f[dep][sum] != -1 && lead == 0 && limit == 0)  return f[dep][sum];
    
    int ed = limit == 1 ? c[dep] : 9;
    int res = 0;
    for(int i = 0; i <= ed; i++){
        if(i == 0 && lead == 1) res += dfs(dep - 1, sum, target, 1, i == ed && limit);
        else{
            res += dfs(dep - 1, sum + (i == target), target, 0, i == ed && limit);
        }
    }
    if(lead == 0 && limit == 0) f[dep][sum] = res;
    return res;
}

void dp(int n, int aux[]){
    if(!n)  return;
    len = 0;
    while(n)    c[++len] = n % 10, n /= 10;
    for(int i = 0; i < 10; i++){
        memset(f, -1, sizeof f);
        aux[i] = dfs(len, 0, i, 1, 1);
    }
}

int main(){
    while(cin >> l >> r, l || r){
        if(l > r)   swap(l, r);
        dp(r, a);
        dp(l - 1, b);
        for(int i = 0; i < 10; i++)
            cout << a[i] - b[i] << " ";
        cout << endl;
    }
    return 0;
}
```

[参考1](https://leopoldacc.github.io/Blogs/2020/12/15/%E6%95%B0%E4%BD%8Ddp/)

[参考2](https://leetcode-cn.com/problems/number-of-digit-one/solution/shu-wei-dp-by-weierstras/)

### [度的数量](https://www.acwing.com/problem/content/1083/)

思路：K个互不相同的B的整数次幂，等价于求数字num在B进制下是否能**由K个1且其余位都为0组成**

```c++
#include <iostream>
#include <vector>
using namespace std;

const int N = 33; // B的范围是2-10，因此为2时最多有32位
int l, r, k, B;
int C[N][N];

void init(){
    for(int i = 0; i < N; i++)
        for(int j = 0; j <= i; j++)
            if(!j)  C[i][j] = 1;
            else    C[i][j] = C[i - 1][j - 1] + C[i - 1][j];
}

int dp(int n){
    if(!n)  return 0; // 0不能由任何的B进制数表示出来，因此为0 
    vector<int> nums;
    while(n)    nums.push_back(n % B), n /= B; // 以B进制形式分解n
    
    int res = 0;
    int last = 0;
    for(int i = nums.size() - 1; i >= 0; i--){
        int x = nums[i];
        if(x){
            res += C[i][k - last]; // 第i位取0的情况
            // 下面是第i位取1的情况
            if(x > 1){
                if(k - last - 1 >= 0) res += C[i][k - last - 1]; // 直接枚举组合数即可
                break;
            }else{
                last++; // 不可以直接枚举组合数，因为枚举出的数可能大于上界
                if(last > k)    break;
            }
        }
        if(!i && last == k) res++;
    }
    return res;
}

int main(){
    cin >> l >> r >> k >> B;
    init();
    cout << dp(r) - dp(l - 1) << endl;
    return 0;
}

// 记忆化搜索：这题没有前导0限制，因为000111010也可能是合法方案（表示的是取法）
#include <iostream>
#include <cstring>
using namespace std;

const int N = 33;
int l, r, k, B;
int len, a[N], f[N][22];

int dfs(int dep, int last, int limit){
    if(last < 0 || dep < last)    return 0;
    if(dep == 0)    return last == 0 ? 1 : 0;
    if(f[dep][last] != -1 && limit == 0)   return f[dep][last];
    
    int ed = limit == 1 ? a[dep] : 9;
    int res = 0;
    for(int i = 0; i <= ed; i++){
        if(i > 1)   break;
        res += dfs(dep - 1, last - i, i == ed && limit);
    }
    if(limit == 0) f[dep][last] = res;
    return res;
}

int dp(int n){
    if(!n)  return 0;
    len = 0;
    while(n)    a[++len] = n % B, n /= B; // 以B进制形式分解n
    return dfs(len, k, 1);
}

int main(){
    cin >> l >> r >> k >> B;
    memset(f, -1, sizeof f);
    cout << dp(r) - dp(l - 1) << endl;
    return 0;
}
```

[参考](https://www.acwing.com/solution/content/3112/)



### [Windy数](https://www.acwing.com/problem/content/1085/)

```c++
#include <iostream>
#include <vector>
using namespace std;

const int N = 11;
int f[N][N]; // f[i][j]:有i位数，最高位为j的windy数的count
int l, r;

void init(){
    for(int i = 0; i < 10; i++) f[1][i] = 1;
    for(int i = 2; i < N; i++)
        for(int j = 0; j < 10; j++)
            for(int k = 0; k < 10; k++)
                if(abs(j - k) >= 2)
                    f[i][j] += f[i - 1][k];
}

int dp(int n){
    if(!n)  return 0; // 根据windy数的定义，0不是windy数
    vector<int> nums;
    while(n)    nums.push_back(n % 10), n /= 10;
    
    int res = 0;
    int last = -2;
    for(int i = nums.size() - 1; i >= 0; i--){
        int x = nums[i];
        for(int j = (i == nums.size() - 1); j < x; j++)
            if(abs(last - j) >= 2)
                res += f[i + 1][j];
        if(abs(last - x) < 2)   break;
        last = x;
        if(!i)  res++;
    }
    
    // 位数小于n位数
    for(int i = 1; i < nums.size(); i++)
        for(int j = 1; j < 10; j++)
            res += f[i][j];
    return res;
}

int main(){
    cin >> l >> r;
    init();
    cout << dp(r) - dp(l - 1) << endl;
    return 0;
}

// 记忆化搜索
#include <iostream>
#include <cstring>
using namespace std;

const int N = 11;
int f[N][N], a[N];
int l, r, len;

int dfs(int dep, int last, int lead, int limit){
    if(dep == 0)    return 1;
    if(f[dep][last] != -1 && lead == 0 && limit == 0)   return f[dep][last];
    
    int ed = limit == 1 ? a[dep] : 9;
    int res = 0;
    for(int i = 0; i <= ed; i++){
        if(abs(i - last) < 2)   continue;
        if(i == 0 && lead == 1) res += dfs(dep - 1, -2, 1, limit && (i == ed));
        else    res += dfs(dep - 1, i, 0, limit && (i == ed));
    }
    if(lead == 0 && limit == 0) f[dep][last] = res;
    return res;
}

int dp(int n){
    if(!n)  return 1;
    len = 0;
    while(n)    a[++len] = n % 10, n /= 10;
    return dfs(len, -2, 1, 1);
}

int main(){
    cin >> l >> r;
    memset(f, -1, sizeof f);
    cout << dp(r) - dp(l - 1) << endl;
    return 0;
}
```



### [数字游戏](https://www.acwing.com/problem/content/1084/)

```c++
#include <iostream>
#include <vector>
using namespace std;

const int N = 11;
int l, r;
int f[N][10]; // f[i][j]:有i位，最高位为j的不降数的count

void init(){
    for(int i = 0; i < 10; i++) f[1][i] = 1;
    for(int i = 2; i < N; i++)
        for(int j = 0; j < 10; j++)
            for(int k = j; k < 10; k++)
                f[i][j] += f[i - 1][k];
}

int dp(int n){
    if(!n)  return 1; // 对于数字0来说，它满足非下降关系，因此返回1
    vector<int> nums;
    while(n)    nums.push_back(n % 10), n /= 10;
    
    int res = 0;
    int last = 0;
    for(int i = nums.size() - 1; i >= 0; i--){
        int x = nums[i];
        for(int j = last; j < x; j++)
            res += f[i + 1][j];
        if(x < last)    break;
        if(!i) res++;
        last = x;
    }
    return res;
}

int main(){
    init();
    while(cin >> l >> r){
        cout << dp(r) - dp(l - 1) << endl;
    }
    return 0;
}

// 记忆化搜索
#include <iostream>
#include <cstring>
using namespace std;

const int N = 11;
int l, r, len;
int f[N][10], a[N];

int dfs(int dep, int last, int limit){
    if(dep == 0)    return 1;
    if(f[dep][last] != -1 && limit == 0)    return f[dep][last];
    
    int ed = limit == 1 ? a[dep] : 9;
    int res = 0;
    for(int i = last; i <= ed; i++)
        res += dfs(dep - 1, i, limit && (i == ed));
    
    if(limit == 0)  f[dep][last] = res;
    return res;
}

int dp(int n){
    len = 0;
    while(n)    a[++len] = n % 10, n /= 10;
    return dfs(len, 0, 1);
}

int main(){
    memset(f, -1, sizeof f);
    while(cin >> l >> r){
        cout << dp(r) - dp(l - 1) << endl;
    }
    return 0;
}
```



### [数字游戏 II](https://www.acwing.com/problem/content/description/1086/)

```c++
#include <iostream>
#include <cstring>
#include <vector>
using namespace std;

const int N = 110;
int f[11][10][N]; // f[i][j][k]:共i位，最高位是j，模的结果是k
int p, l, r;

int mod(int n){
    return (n % p + p) % p;
}

void init(){
    memset(f, 0, sizeof f);
    for(int i = 0; i < 10; i++) f[1][i][i % p]++;
    for(int i = 2; i < 11; i++)
        for(int j = 0; j < 10; j++)
            for(int k = 0; k < p; k++)
                for(int t = 0; t < 10; t++)
                    f[i][j][k] += f[i - 1][t][mod(k - j)]; // (j+X)%p = k，则有j%p + X%p = k，即X%p = k-j%p = (k - j)%p
}

int dp(int n){
    if(!n)  return 1; // 0 mod p == 0，因此返回1
    vector<int> nums;
    while(n)    nums.push_back(n % 10), n /= 10;
    
    int res = 0;
    int last = 0;
    for(int i = nums.size() - 1; i >= 0; i--){
        int x = nums[i];
        for(int j = 0; j < x; j++)
            res += f[i + 1][j][mod(-last)]; // (last+X)%p = 0，即X%p = -last%p
        last = (last + x) % p;
        if(!last)  res++;
    }
    return res;
}

int main(){
    while(cin >> l >> r >> p){
        init();
        cout << dp(r) - dp(l - 1) << endl;
    }
    return 0;
}

// 记忆化搜索
#include <iostream>
#include <cstring>
using namespace std;

const int N = 110;
int f[11][N], a[11];
int p, l, r, len;

int dfs(int dep, int last, int limit){
    if(dep == 0)    return last == 0 ? 1 : 0;
    if(f[dep][last] != -1 && limit == 0)    return f[dep][last];
    
    int ed = limit == 1 ? a[dep] : 9;
    int res = 0;
    for(int i = 0; i <= ed; i++){
        res += dfs(dep - 1, (last + i) % p, limit && (i == ed));
    }
    if(limit == 0)  f[dep][last] = res;
    return res;
}

int dp(int n){
    len = 0;
    while(n)    a[++len] = n % 10, n /= 10;
    return dfs(len, 0, 1);
}

int main(){
    while(cin >> l >> r >> p){
        memset(f, -1, sizeof f);
        cout << dp(r) - dp(l - 1) << endl;
    }
    return 0;
}
```



### [不要62](https://www.acwing.com/problem/content/1087/)

```c++
#include <iostream>
#include <vector>
using namespace std;

const int N = 11;
int f[N][10];
int l, r;

void init(){
    for(int i = 0; i < 10; i++)
        if(i != 4)
            f[1][i] = 1;
    for(int i = 2; i < N; i++)
        for(int j = 0; j < 10; j++)
            if(j != 4)
                for(int k = 0; k < 10; k++){
                    if(k == 4 || (j == 6 && k == 2))    continue;
                    f[i][j] += f[i - 1][k];
                }
}

int dp(int n){
    if(!n)  return 1; // 0是满足条件的，因此返回1
    vector<int> nums;
    while(n)    nums.push_back(n % 10), n /= 10;
    
    int res = 0;
    int last = 0;
    for(int i = nums.size() - 1; i >= 0; i--){
        int x = nums[i];
        for(int j = 0; j < x; j++){
            if(j == 4 || (last == 6 && j == 2)) continue;
            res += f[i + 1][j];
        }
        if(x == 4 || (last == 6 && x == 2))   break;
        if(!i)   res++;
        last = x;
    }
    return res;
}

int main(){
    init();
    while(cin >> l >> r, l && r){
        cout << dp(r) - dp(l - 1) << endl;
    }
    return 0;
}

// 记忆化搜索
#include <iostream>
#include <cstring>
using namespace std;

const int N = 11;
int f[N][10], a[N];
int l, r, len = 0;

// dep：深度，last：上一个状态，limit：最高位限制
int dfs(int dep, int last, int limit){
    if(dep == 0)    return 1;
    if(f[dep][last] != -1 && limit == 0) return f[dep][last];
    int ed = limit == 1 ? a[dep] : 9;
    int res = 0;
    for(int i = 0; i <= ed; i++){
        if(i == 4 || (last == 6 && i == 2)) continue;
        res += dfs(dep - 1, i, limit && (i == ed));
    }
    if(limit == 0)  f[dep][last] = res;
    return res;
}

int dp(int n){
    len = 0;
    while(n)    a[++len] = n % 10, n /= 10;
    return dfs(len, 0, 1);
}

int main(){
    memset(f, -1, sizeof f);
    while(cin >> l >> r, l && r){
        cout << dp(r) - dp(l - 1) << endl;
    }
    return 0;
}
```



## 参考

### 记忆化搜索

[dfs版的数位dp](https://www.luogu.com.cn/blog/virus2017/shuweidp)

几个状态：

#### 需要记录的信息

1. dep：记录搜索的深度
2. last：记录上一个状态

#### 前导0标记lead

如果是有关数字结构上的性质，0会影响数字结构，必须判断前导0；如果研究数字的组成，则无影响([数字游戏](#数字游戏)、[数字游戏 II](#数字游戏 II)、[不要62](#不要62))

1. 如果lead=1并且当前位也是0，则dep-1继续搜
2. 如果lead=1但当前位不是0，则当前位作为最高位，dep-1继续搜

#### 最高位标记limit

比如在搜素[0, 555]时，最高位的不同会影响下一位

1. 最高位是1-4时，下一位取值范围为0-9
2. 最高位是5时，下一位取值范围为0-5

为了区分上述情况，引入limit标记

1. 如果limit=1并且当前位取到了能取到的最高位，则下一位limt=1
2. 如果limit=1并且当前位没有渠道最高位，则下一位limit=0
3. 如果limit=0，下一位limit=0

#### 备忘录

备忘录数组的**下标**表示的是**一种状态**，只要**当前的状态和之前搜过的某个状态完全一样**，我们就可以直接返回原来已经记录下来的备忘录值。

举个例子

假如我们找 \[0,123456\]中符合某些条件的数

1. 当我们搜到 `1000??`时，dfs从下返上来的数值就是**当前位是第5位，前一位是0时的方案种数**，搜完这位会向上反，这是我们可以记录一下：当前位第5位，前一位是0时，有这么多种方案种数

1. 当我们继续搜到`1010??`时，我们发现当前状态又是搜到了第5位，并且上一位也是0，这与我们之前记录的情况相同，这样我们就可以不继续向下搜，直接把上次的值返回就行了。

注意，我们返回的值必须和当前**处于完全一样的状态**，这就是为什么备忘录数组下标要记录 dep，last等参数。

如果我们搜到了`1234??`，我们能不能直接返回之前记录的：当前第5位，前一位是4时的值？**答案是否定的**。

结论是：

1. 当limit=1时，不能记录和取用备忘录
2. 当lead=1时，不能记录和取用备忘录（前导0影响数字结构的情况）