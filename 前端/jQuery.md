# jQuery

### button

```javascript
// 设置button 的文本值
$('#node').text('保存')
// 获取button 文本的值
$('#node')[0].innerHTML
```



### css选择器

```javascript
// 1.
$('#myModal input[name=empName]')
// 2.选择css样式有school的元素
$('.school')
// 3.选择css样式有xl的元素的option子元素，且该元素是被选中的
var schools = $('.xl option:checked')
// 4. 2和3所选的元素一般有多个，遍历过程中不能直接调用val()方法，正确写法如下
$(schools[0]).val()
```



### input

```javascript
// 设置文本值
$('#node').val('保存')
```



### radio & option

```javascript
// 设置单选的选中值
$('#node').val([value])
// 设置下拉框的选中值
$('#node').val([value])
```



某标签nodeA前添加标签nodeB

```javascript
$(nodeA).before(nodeB)
```



### Ajax同时上传表单序列化参数+自定义参数

```javascript
$.ajax({
    url: '',
    method: 'PUT',
    data: $('#edit-form').serialize() +'&'+$.param({'param1': value1, 'param2':value2})
})
```



### 清空元素的内容以及移除元素自身

```javascript
$('#node').empty()  // 清空该元素所包含的子元素以及内容，但自身还存在
$('#node').remove() // 移除自身以及子元素
```

