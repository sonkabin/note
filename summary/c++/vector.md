## 初始化

1. 二维初始化：`vector<vector<int> a(n, vector<int>(n, -1)) `

2. 初始化大小10，赋值为10：`vector<int> a(10, 10)`

3. 重新初始化大小为 n，赋值为-1：`a.resize(n, -1)`

4. 初始化不同的值

   ```c++
   vector<int> a = {1, 2, 3};
   
   vector<vector<int>> res;
   // 匿名
   res.push_back(vector<int>{1, 2});
   ```

   