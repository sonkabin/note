# Leetcode的总结

## String

1. 要遍历String时，将String转为char[]，`String.toCharArray()`，能加快速度，但消耗内存。空间换时间

## Integer.MIN_VALUE

```java
System.out.print(Integer.MIN_VALUE%10); // 显然为-8
/*注意这行输出*/
System.out.print(-Integer.MIN_VALUE%10); // 结果居然是-8，因为添加负号后溢出了，又变成了它本身
int a = Integer.MIN_VALUE%10;
int b = -a;
System.out.print(a + " " + b); // 意料之中的-8和8 
```

## 逆向思维

数组可以顺序遍历，也可以逆序遍历。有时候逆序遍历会带来好的结果。

## StringBuffer.append(str, offset, len)

并不是开始和结尾的索引

## 关于位运算

`>>`相当于`/2`，但是运算速度更快

`<<`相当于`*2`，但是运算速度更快

`>>>` 与 `>>` 的区别在于：对于正数作用相同，对于负数，`>>>`不保留符号，例`-1>>>1 ==> Integer.MAX_VALUE, 而-1>>1 ==> -1`

## 二分查找

```java
// 需要理解二分查找没有找到时返回的索引是表示小于该值的还是大于该值的？
int binarySearch(int[] arr, int target){
    int left = 0, right = arr.length-1;
    while(left <= right){
        int mid = (left+right)/2;
        if(arr[mid] == target)	return mid;
        else if(arr[mid] < target)	left = mid+1;
        else	right = mid-1;
    }
    // 若没有找到，left>right, 则返回right表示的是恰好小于target的值的索引，返回left表示的是恰好大于target的值的索引
    return left(or right); 
}
```

## 关于题目

不要往复杂的解法想（既浪费时间，又浪费注意力，还消耗心），否则折了夫人又赔兵

## StringBuilder

当长度固定时，用char[]比StringBuilder要好

## 链表

逆序打印链表：递归

从末尾开始数删除第n节点：快慢节点 or 递归

## <<与<<=

移位操作的优先级小于+-，<<=1相当于*=2

## 整体看问题与分解看问题，是两种方法

## 树的问题

BFS与DFS，前序、中序、后序、层次

## 先思考，大致步骤想完后再写

## 回溯与DFS的异同

两者都要设置状态。回溯其实是特殊的DFS，不过回溯在遍历某个位置时，遍历完之后要恢复状态

## 为什么List不能存基本类型而能存基本类型的数组？为什么Arrays.sort给多维基本类型数组和泛型使用同一个方法，而对于一维数组需要每种基本类型都来一个方法？

1. 因为数组是引用类型，不是基本类型，只要是引用类型，都可放入List，而基本类型不能直接放入List，需要包装类型才能放入List

2. 因为对一维数组排序，处理的是每个基本类型的元素，所以每种基本类型都需要一个方法；而对于多维数组来说，处理时至少为一维数组，即引用类型，所以可以和泛型共用一个方法。补充，包装类型调用的是泛型方法，因为包装类型是引用类型

3. 所以可以看出以下代码的不同

   ```java
   // x为List<String>
   String[] y = x.toArray(new String[0]);
   // x为List<int[]>
   int[][] y = x.toArray(new int[0][]);
   // x为List<int>，不可能情况
   // 所以就没有int[] y = x.toArray(int[0]); 因为List中不能存放基本类型
   // x为List<Integer>
   Integer[] arr = l.toArray(new Integer[0]);
   ```

4. 对于三维数组的排序，可以先比较(0,0)处元素，然后再比较(0,1)或(1,0)处元素。二维以上数组，基本类型和引用类型的操作一样的，因为都是数组

5. **补充**，Arrays.sort对基本类型使用快排（因为对于基本类型稳定性没有意义），而对对象类型使用归并排序（因为对象的稳定性很重要，另一个原因就是归并排序相比较于快排，比较次数少，移动次数多，对于对象来说比较比移动耗时）

## 二叉查找树

它的中序遍历相当于在访问一个排序的数组

## 位运算

```java
num & (num-1) // 该数最右边的1变成0
// 比如  1100&(1011) = 1000
num & (-num) // 该数从右边往左找到第一个为1，形成以该位为1，其余位都为0的数
// 比如  1100&(-1100) = 100
// 原因：正数的补码等于原码，而负数的补码是正数原码的反码+1，以1100来说，负数的反码是10...0011（共32-3个0），补码是10...0100，与1100与之后得到100
```

## while(++i ..)与while(..) ++i

两种方式虽然大部分情况下相同，但也存在着不同，即当有两个循环+其他判断条件时

```java
/* 以上两种方法看起来好像差不多，但是给出两个例子
1. A = [1,2,3], B = [1,2,3]
该情况下，两种方法效果一样
2. A = [1,2,3], B = [1,1,2,2,3]
该情况下，第一种方法无法走完B，而第二种方法能走完B。也就是说关键在于你是否每次进入while循环都想让它+1。所以我建议for循环要改成while循环时，选用第一种形式，因为第二种形式在某些情况下会引入不易察觉的bug
*/
int i = 0, j = 0;
while(true){
    while(++i < n && A[i] != B[j]);
    j++;
}
while(true){
    while(i < n && A[i] != B[j]) ++i;
    j++;
}

for(int i = 0; i < n; i++) {...}
=> int i = -1;
while(++i < n){
    ...
}
```

## 关于int整型的除法问题

1. 是否要溢出，如果要溢出，是否可以使用long来代替它

   一些特例

   ```java
   -2147483648 / -1  // 溢出特例
   -1 / -2147483648
   ```

2. 两个数的符号相同还是不同，不同的处理是不一样的

   ```java
   -1 / 2 // 两种特例
   1 / -2
   1 / 2 
   -1 / -2
   ```

## 不要浮躁

仔细考虑边界问题，特殊测试用例

## dp不是只有一维dp，二维甚至高维dp都有可能出现

## 状态压缩相关

一个数组中只有[0-9]的数字，判断每个数字出现次数是奇数还是偶数

```java
// 使用一个数字来压缩状态
int s = 0;
for(int i = 0; i < nums.length; i++){
    s ^= (1 << nums[i]);
}
```

[题目1：其中的2也用到了状态压缩](数据结构和算法/前缀和.md)

[题目2：二叉树中的伪回文路径](<https://leetcode-cn.com/problems/pseudo-palindromic-paths-in-a-binary-tree/>)

## Arryas.asList

```java
int[] nums = {1,2,3,4};
Arrays.asList(nums); // 结果为只存放一个元素（为数组）的List
Integer[] nums2 = {1,2,3,4};
Arrays.asList(nums2); // 结果正确
// 因为Arrays.asList的函数签名为 T..a，而int[]是引用类型, int是基本类型，所以不会展开
```

## 暴力解法是其他解法的基础，所以当没有思路时，先用暴力解法，再优化

## int和long的溢出

```java
System.out.println(Math.log10(Integer.MAX_VALUE)); //  10^9 + C
System.out.println(Math.log10(Long.MAX_VALUE)); //    10^18 + C
```

当m=10^5, nums[i] =  [1,10^7]时，结果为10^12，不会溢出long



## 回溯

回溯算法首先需要根据问题**确定解的空间**

## 返回结果

**返回下标与返回元素值的做法是不同的**。返回下标表示一定不能排序，因此可能用到map，返回元素值是可以排序的



## 棋盘

棋盘的一个非常容易让人迷惑的点就是用格子的哪部分来代表这个格子（常见的是用右下角，题目有[棋盘分割](https://www.acwing.com/problem/content/323/)、[电路维修](https://www.acwing.com/problem/content/177/)）

