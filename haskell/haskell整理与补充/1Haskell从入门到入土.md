# Haskell从入门到入土

## 开始做菜前的准备

安装`sudo apt-get install haskell-platform`

打开ghc交互模式，输入`ghci`进入命令行，输入`:q`退出

`Prelude`在输入什么后会变长，所以输入`:set prompt "ghci> "`比较好

在ghci下，我们可以使用let关键字来定义一个常量。在ghci下执行`let a =1`与在脚本中编写a=1是等价的

Haskell是静态类型、惰性的编程语言

函数式编程最大的特点是：不是告诉程序怎么做，而是告诉程序**问题是什么**

不能用Tab来缩进，只能用空格



## 函数

1. \* 就是一个将两个数相乘的函数，称为中缀函数，Haskell中大部分为前缀函数，调用形式为 `函数名, 空格, 空格分隔的参数表`，如`succ 8`、`max 9 10`。也可以通过反引号的形式将前缀函数括起，以中缀形式调用，如

    ```haskell
    10 `div` 2 
    等价于
    div 10 2
    ```

2. 在haskell中，函数的调用必使用空格，例如`bar (bar 3)`，它并不表示以bar和3两个参数去调用bar，而是以bar 3所得的结果作为参数去调用bar。在C中，就相当于`bar(bar(3))`

3. 函数有最高优先级

4. 声明函数然后保存为`xx.hs`。装载函数：`:l xx`

5. 函数的首字母不能大写

6. 没有参数的函数通常被称作“定义”（或者“名字”），一旦定义，`f和"foo, bar"完全等价`，**它的值不可以修改**

   ```haskell
   foo = "foo, bar"
   ```

7. 

可以在其他函数中调用你编写的函数，haskell中的函数并没有顺序，所以先声明doubleUs还是先声明doubleMe都是同样的

```haskell
doubleMe x = x + x
doubleUs x y = doubleMe x + doubleMe y
```

## if语句

1. Haskell中if语句的else部分是不可省略

2. if语句其实是个表达式，表达式就是返回一个值的一段代码：5是个表达式，它返回5；x+y是个表达式，它返回x+y的结果。由于else是强制的，if语句一定会返回某个值

   ```haskell
   doubleSmallNumber x = if x > 100 then x else x*2
   在前一个函数的所有结果加1，若去掉括号表示只在<100的时候加1
   doubleSmallNumber' x = (if x > 100 then x else x*2) + 1
   ```


## List入门

1. 字符串实际上就是一组字符的List，"Hello"是['H', 'e', 'l', 'l', 'o']的语法糖
2. **将两个List合并可以通过++运算符实现**。**注意**：在使用++运算符处理长字符串时要格外小心(对长List也是同样)，Haskell会遍历整个的List(++符号左边的那个)
3. 用`:`在List前端插入元素，`5:[1,2,3,4]`，只能一个一个插入，而`[1,2,3]其实是1:2:3[]的语法糖`
4. 按照索引取得List中的元素，使用`!!`
5. List可以套娃List
6. 当List内装有可比较的元素时，使用 > 和 >=可以比较List的大小。它会先比较第一个元素，若它们的值相等，则比较下一个，以此类推。
7. `head/last`取首尾，`tail/init`取去除首尾的结果；`length`返回List的长度，`null`检查一个List是否为空，`reverse`反转；`take 2 [2,1,2,5,6]`取前2个元素，`drop n list`丢弃前n个元素；`maximum/minimum/sum/product`；`elem`判断一个元素是否在包含于一个List，通常以中缀函数的形式调用它

## 使用Range

生成List与取无限长List中的数。另外避免使用浮点数

```haskell
[1..10] 等价于 [1,2,3,4,5,6,7,8,9,10]
[1,3,..20] 等价于 [1,3,5,7,9,11,13,15,17,19]
take 8 [4,8,..] 等价于 [4,8,12,16,20,24,28,32]
take 8 (cycle [1,2,3]) 等价于 [1,2,3,1,2,3,1,2]
take 8 (repeat 5)	等价于 [5,5,5,5,5,5,5,5]
replicate 3 10 等价于 [10,10,10]
```

## List comprehension

1. 实现 $S = \{2*x | x \in N, x \leq 10\}$ 

    ```haskell
    [x*2 | x <- [1..10]] 取前10个偶数
    [x*2 | x <- [1..10], x*2 >= 12] 取乘2之后大于等于12的元素（,后面加过滤操作）
    ```

2. 除了多个限制条件，也可以从多个List中取值进行组合

   ```haskell
   [x+y | x <- [3,5,7], y <- [2,3,4]]
   结果为[5,6,7,7,8,9,9,10,11]，即3*3=9个数
   当然也可以为它加入限制条件
   [x+y | x <- [3,5,7], y <- [2,3,4], x+y > 5, x+y < 10]
   结果为[6,7,7,8,9,9]
   ```

3. 嵌套的List过滤

   ```haskell
   let xxs = [[1..10], [8..15], [7..11]]
   [[x | x <- xs, odd x] | xs <- xxs]
   结果为[[1,3,5,7,9],[9,11,13,15],[7,9,11]]
   ```


## Tuple

1. zip：取两个List将他们交叉配对形成tuple的List

   ```haskell
   zip [1..] ["ff", "a", "ww"] 
   结果为[(1,"ff"),(2,"a"),(3,"ww")]
   ```

2. 应用：取得三边长度皆为整数且小于等于10，周长为24的直角三角形

   ```haskell
   1.取得满足所有三边长度皆为整数且小于等于10的tuple
   [(a, b, c) | a <- [1..10], b <- [1..10], c <- [1..10]]
   2.满足直角条件和周长条件
   [(a, b, c) | a <- [1..10], b <- [1..10], c <- [1..10], a^2 + b^2 == c^2, a < b, b < c, a+b+c == 24]
   ```

## 补充知识

使用`:info`查看某个操作符的优先级（1最低，9最高）和结合性（infixl：左结合；infixr：右结合）

```haskell
ghci> :info (+)
class Num a where
  (+) :: a -> a -> a
  ...
        -- Defined in ‘GHC.Num’
infixl 6 +
```



`^`是对于整数的，浮点数作为指数需要用`**`

```haskell
ghci> let e = exp 1
ghci> e ** pi - pi
19.99909997918947
```



分数，此外`:set +t`和`:unset +t`，显示和隐藏类型

```haskell
ghci> :set +t
ghci> :m Data.Ratio
ghci> 18 % 7
18 % 7
it :: Integral a => Ratio a
```



## 练习

1. 统计行数

   保存为WC.hs

   ```haskell
   main = interact wordCount
       where wordCount input = show (length (lines input)) ++ "\n"
   ```

   

   新建a.txt，内容为

   ```
   Teignmouth, England
   Paris, France
   Ulm, Germany
   Auxerre, France
   Brunswick, Germany
   Beaumont-en-Auge, France
   Ryazan, Russia
   ```

   

   在shell执行`runghc WC <  a.txt`

2. 修改例子`WC.hs`，使得可以计算一个文件中的单词个数

   ```haskell
   main = interact wordCount
     where wordCount input = show (length $ words input) ++ "\n"
   ```

   

3. 再次修改`WC.hs`，可以输出一个文件的字符个数

   ```haskell
   main = interact wordCount
     where wordCount input = show (length input) ++ "\n"
   ```

   