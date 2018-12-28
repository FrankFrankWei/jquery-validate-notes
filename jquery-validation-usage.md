[官方网站](https://jqueryvalidation.org/documentation/)
# jQuery Validate插件介绍
jQuery Validate插件提供了强大的表单验证和错误信息提示功能，让客户端表单验证变得更简单，该插件内置诸多常用验证方法，还可自定义验证方法/规则和错误信息及其显示。默认错误信息为英文，支持多种语言，只需要下载对应的文件。

---

### 1.内置的验证方法（method）
- required - 必需
- remote - ajax请求远程资源（如api）验证元素的有效性
- minlength - 最小长度
- maxlength - 最大长度
- rangelength - 长度范围
- min - 最小值
- max - 最大值
- range - 取值范围
- step - 数值的步长，如step:10，是要求数值为10的倍数
- email - 邮箱格式
- url - url格式
- date - 日期格式
- dateISO - ISO日期格式
- number - 数值（整数、小数、负数等）
- digits - 整数
- equalTo - 等于另一个元素的值

还有更多的方法以附加(add-ons)的形式提供，放在*additional-methods.js*中，下列是部分额外方法：
- accept - 指定上传文件的mime-types
- creditcard - 信用卡
- extension - 文件后缀
- phoneUS - 美国电话号码
- require_from_group - 组中指定数量的字段必填/选 （demo用上）

完整的方法列表可以查看在Github上的[仓库](https://github.com/jquery-validation/jquery-validation/tree/master/src/additional)

demo用上尽量多的属性

内置的验证方法也可以使用`jQuery.validator.methods`重定义。如重定义`email`
```javascript
$.validator.methods.email = function(value, element) {
    return this.optional(element) || /[a-z]+@[a-z]+\.[a-z]+/.test(value);
}
```

---
### 2.部分方法的重点解析：

- [required](https://jqueryvalidation.org/required-method/)
此方法有三种重载`required()`, `required(dependency-expression)`, `required(dependency-callback)`，返回布尔值。可用于text, radio, checkbox, select等控件。
#### 1. 总是必需
```javascript
$("#myform").validate({
    rules: {
        field: {
            required: true
        }
    }
});
```

#### 2. 条件必需
当复选框勾选时该字段为必填/选
```javascript
$("#myform").validate({
    rules: {
        field: {
            required: "#checkboxid:checked"
        }
    }
});
```

#### 3.函数
当age的值小于13时，parent字段为必需
```javascript
$("#myform").validate({
    rules: {
        age: {
            required: true,
        },
        parent: {
            required: function (element) {
                return $("#age").val() < 13;
            }
        }
    }
});
```
---
- [remote](https://jqueryvalidation.org/remote-method/)
例1，设定email字段为必需、邮箱格式并以ajax方式请求`check-email.php`进行验证
```javascript 
$("#myform").validate({
    rules: {
        email: {
            required: true,
            email: true,
            remote: "check-email.php"
        }
    }
});
```

例2，设定email字段为必需、邮箱格式并以ajax方式请求`check-email.php`进行验证，并且请求方式设置为`post`，将username字段的值也发送到请求地址。
```javascript 
$("#myform").validate({
    rules: {
        email: {
            required: true,
            email: true,
            remote: {
                url: "check-email.php",
                type: "post",
                data: {
                    username: function () {
                        return $("#username").val();
                    }
                }
            }
        }
    }
});
```
---
- [equalTo](https://jqueryvalidation.org/equalTo-method/)
password_again的值与password相等
```javascript
$("#myform").validate({
    rules: {
        password: "required",
        password_again: {
            equalTo: "#password"
        }
    }
});
```
---
- [require_from_group](https://jqueryvalidation.org/require_from_group-method/)
三个号码至少要填一个才能通过验证：
```html
<label for="mobile_phone">Mobile phone: </label>
<input class="left phone-group" id="mobile_phone" name="mobile_phone">
<br/>
<label for="home_phone">Home phone: </label>
<input class="left phone-group" id="home_phone" name="home_phone">
<br/>
<label for="work_phone">Work phone: </label>
<input class="left phone-group" id="work_phone" name="work_phone">
<br/>
```
```javascript
$("#myform").validate({
    rules: {
        mobile_phone: {
            require_from_group: [1, ".phone-group"]
        },
        home_phone: {
            require_from_group: [1, ".phone-group"]
        },
        work_phone: {
            require_from_group: [1, ".phone-group"]
        }
    }
});
```
---
### 3.自定义验证方法
[jQuery.validator.addMethod(name, method [, message])](https://jqueryvalidation.org/jQuery.validator.addMethod/)
例1：验证输入是否以域名`http://mycorporatedomain.com`开头的方法.
```javascript
jQuery.validator.addMethod("domain", function (value, element) {
    return this.optional(element) || /^http:\/\/mycorporatedomain.com/.test(value);
}, "Please specify the correct domain for your documents");
```

例2：验证输入值是否等于给定的另外两个参数的和的方法.
```javascript
jQuery.validator.addMethod("math", function (value, element, params) {
    return this.optional(element) || value == params[0] + params[1];
}, jQuery.validator.format("Please enter the correct value for {0} + {1}"));
```

例3：重用内置方法添加自定义内容
```javascript
// alias required to cRequired with new message
 $.validator.addMethod("cRequired", $.validator.methods.required,
   "Customer name required");
 // alias minlength, too
 $.validator.addMethod("cMinlength", $.validator.methods.minlength,
   // leverage parameter replacement for minlength, {0} gets replaced with 2
   $.validator.format("Customer name must have at least {0} characters"));
```
---
### 4.验证规则（rules）
验证规则代表验证方法的集合。用于将多个方法应用到同一个输入元素上。如：
```javascript 
$("#myform").validate({
    rules: {
        email: {
            required: true, 
            email: true,
            minlength: 50
        }
    }
});
```

我们还可以使用[jQuery.validator.addClassRules()](https://jqueryvalidation.org/jQuery.validator.addClassRules/)打包多个规则为一个“复合类方法”（*compound class method*）：
```javascript
// combine them both, including the parameter for minlength
 $.validator.addClassRules("customer", { cRequired: true, cMinlength: 2 });
```
而后可以将方法附加到元素的class里面：
```html
 <input name="customer1" class="customer">
 <input name="customer2" class="customer">
 <input name="customer3" class="customer">
```
---
### 4. 常见问题
#### （1）字段名包含括号/点号的，需要用双引号将字段名括起来：
```javascript
$("#myform").validate({
    rules: {
        // no quoting necessary
        name: "required",
        // quoting necessary!
        "user[email]": "email",
        // dots need quoting, too!
        "user.address.street": "required"
    }
});
```
 
#### （2）无限递归
```javascript
$("#myform").validate({
    submitHandler: function (form) {
        // some other code
        // maybe disabling submit button
        // then:
        $(form).submit();
    }
});
```
这段代码会无限递归，因为`$(form).submit()`会再次触发验证。应该使用`form.submit()`来触发表单的原生提交事件（native submit event），而不是验证事件。

所以，正确的代码应该是这样：
```javascript
$("#myform").validate({
    submitHandler: function (form) {
        // some other code
        // maybe disabling submit button
        // then:
        form.submit();
    }
});
```
### 3.示例
为了尽可能多的用到内置的验证方法，例子的内容仅做演示，跟实际并不一定一致。
1、基本使用，包含自定义方法和自定义错误信息
2、自定义错误信息的显示 （errorContainer，与noty结合）
3、
