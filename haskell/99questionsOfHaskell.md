## question1

Find the last element of a list

```haskell
λ> myLast [1,2,3,4]
4
λ> myLast ['x','y','z']
'z'
myLast :: [a] -> a
myLast = foldl1 (\acc x -> x)
```

## question2

找List的倒数第二个元素

```haskell
λ> myButLast [1,2,3,4]
3
λ> myButLast ['a'..'z']
'y'
myButLast :: [a] -> a
myButLast xs = xs !! (length xs - 2)
```



## question3

 Find the K'th element of a list. The first element in the list is number 1.

```haskell
λ> elementAt [1,2,3] 2
2
λ> elementAt "haskell" 5
'e'
elementAt :: [a] -> Int -> a
elementAt (x:xs) k
  | k == 1    = x
  | otherwise = elementAt xs (k-1)
```

## question4

Find the number of elements of a list.

```haskell
λ> myLength [123, 456, 789]
3
λ> myLength "Hello, world!"
13
myLength :: (Num b) => [a] -> b
myLength [] 	= 0
myLength (x:xs) = myLength xs + 1
```

## question5

翻转List

```haskell
λ> myReverse "A man, a plan, a canal, panama!"
"!amanap ,lanac a ,nalp a ,nam A"
λ> myReverse [1,2,3,4]
[4,3,2,1]
myReverse :: [a] -> [a]
myReverse = foldl (\acc x -> x:acc) []
```

## question6

Find out whether a list is a palindrome

```haskell
λ> isPalindrome [1,2,3]
False
λ> isPalindrome "madamimadam"
True
λ> isPalindrome [1,2,4,8,16,8,4,2,1]
True
isPalindrome :: (Eq a) => [a] -> Bool
isPalindrome []				   = True
isPalindrome [x]			   = True
isPalindrome (x:xs)
  | x /= xs !! (length xs - 1) = False
  | otherwise                  =  isPalindrome (init xs)
```

