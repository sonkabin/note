## 基础

双大括号表达式`{{}}`里面是JavaScript

指令一：强制数据绑定

```html
v-bind:src
等价于
:src
```

指令二：绑定事件监听

```html
v-on:click
等价于
@click
```

计算属性：存在缓存

```html
<input type="text" placeholder="Full Name1" v-model="fullName1"><br>
new Vue({
    el: '#demo',
    data: {
        firstName: 'a',
        lastName: 'b'
    },
    computed: {
        fullName1(){//计算属性中的一个方法，方法的返回值作为属性值
            console.log('fullName1')
            return this.firstName + ' ' +this.lastName
    	}
    }
})
```

**回调函数**：1.你定义的2.你没有调用它3.但它最终执行了

```javascript
//回调函数，当需要当前属性值时回调，根据相关的数据（相当于计算属性）计算并返回
get(){
    return this.firstName + ' ' + this.lastName
},
//回调函数，监视当前属性值的变化，当属性值发生变化时回调，更新相关的属性数据
set(value){
    const names = value.split(' ');
    this.firstName = names[0]
    this.lastName = names[1]
}
```

**变异方法**：它们将会触发视图更新：1. 调用原生的数组的对应方法2. 更新界面

- push()
- pop()
- shift()
- unshift()
- splice()
- sort()
- reverse()

**:key为什么要指定**：为了给 Vue 一个提示，以便它能跟踪每个节点的身份，从而重用和重新排序现有元素，理想的 `key`值是每项都有的唯一 id，它的工作方式类似于一个属性，所以你需要用 `v-bind`来绑定动态值

**js中的const, let, var的区别**：

- const：块级变量（也就是局部变量），用于声明常量
- let：块级变量
- var：全局变量

**ES6语法**：=>

```javascript
(x) => x + 6等价于
function(x){
    return x + 6;
};
```

**取出相关数据**

```javascript
const {searchName, persons} = this
```

**!=与!==： != 比较若类型不同**，先偿试转换类型，再作值比较，最后返回值比较结果；!== 只有在相同类型下,才会比较其值

**阻止表单的默认提交行为**：注意不是onsubmit而是submit

```html
<form action="/xxx" @submit.prevent="handle"></form>
```



## Vue-cli

```shell
vue init webpack VueDemo ： webpack打包方式，VueDemo为项目名

? Project name vue_demo				项目的必须小写
? Project description A Vue.js project
? Author sonkabin <sonkabin1996@gmail.com>
? Vue build standalone
? Install vue-router? No					暂时NO
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Set up unit tests No						NO
? Setup e2e tests with Nightwatch? No		NO
? Should we run `npm install` for you after the project has been created? (recommended) npm
```

开始

```shell
cd VueDemo
npm run dev
```

## vue eslint规则

在.eslintrc.js中的rules处配置

```javascript
rules: {
    'indent': 'off'
}
```

Vue报错：Do not use 'new' for side effects，解决方法，添加 `/* eslint-disable no-new */`

```javascript
/* eslint-disable no-new */
new Vue({
  el: '#app',
  components: {
    App
  },
  template: '<App/>'
})

```

Vue中写注释，空格不多不少只能有一个。。。。,注释之后必须有空格，函数名与括号之间要有空格

```javascript
Multiple spaces found before	
data () {  // 在组件中data必须写为函数
```



## Vue的组件

组件的data、mounted必须是一个函数：只有data的形式与vm中的形式发生了变化，其他都一致

```javascript
data: function () {
  return {
    count: 0
  }
}
或者
data () {
    return {
    	count: 0
    }
}
```



组件是可复用的 Vue 实例，且带有一个名字：如<App/>，组件间的通信不要写到无关的地方去，如

```html
<!-- 错误写法 -->
<ul class="list-group" v-for="(comment, index) in comments" :key="index" :comment="comment">
</ul>

<!-- 正确写法 -->
<ul class="list-group">
    <Item v-for="(comment, index) in comments" :key="index" :comment="comment"/>
</ul>
```

**数据在哪个组件，更新数据的行为（方法）就应该定义在哪个组件中**



**由于Vue不允许动态添加根级别的响应属性，因此必须通过预先声明所有根级别的响应数据属性来初始化Vue实例，即使是空值** 

```javascript
用v-model绑定的，必须要在data中声明
<input type="text" placeholder="请输入你的任务名称，按回车键确认" v-model="title" />
    
data () {
    return {
        title: ''
    }
}
```



**计算属性默认只有 getter（计算） ，不过在需要时你也可以提供一个 setter (监听)**

```javascript
computed: {
    //getter和setter
    isCheckAll: {
        get () { //原本应是get: function ()
            return this.completeSize === this.todos.length
        },
        set (value) {
           this.selectAllCheck(value)
        }
    },
    
    //默认只有getter
    completeSize () {
        return this.todos.reduce((preTotal, todo) => preTotal + (todo.complete?1:0), 0)
    }
    //如上写法在文档中是如下写法，因此可以推断出: function可以省略
    completeSize: function () {
      return this.todos.reduce((preTotal, todo) => preTotal + (todo.complete?1:0), 0)
    }
}
```



**深度监视**:handler的意思是处理者，handle 的意思是句柄

```javascript
watch: {
    todos: {
        deep: true, // 深度监视，普通监视是不监视对象的内部变化的，但变异方法会触发视图更新
            handler (value){
            // 通过JSON将数组转化为字符串
            window.localStorage.setItem('todos_key', JSON.stringify(value))
        }
    }
}
```



### 组件间通信

1. props

