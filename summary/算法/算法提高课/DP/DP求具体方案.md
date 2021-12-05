## LCS求具体方案

```c++
#include<iostream>
using namespace std;

const int N = 1010;

int n;
char s1[N], s2[N], path[N];
int f[N][N];

void dfs(int x, int y, int u){
    if(f[x][y] == 0){
        for(int i = u - 1; i >= 0; i--){
            printf("%c ", path[i]);
        }
        puts("");
        return;
    }
    if(s1[x] == s2[y]){
        path[u] = s1[x];
        dfs(x - 1, y - 1, u + 1);
    }else if(f[x - 1][y] > f[x][y - 1]){
        dfs(x - 1, y, u);
    }else if(f[x - 1][y] < f[x][y - 1]){
        dfs(x, y - 1, u);
    }else{
        dfs(x - 1, y, u);
        dfs(x, y - 1, u);
    }
}

int main(){
    scanf("%d%s%s", &n, s1 + 1, s2 + 1);
    for(int i = 1; i <= n; i++){
        for(int j = 1; j <= n; j++){
            if(s1[i] == s2[j])  f[i][j] = f[i - 1][j - 1] + 1;
            else    f[i][j] = max(f[i - 1][j], f[i][j - 1]);
        }
    }
    dfs(n, n, 0);
    return 0;
}
```

