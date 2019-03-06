# Thymeleaf

要让Thymeleaf解析，必须把html文件放在template文件夹下

### 2019-1-21

表单回显选中单选按钮

```html
<input class="form-check-input radio-template" type="radio" name="sex" value="0" th:checked="${employee.sex==0}">
<input class="form-check-input radio-template" type="radio" name="sex" value="1" th:checked="${employee.sex==1}">
```

使用JavaScript inline

```html
<script th:inline="javascript">
    var education = [[${employee.education}]]
    var edus = education.split(';')
    // 外部不能用$
    jQuery(document).ready(function($){
        //把这段代码放在这里（在这里面依旧可以用$）;
        $('#education').empty()
        for (var i in edus) {
            var input = $('<input type="text" class="col-sm-9 form-control">').val(edus[i])
            if (i != 0) {
                input.addClass('offset-sm-2')
            }
            $('#education').append(input)
        }
    })
</script>
```

