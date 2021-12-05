## MVVM思想

开发人员只需关注于Model和View的开发；VM是Model-View，它实现当Model或View更新时的双向更新，是Model和View的中间一层

## 指令

1. v-text, v-html：直接写在标签上

2. v-bind：绑定属性，比如href、class、style，可以用对象形式，对于 class和style 做了增强，可以简写为`:href`

   ```html
   <div id="app">
   	<a v-bind:href="link">go</a>
       <!-- font-size 也可以用驼峰式命名-->
       <span v-bind:style="{color:color, 'font-size': size}">Hello</span>
   </div>
   <script>
       let vm = new Vue({
           el:'#app',
           data:{
               link: 'http://www.baidu.com',
               color: 'red',
               size: '30px'
           }
       })
   </script>
   ```

3. v-model是双向绑定，常用于表单