## ES6

### var、let、const

1. let有严格的作用域

   ```javascript
   {
       var a = 1
       let b = 1
   }
   console.log(a) // 1
   console.log(b) // b is not defined
   ```

   

2. var可以声明多次，let只能声明一次

   ```javascript
   var m = 1
   var m = 2
   let n = 1
   let n = 2 // b has already been declared
   console.log(m) // 2
   ```

   

3. var会变量提升，let不会变量提升

   ```javascript
   console.log(x) // undefined
   var x = 1
   console.log(y) // y is not defined
   let y = 2
   ```

4. const声明的是常量

### 解构表达式

1. 数组解构

   ```javascript
   let arr = [1, 2, 3]
   let [a, b, c] = arr
   ```

   

2. 对象解构

   ```javascript
   const person = {
       name: "sjm",
       age: "24",
       language: ["Java", "MySQL"]
   }
   const {name, age, language} = person
   console.log(name)
   // 扩展，nn是新的变量名
   const {name: nn, age, language} = person
   console.log(nn, name)
   ```

### 字符串

1. 字符串模板

   ```javascript
   let str = `
   <div>
   	<span>qqq</span>
   </div>
   `
   ```

2. 插入变量和表达式

   ```javascript
   function func(){
       return "func!"
   }
   let info = `我是${name}，今年${age + 10}岁，我说：${func()}`
   ```

### 函数优化

1. 支持默认值

2. 支持不定参数：`function fn(...values)`

3. 箭头函数（lambda表达式）

   ```javascript
   /*等价于
   var print = function(obj){
   	console.log(obj)
   }
   */
   var print = obj => console.log(obj)
   print()
   ```

### 对象优化

1. 新增API

   ```javascript
   const person = {
       name: "sjm",
       age: "24",
       language: ["Java", "MySQL"]
   }
   // Objects.values, Objects.entries
   console.log(Objects.keys(person))
   
   const target = {a:1}
   const source1 = {b:2}
   const source2 = {c:3}
   // {a:1, b:2, c:3}
   Objects.assign(target, source1, source2)
   console.log(target)
   ```

2. 声明对象简写

   ```javascript
   const name = "sjm"
   const age = 12
   const person = {name, age}
   ```

3. 对象函数属性简写

   ```javascript
   let person1 = {
       name: "sjm",
       // 箭头函数不能使用this关键字，会丢失，要用对象.属性 的方式
       eat: food => console.log(person1.name + "吃" + food),
       eat2(food){
           console.log(this.name + "吃" + food)
       }
   }
   ```

4. 对象拓展运算符

   ```javascript
   // 拷贝对象（深拷贝）
   let person = {name:"sjm", age:12}
   let someone = {...person}
   console.log(someone) // {name:"sjm", age:12}
   
   // 合并对象
   let age = {age:15}
   let name = {name:'q'}
   let person = {name:'sjm'}
   person = {...age, ...name}
   console.log(person) // // {name:"q", age:15}
   ```

5. map和reduce

   ```javascript
   map() // 接收一个函数，将原数组中的所有元素用这个函数处理后放入新数组返回
   reduce() // 为数组中每个元素依次执行回调函数，不包括数组中被删除或从未赋值的元素
   // index是当前元素的索引，array是调用reduce的函数
   arr.reduce((previousValue, currentValue, index, array) => {}, [initValue])
   ```

### Promise

封装异步请求

```javascript
let p = new Promise((resolve, reject) => {
    $.ajax({
        url: 'mock/user.json',
        success: function(data){
            resolve(data)
        },
        error: function(err){
            reject(err)
        }
    })
})
p.then((data) => {
    return new Promise((resolve, reject) => {
        $.ajax({
            url: `mock/user_courese_${data.id}.json`,
            success: function(data){
                resolve(data)
            },
            error: function(err){
                reject(err)
            }
        })
    })
}).then((data) => {
    // 后面没有异步了，所以不封装成Promise也可以
    $.ajax({
        url: `mock/courese_score_${data.id}.json`,
        success: function(data){
            // work
        },
        error: function(err){
            // err
        }
    })
}).catch(err){
    console.log("出现异常", err)
}
// 封装
get(url, data) => {
    return new Promise((resolve, reject) => {
        $.ajax({
            url: url,
            data: data,
            success: function(data){
                resolve(daata)
            },
            error: function(err){
                reject(err)
            }
        })
    })
}
get("mock/user.json")
	.then((data) => {
    	// work
    	return get(`mock/user_courese_${data.id}.json`)
	})
	.then((data) => {
    	// work
    	// 不需要返回Promise，因为没有下一步的异步操作了
    	get(`mock/courese_score_${data.id}.json`)
    })
	.catch(err => console.log("出现异常", err))
```



## Node.js

包管理工具

淘宝镜像：`npm config set registry http://registry.npm.taobao.org`