2. **绑定监听**
   1. 使用@xx="xx"与emit('xx', data)的方式只能传一层（即只能父子组件之间）

   ```html
   //方法1
   <TodoHeader @addTodo="addTodo"/>
   this.$emit('addTodo', todo)
   
   //方法2
   <TodoHeader :addTodo="addTodo"/>
   props: {
   	addTodo: Function
   }
   this.addTodo(todo)
   ```

   2. 使用事件绑定可以一层一层传


3. 安装pubsub-js ：两个方法subs（订阅消息），publish（发布消息），可以进行组件间通信

   **注：回调函数用箭头函数** 

   ```shell
   npm install --save pubsub-js 
   ```

   ```javascript
   App.vue
   mounted () {
       PubSub.subscribe('deleteTodo', (msg, index) => {
       	this.deleteTodo(index)
       })
   }
   
   TodoItem.vue
   PubSub.publish('deleteTodo', index)
   ```

4. slot：标签间的通信

5. vuex



## Ajax请求

安装axios

```shell
npm install axios --save
```

使用

```javascript
import axios from 'axios'
mounted () {
    const url = `https://api.github.com/search/repositorys?q=vu&sort=star`
    axios.get(url).then(response => {
        const result = response.data
        const mostRepo = result.item[0]
        this.repoName = mostRepo.name
        this.repoUrl = mostRepo.html_url
    }).catch(error => {
        alert('出错了')
    })
}
```



## vue-router

下载

```shell
npm install vue-router --save
```

配置路由器模块

```javascript
/*
路由器模块
 */
import Vue from 'vue'
import VueRouter from 'vue-router'

import About from '../views/About'
import Home from '../views/Home'

Vue.use(VueRouter)

export default new VueRouter({
  routes: [
    {
      path: '/about',
      component: About
    },
    {
      path: '/home',
      component: Home
    },
    {
      path: '/',
      redirect: '/about'
    }
  ]
})
```

**编写路由的步骤**

1. 定义路由组件（在views或pages文件夹下）

2. 注册（配置）路由

3. 使用路由

   ```html
   <router-link></router-link>
   <router-view></router-view>
   ```


缓存路由

```javascript
<keep-alive>
    <router-view></router-view>
</keep-alive>
```

### 向路由组件传递数据

1. 路由路径携带参数

   组件报错：Component template should contain exactly one root element. If you are using v-if on multiple elements, use v-else-if to chain them instead.

   原因：**vue模板只支持一个元素，不能并列包含两个及以上**，错误实例如下，应包含在div标签中

   ```html
   <template>
       <ul>
           <li v-for="message in messages" :key="message.id">
               <router-link :to="`/home/message/detail/${message.id}`">{{message.title}}</router-link>
           </li>
       </ul>
       <hr>
       <router-view></router-view>
   </template>
   ```

   ```javascript
   {
       path: 'message', // 相对路径
       component: Message,
       children: [
           {
               path: 'detail/:id', // :id占位符
               component: MessageDetail
           }
      ]
   }
   ```



   把文本转为数字 `const id = this.$route.params.id*1 // 把文本转为数字`

2. `<router-view msg="abc"></router-view>`属性携带参数，在子路由中用props接收

### 编程式路由导航

- this.$router.push(path)：相当于点击路由链接（可以返回到当前路由界面）
- this.$router.replace(path)：用新路由替换当前路由（不可以返回到当前路由界面）
- this.$router.back()：返回上一个路由



## vuex

作用：多组件共享状态

下载

```shell
npm install --save vuex
```

store.js：格式很固定

```javascript
/*
vuex的核心管理对象模块
 */
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

// 状态对象
const state = {
  count: 0
}
// 包含多个更新state函数的对象
const mutations = {
  // 增加的mutation
  INCREMENT (state) {
    ++state.count
  },
  // 减少的mutation
  DECREMENT (state) {
    --state.count
  }
}
// 包含多个对应事件回调函数的对象
const actions = {
  increment ({commit}) {
    // 提交到增加的mutation
    commit('INCREMENT')
  },
  decrement ({commit}) {
    commit('DECREMENT')
  },
  incrementIfOdd ({commit, state}) {
    if(state.count%2 === 1){
      commit('INCREMENT')
    }
  },
  incrementAsync ({commit}) {
    setTimeout(() => {
      commit('INCREMENT')
    }, 1000)
  }
}
// 包含多个对应事件回调函数的对象
const getters = {
  oddOrEven (state) {
    return state.count % 2 === 0 ? '偶数' : '奇数'
  }
}

export default new Vuex.Store({
  state, // 状态对象
  mutations, // 包含多个更新state函数的对象
  actions, // 包含多个对应事件回调函数的对象
  getters // 包含多个getter计算属性函数的对象
})
```

在App.vue中使用的两种方式

```javascript
<!--<p>click {{$store.state.count}} times, count is {{oddOrEven}}</p>-->
<p>click {{count}} times, count is {{oddOrEven}}</p>

computed: {
    // mapState()是一个函数
    ...mapState(['count']),
    ...mapGetters(['oddOrEven'])
    /*oddOrEven () {
      return this.$store.getters.oddOrEven
    }*/
},
methods: {
    ...mapActions(['increment', 'decrement', 'incrementIfOdd', 'incrementAsync'])
}
/*methods: {
    increment () {
      this.$store.dispatch('increment')
    }
}*/
```



**commit的方法传递的数据要用对象包起来** 

```javascript
add_todo ({commit}, todo) {
	commit(ADD_TODO, {todo})
}
//mutations中要对应包成对象
ADD_TODO (state, {todo}) { // 默认是字符串，即等价于'ADD_TODO'，要想把它变成变量，用[]把它包起来
    state.todos.unshift(todo)
}
```



**getters**：接收state作为第一个参数，也可以接收其他getter作为第二个参数

```javascript
isAllComplete (state, getters) {
    return getters.completeCount === getters.totalCount && state.todos.length > 0
}
```

