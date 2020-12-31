# BFS

## 最短路模型

网格图上（或者数轴上等等），寻找到某一个点的最小距离



## 多源BFS

一个网格图上，到多个特定点的距离取最小值。思路类似于多源最短路（建立虚拟节点，连接到多个源点，距离为0），网格图上不需要建立虚拟节点，先将多个源点加入到队列中，然后执行BFS即可



## 最小步数模型

和最短路模型不同的是，它不是在内部移动，而是有一堆操作，通过这些操作达到目标态最少需要多少次操作



## 双向BFS

只能用于解决最小步数模型的问题，将解空间大量减少

题目：

1. [单词接龙](https://leetcode-cn.com/problems/word-ladder/)

2. [字串变换](https://www.acwing.com/problem/content/192/)



## 双端队列BFS

如果图的边权只有1和0，可以使用该算法。时间复杂度线性，比最短路算法快

题目：

1. [电路维修](https://www.acwing.com/problem/content/description/177/)

   ```c++
   #include <iostream>
   #include <cstring>
   #include <deque>
   using namespace std;
   typedef pair<int, int> PII;
   
   const int N = 510;
   int n, m;
   char g[N][N];
   int dist[N][N];
   bool visited[N][N];
   
   int bfs(){
       memset(dist, 0x3f, sizeof dist);
       memset(visited, 0, sizeof visited);
       dist[0][0] = 0;
       deque<PII> q;
       q.push_back({0, 0});
       
       int dx[] = {-1, -1, 1, 1}, dy[] = {-1, 1, 1, -1}; // 表示格点上的坐标
       /*
       表示元件在输入数组中的位置(第一个元件在(0,0), 第二个元件在(0, 1))相对于当前格点位置的关系，顺时针方向
       以格点坐标(1,2)为例，四个方向的格点位置为(0, 1), (0, 3), (2, 3), (2, 1)
       而这4个格点构成放4个元件的区域,对应位置为(0, 1), (0, 2), (1, 2), (1, 1)
       */
       int ix[] = {-1, -1, 0, 0}, iy[] = {-1, 0, 0, -1};
       char sc[] = "\\/\\/"; // 对应的无需旋转情况下，元件的方向
       
       while(!q.empty()){
           PII t = q.front();
           q.pop_front();
           
           int x = t.first, y = t.second;
           if(x == n && y == m)    return dist[x][y]; // 出队时距离一定是最小的
           if(visited[x][y])  continue;
           visited[x][y] = true;
           for(int i = 0; i < 4; i++){
               int cx = x + dx[i], cy = y + dy[i];
               if(cx < 0 || cx > n || cy < 0 || cy > m)    continue;
               int gx = x + ix[i], gy = y + iy[i];
               int w = g[gx][gy] != sc[i];
               int d = dist[x][y] + w; // 可用于更新相邻格点的距离
               if(d <= dist[cx][cy]){ // cx、cy的dist可能被队列中后出队的更新。比如有t1和t2,t1比t2先出队，从原点到t1的距离与到t2的距离相等，t1到终点需要一步转换操作，而t2到终点不需要转换操作，此时最短距离等于t2的距离
                   dist[cx][cy] = d;
                   if(w)   q.push_back({cx, cy});
                   else    q.push_front({cx, cy});
               }
           }
       }
       return -1;
   }
   
   int main(){
       int T;
       cin >> T;
       while(T--){
           cin >> n >> m;
           for(int i = 0; i < n; i++)
               scanf("%s", g[i]);
           // 从(0,0)开始走，只可能走到 横坐标+纵坐标 之和为偶数的格点上
           if((n + m) & 1) puts("NO SOLUTION");
           else{
               printf("%d\n", bfs());
           }
       }
       return 0;
   }
   ```

   

2. [P1346 电车](https://www.luogu.com.cn/problem/P1346)

   ```c++
   #include <iostream>
   #include <cstring>
   #include <deque>
   using namespace std;
   
   const int N = 110, M = N * N;
   int n, a, b;
   int h[M], e[M], ne[M], w[M], idx;
   int dist[N];
   bool visited[N];
   
   void add(int a, int b, int c){
       e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
   }
   
   int bfs(){
       memset(dist, 0x3f, sizeof dist);
       dist[a] = 0;
       deque<int> q;
       q.push_back(a);
       while(!q.empty()){
           int t = q.front();
           q.pop_front();
           if(t == b)  return dist[b];
           if(visited[t])  continue;
           visited[t] = true;
   
           for(int i = h[t]; ~i; i = ne[i]){
               int j = e[i], v = w[i];
               int d = dist[t] + v;
               if(d <= dist[j]){
                   dist[j] = d;
                   if(v)   q.push_back(j);
                   else    q.push_front(j);
               }
           }
       }
       return -1;
   }
   
   int main(){
       cin >> n >> a >> b;
       memset(h, -1, sizeof h);
       for(int i = 1; i <= n; i++){
           int k;
           cin >> k;
           for(int j = 0; j < k; j++){
               int t;
               cin >> t;
               if(j == 0)  add(i, t, 0);
               else    add(i, t, 1);
           }
       }
       printf("%d\n", bfs());
       return 0;
   }
   ```

   

## A*

$f(n) = g(n) + h(n)$，f(n)表示从初始状态经由中间状态n达到目标状态的代价估计，g(n)表示从初始状态到中间状态n的实际代价，h(n)表示从中间状态n到目标状态的代价估计。

[路径规划之 A* 算法](https://zhuanlan.zhihu.com/p/54510444)

```
* 初始化open_set和close_set；
* 将起点加入open_set中，并设置优先级为0（优先级最高）；
* 如果open_set不为空，则从open_set中选取优先级最高(f(n)最小)的节点n：
    * 如果节点n为终点，则：
        * 从终点开始逐步追踪parent节点，一直达到起点；
        * 返回找到的结果路径，算法结束；
    * 如果节点n不是终点，则：
        * 将节点n从open_set中删除，并加入close_set中；
        * 遍历节点n所有的邻近节点：
            * 如果邻近节点m在close_set中，则：
                * 否则，跳过，选取下一个邻近节点
            * 如果邻近节点m也不在open_set中，则：
                * 设置节点m的parent为节点n
                * 计算节点m的优先级
                * 将节点m加入open_set中
```

应用的环境：

1. 有解（无解时，仍然会把所有空间搜索，会比一般的bfs慢，因为优先队列的操作是logn的）
2. 边权非负，如果是负数，那么终点的估值有可能是负无穷，终点可能会直接出堆

性质：

1. h(n)小于等于真实距离，则一定能找到最短路径（任意节点第一次出堆一定是最短的），但h(n)越小，找的点就越多，极端情况h(n)=0，算法退化成Dijkstra算法
2. h(n)大于真实值，则不保证找到最短路径，但速度快

题目：

1. [八数码](https://www.acwing.com/problem/content/181/)

   ```c++
   #include <iostream>
   #include <queue>
   #include <vector>
   #include <algorithm>
   #include <unordered_map>
   using namespace std;
   typedef pair<int, string> PIS;
   
   int h(string state){ // 估价函数：当前位置到目标位置的曼哈顿距离
       int res = 0;
       for(int i = 0; i < 9; i++){
           if(state[i] != 'x'){
               int t = state[i] - '1'; // 1位于0索引处,2位于1索引处
               res += abs(t / 3 - i / 3) + abs(t % 3 - i % 3);
           }
       }
       return res;
   }
   
   string astar(string start){
       unordered_map<string, int> dist;
       unordered_map<string, pair<char, string>> prev;
       priority_queue<PIS, vector<PIS>, greater<PIS>> heap;
   
       string ed = "12345678x";
       int dx[] = {-1, 1, 0, 0}, dy[] = {0, 0, -1, 1};
       char op[] = "udlr";
       dist[start] = 0;
       heap.push({h(start), start});
   
       while(!heap.empty()){
           PIS t = heap.top();
           heap.pop();
           string state = t.second;
           if(state == ed) break;
           int x, y;
           for(int i = 0; i < 9; i++)
               if(state[i] == 'x'){
                   x = i / 3;
                   y = i % 3;
                   break;
               }
           string source = state;
           for(int i = 0; i < 4; i++){
               int cx = x + dx[i], cy = y + dy[i];
               if(cx < 0 || cx > 2 || cy < 0 || cy > 2)    continue;
               state = source;
               swap(state[cx * 3 + cy], state[x * 3 + y]);
               if(dist[state] == 0 || dist[state] > dist[source] + 1){
                   dist[state] = dist[source] + 1;
                   prev[state] = {op[i], source};
                   heap.push({dist[state] + h(state), state});
               }
           }
       }
   
       string res;
       while(ed != start){
           res += prev[ed].first;
           ed = prev[ed].second;
       }
       reverse(res.begin(), res.end());
       return res;
   }
   
   int main(){
       string start, seq;
       char c;
       for(int i = 0; i < 9; i++){
           cin >> c;
           start += c;
           if(c != 'x')    seq += c;
       }
       int cnt = 0;
       for(int i = 0; i < 9; i++)
           for(int j = i; j < 9; j++)
               if(seq[i] < seq[j]) cnt++;
       if(cnt & 1)   cout << "unsolvable" << endl;
       else cout << astar(start) << endl;
       return 0;
   }
   ```

   



