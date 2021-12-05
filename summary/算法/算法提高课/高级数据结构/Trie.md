# Trie

## [前缀统计](https://www.acwing.com/problem/content/144/)

```c++
#include<iostream>
using namespace std;

// Trie里的长度为M，需要预估可能用到多少个节点
const int N = 1e6 + 5, M = 5e5 + 5;

int n, m;
// son表示Trie，cnt表示在Trie中以某个点结尾的单词个数
int son[M][26], cnt[N], idx;
char str[N];

void insert(){
    int p = 0; // 0表示根节点
    for(int i = 0; str[i]; i++){
        int &s = son[p][str[i] - 'a'];
        if(!s)  s = ++idx;
        p = s;
    }
    cnt[p]++;
}

int query(){
    int p = 0, res = 0;
    for(int i = 0; str[i]; i++){
        int &s = son[p][str[i] - 'a'];
        if(!s)  break;
        res += cnt[s];
        p = s;
    }
    return res;
}

int main(){
    scanf("%d%d", &n, &m);
    while(n--){
        scanf("%s", str);
        insert();
    }
    
    while(m--){
        scanf("%s", str);
        printf("%d\n", query());
    }
    return 0;
}
```



## [最大异或对](https://www.acwing.com/problem/content/145/)

```c++
#include<iostream>
using namespace std;

// M = 31 * 1e5
const int N = 1e5 + 5, M = 3e6;

int n;
int a[N];
int son[M][2], idx;

void insert(int x){
    int p = 0;
    for(int i = 30; ~i; i--){
        int &s = son[p][x >> i & 1];
        if(!s)  s = ++idx;
        p = s;
    }
}

int query(int x){
    int p = 0, res = 0;
    // ~为取反符号
    for(int i = 30; ~i; i--){
        int s = x >> i & 1;
        // !为逻辑非，因此0变成1，其他数都变成0
        // 在逻辑非这里，只有false和true的区别，而0是false，其他数都是true
        // 如果另一边存在，则走另一边
        if(son[p][!s]){
            p = son[p][!s];
            res |= 1 << i;
        }else   p = son[p][s];
    }
    return res;
}

int main(){
    scanf("%d", &n);
    for(int i = 0; i < n; i++){
        scanf("%d", &a[i]);
        insert(a[i]);
    }
    int res = 0;
    for(int i = 0; i < n; i++)
        res = max(res, query(a[i]));
    printf("%d\n", res);
    return 0;
}
```



## [牛异或](https://www.acwing.com/problem/content/1416/)

```c++
#include<iostream>
using namespace std;

const int N = 1e5 + 5, M = 21 * N;

int n;
int s[N];
int son[M][2], idx, id[M];

void insert(int x, int k){
    int p = 0;
    for(int i = 20; ~i; i--){
        int u = x >> i & 1;
        if(!son[p][u])  son[p][u] = ++idx;
        p = son[p][u];
    }
    id[p] = k;
}

int query(int x){
    int p = 0;
    for(int i = 20; ~i; i--){
        int u = x >> i & 1;
        if(son[p][!u])  p = son[p][!u];
        else    p = son[p][u];
    }
    return id[p];
}

int main(){
    scanf("%d", &n);
    for(int i = 1; i <= n; i++){
        scanf("%d", &s[i]);
        s[i] ^= s[i - 1];
    }
    
    int res = -1, start, ed;
    insert(s[0], 0);
    for(int i = 1; i <= n; i++){
        int k = query(s[i]);
        int x = s[i] ^ s[k];
        if(x > res) res = x, start = k + 1, ed = i;
        insert(s[i], i);
    }
    printf("%d %d %d\n", res, start, ed);
    return 0;
}
```



## [最长异或值路径](https://www.acwing.com/problem/content/146/)

```c++
#include<iostream>
#include<cstring>
using namespace std;

const int N = 1e5 + 5, M = 31 * N;

int n;
int h[N], e[N * 2], ne[N * 2], w[N * 2], idx;
int son[M][2], sidx;
int res;

void add(int a, int b, int c){
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}

void insert(int x){
    int p = 0;
    for(int i = 31; ~i; i--){
        int u = x >> i & 1;
        if(!son[p][u])  son[p][u] = ++sidx;
        p = son[p][u];
    }
}

int query(int x){
    int p = 0, res = 0;
    for(int i = 31; ~i; i--){
        int u = x >> i & 1;
        if(son[p][!u])  p = son[p][!u], res |= 1 << i;
        else    p = son[p][u];
    }
    return res;
}

void dfs1(int root, int fa, int d){
    for(int i = h[root]; ~i; i = ne[i]){
        int j = e[i];
        if(j == fa) continue;
        insert(d ^ w[i]);
        dfs1(j, root, d ^ w[i]);
    }
}

void dfs2(int root, int fa, int d){
    for(int i = h[root]; ~i; i = ne[i]){
        int j = e[i];
        if(j == fa) continue;
        res = max(res, query(d ^ w[i]));
        dfs2(j, root, d ^ w[i]);
    }
}

int main(){
    scanf("%d", &n);
    memset(h, -1, sizeof h);
    int root;
    for(int i = 1; i < n; i++){
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c), add(b, a, c);
        root = a;
    }
    insert(0);
    dfs1(root, -1, 0);
    dfs2(root, -1, 0);
    printf("%d\n", res);
    return 0;
}
// 改进思路：先一遍dfs得到从根节点到当前节点的距离，然后把所有的距离加入到Trie中，再查询
```



