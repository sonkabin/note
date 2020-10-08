# KMP算法

## 基本思想

当出现不匹配时，就能知道一部分文本的内容（因为匹配失败之前文本已经与模式匹配），可以利用这些信息，避免回退到所有这些已知的字符之前

## 内容

更新next数组

```c++
const int N = 10010;
int ne[N]; // ne[i]表示以i为终点的后缀数组与以第一个字符为起点的前缀数组相同时，最大的长度
char pat[N], s[N];
void init(){
    for(int i = 1, j = -1; i < n; i++){ // i从1开始是因为第一个字符退无可退
        while(j > -1 && pat[i] != pat[j + 1])	j = ne[j]; // i是与j+1的字符进行匹配的，尝试让j回退直到能匹配或者j退无可退
        if(pat[i] == pat[j + 1])	j++;
        ne[i] = j;
    }
}
int match(){
    for(int i = 0, j = -1; i < m; i++){
        while(j > -1 && s[i] != pat[j + 1])	j = ne[j];
        if(s[i] == pat[j + 1])	j++;
        if(i == n - 1){
            return i - n + 1;
        }
    }
}
```

### 通过有限状态自动机（DFA）确定模式的状态

该部分内容请将j理解为**状态**而不是下标

这个确定状态的过程其实也可以看作是一个动态规划的过程

1. 定义正在处理的字符为`c`，模式为`pat`，模式的第几个状态为`j`，有限状态自动机为`dfa[][]`，其中第一维表示第几个状态，而第二维表示该状态下的字符，数据内容表示下一个状态

2. 如果`c == pat[j]`，那么只要让模式的状态前进即可，即`dfa[j][c] = j + 1`

3. 如果`c != pat[j]`，则必须回到某个状态（或不动），即状态重启。再定义一个名字：影子状态X（即和当前状态有着相同前缀的状态）。因为X到j之间的AB可以被X之前的AB所匹配到，因此状态j准备重启时，可以根据X的状态来获得最近的重启图。举例来说，如果j位置遇到字符`A`，进行状态重启，即

   `dfa[j]['A'] = dfa[X]['A']`，显然会位于状态3；如果j位置遇到字符`D`，进行状态重启，即

   `dfa[j]['D'] = dfa[X]['D']`，在X状态给予一个`D`，X的状态会回到0（因为初始化时所有状态都会回到0，所以该状态会隐式转移，当然也可以显示指出）

![DFA](img/DFA.jpg)

4. 实现

   ```java
   public class KMP{
       private int[][] dfa;
       private String pat;
       private final int R = 256; // 为自动机准备ASCII码256个字符的位置
       public KMP(String pat){
           this.pat = pat;
           int M = pat.length();
           dfa = new int[M][R];
           // 设置dfa初始状态，只有该字符才允许进入下一个状态
           dfa[0][pat.charAt(0)] = 1;
           // 准备其他的状态转移，说到状态转移，这正是dp（初始状态，状态转移方程，最优解）
           int X = 0; // 设置初始影子状态为0
           for(int j = 1; j < M; j++){
               // 写法一
               for(int c = 0; c < R; c++){
                   if(c == pat.charAt(j))	dfa[j][c] = j + 1; // 匹配时只需要前进
                   else	dfa[j][c] = dfa[X][c]; // 无法匹配，由影子状态来决定下一个转移
               }
               // 更新影子状态。当前状态是X，遇到字符pat[j]，下一个状态跟从影子状态的转移
               X = dfa[X][pat.charAt(j)];
           }
       }
   }
   ```

   

### 搜索

搜索就是利用上述的DFA进行状态的转移

```java
public int search(String text){
    int M = pat.length();
    int j = 0; // 从状态0出发
    for(int i = 0; i < text.length(); i++){
        // 计算下一个状态
        j = dfa[j][text.charAt(i)];
        if(j == M)	return i - M + 1; // 匹配成功
    }
    return -1;
}
```

### 结合两部分

```java
public class KMP{
    private int[][] dfa;
    private String pat;
    private final int R = 256;
    public KMP(String pat){
        this.pat = pat;
        int M = pat.length();
        dfa = new int[M][R];
        dfa[0][pat.charAt(0)] = 1;
        int X = 0;
        for(int j = 1; j < M; j++){
            // 写法二
            for(int c = 0; c < R; c++){
                dfa[j][c] = dfa[X][c];
            }
            dfa[j][pat.charAt(j)] = j + 1;
            X = dfa[X][pat.charAt(j)];
        }
    }
    public int search(String text){
        int M = pat.length();
        int j = 0;
        for(int i = 0; i < text.length(); i++){
            j = dfa[j][text.charAt(i)];
            if(j == M)	return i - M + 1;
        }
        return -1;
    }
}
```



## 算法复杂度

时间复杂度平均O(1.1N)，最坏情况O(2N)，空间复杂度O(MR)；而暴力算法时间复杂度平均O(1.1N)，最坏O(MN)

## 参考资料

1. 算法第四版
2. [动态规划之KMP字符匹配算法](https://labuladong.gitbook.io/algo/dong-tai-gui-hua-xi-lie/dong-tai-gui-hua-zhi-kmp-zi-fu-pi-pei-suan-fa#yi-kmp-suan-fa-gai-shu)



