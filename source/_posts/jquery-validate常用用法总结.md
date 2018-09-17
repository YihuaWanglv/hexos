---
title: jquery-validate常用用法总结
date: 2016-01-04 17:05:28
tags: [jquery,validate]
categories: javascript
---


1. juqry validate中，如何对使用ajax提交的方式进行校验？
有两种方式
    1)使用submitHandler属性配置ajax提交，submithandler：当表单全部校验通过之后会回调配置的代码，此处也就是当校验通过之后调用ajax提交。
    2)使用valid方法，监听form的submit事件，当$('#form').valid()返回true的时候再提交。
    ```
        //通过监听form的submit事件，对form进行ajax提交。
         $('#formId').submit(function() {
             if (!$("#formId").valid()) 
                 return false;
             $(this).omAjaxSubmit({});
             return false; //此处必须返回false，阻止常规的form提交
         });
    ```
2. juqry validate中，如何对校验错误的提示信息位置和样式进行更改？
    1）js代码：
    ```
    $(function(){  
        $("#form1").validate({  
            errorPlacement: function (error, element) {  
                error.appendTo(element.parent("td"));  
            },  
            rules:{              
            }  
        });  
        $("[name$='.sign']").each(function(){  
            $(this).rules("add",{required:true,messages:{required:"至少选择一个选项"}});  
        });  
    }); 
    ```
    2）validate会默认追加一个label，样式默认是error，所以我们做的就是写一个class为error的css就可以了
    ```
        <style>  
        label.error {   
            color:Red;   
            font-size:13px;   
            margin-left:5px;   
            padding-left:16px;   
        }   
        </style> 
    ```
    3) 校验时机，可以自定义在js中想要的时候去校验
    ```
        $('.selector').click(function() {
            if ($("selector of you form").valid()) {
                alert('in');
            } else {
                alert('out');
            }
        });
        $("selector of you form").validate({
            rules: {
                receiveDayFrom: {
                    required: true
                }
            }
        }); 
    ```
    4) 例子：
    ```
        $("#form").validate({
            rules: {
                name: {
                    required: true
                },
                firstname: {
                    required: true
                }
            },
            messages: {
                name: {
                    required: "Enter name"
                },
                firstname: {
                    required: "Enter firstname"
                }
            },
            errorPlacement: function ($error, $element) {
                var name = $element.attr("name");
                $("#error" + name).append($error);
            }
        });
    ```
        or:
    ```
        $("#form").validate({
            errorLabelContainer: "#errors",
            rules: {
                name: {
                    required: true
                },
                firstname: {
                    required: true
                }
            },
            messages: {
                name: {
                    required: "Enter name"
                },
                firstname: {
                    required: "Enter firstname"
                }
            }
        });
    ```

3. 如何添加自定义的校验？

- 添加一个方法
    ```
    // 字符验证，只能包含英文、数字、下划线等字符。    
    jQuery.validator.addMethod("nameStringCheck", function(value, element) {   
         return this.optional(element) || /^[a-zA-Z0-9-_]+$/.test(value); 
    }, "只能包含英文、数字、下划线等字符");
    ```
- 在validate配置中使用新加的方法
    ```
    $('#form selector').validate({
        rules: {
            username: {
                required: true,
                nameStringCheck: true
            }
        }
    });
    ```

4. 如何使用ajax请求进行远程校验
    ```
    jQuery.validator.addMethod("checkUnique", function(value, element) {
        return validateUsernameByAjax(value, element.name, 'ajax请求地址');
    }, "此输入的值不可用");
    ```

    ```
    /**
     * 同步做用户名或邮箱的请求，检查到不可用，则返回false，否则返回true
     */
    function validateUsernameByAjax(value, fieldname, url) {
        var _request = url + "?"+fieldname+"="+value;
        var deferred = $.Deferred();//创建一个延迟对象
        $.ajax({
            url:_request,
            async:false,//要指定不能异步,必须等待后台服务校验完成再执行后续代码
            dataType:"json",
            success:function(data) {
                if (data.status === "error" || data.status === "fail") {
                    deferred.reject();
                } else{
                    deferred.resolve();  
                }
            }
        });
        //deferred.state()有3个状态:pending:还未结束,rejected:失败,resolved:成功
        return deferred.state() == "resolved" ? true : false;
    }
    ```

    ```
    $('#form selector').validate({
        rules: {
            username: {
                checkUnique: true
            }
        }
    });
    ```
