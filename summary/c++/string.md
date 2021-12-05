# string API

## 遍历

方法一

```c++
string s;
cin >> s;
for(int i = 0; i < s.length(); i++){
    cout << s[i] - '0' << endl; // 转数字
}
```

方法二(要求C++11)

```c++
for(auto c : s){
    cout << c - '0' << endl;
}
```

## substr

1. substr(start)
2. substr(start, len)

