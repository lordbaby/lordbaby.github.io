title: 策略模式的应用
date: 2016-04-14 17:30:06
tags:
categories: JavaScript
---

策略策略字面上的意义可以推敲出肯定不止一种，那么放到程序中，那就是实现方法有多种。话说程序追求低耦合、高内聚，那么就降低耦合度来说，这种策略（即多种实现方法）需要封装在一个对象中，到时想用啥就用啥。其实说白了就是方便维护及提高程序可读性。

<!--more-->

举个栗子

公司年终对针对员工的kpi来进行发放奖金，比如考核等级分S，A，B，对应的就是（4*salary，3*salary，2*salary）

```javascript
//可能会这样写
 var calculateBounds=function(level,salary){
    if(level==='S'){
        return 4*salary;
    }
    if(level==='A'){
        return 3*salary;
    }
    if(level==='B'){
        return 2*salary;
    }
 }
```

如果新增kpi level就得修改上述代码，导致代码臃肿，别的地方如果用到概算法也无法复用，可见上述代码可维护复用性性比较差。

为了方便维护，那把策略放到一个对象中，到时需要什么策略就添加就添加什么策略。

```javascript

var strategies={
    "S":function(salary){
        return 4*salary;
    },
    "A":function(salary){
        return 3*salary;
    },
    "B":function(salary){
        return 2*salary;
    }
}

var calculateBounds=function(level,salary){
    return strategies[level]&&strategies[level](salary);
}

```

`策略模式的思想：定义一系列算法，把它们一个个封装起来，并且使它们可以互相替换`

在实际的开发中仅仅封装算法有点大材小用，使用策略模式可以封装业务规则，刚好在Web开发中表单校验是一个常见的话题

比如某个注册页面，在注册之前有如下几条校验规则

* 用户名不为空
* 密码长度不能少于6位
* 手机号必须符合格式

```html
<form action="http://xx.com/register" method="post" id="regForm">
    用户名：<input type="text" name="name" >
    密码：  <input type="password" name="password">
    手机：  <input type="text" name="phone">
</form> 
<script type="text/javascript">
    var regForm=document.getElementById('regForm');

    //策略定义
    var strategies={
        isNotEmpty:function(value,errorMsg){
            if(value===''){
                return errorMsg;
            }
        },
        minLength:function(value,length,errorMsg){
            if(value.length<length){
                return errorMsg;
            }
        },
        isMobile:function(value,errorMsg){
            if(!/^1[3][5][8][7][0-9]{9}$/.test(value)){
                return errorMsg;
            }
        }
    }

    //给元素添加校验规则 还需要通过一个辅助类 Validator进行统一添加
    
    var Validator=function(){
        this.cache=[];
    }
    Validator.prototype={
        constructor:Validator,
        //绑定校验规则
        add:function(dom,rules){
            var self=this;
            //支持绑定多个规则
            for(var i=0,rule;rule=rules[i++];){
                (function(rule){
                    var strategyArr=rule.strategy.split(':'), //比如 minLength:6 
                        errorMsg=rule.errorMsg||'';
                    self.cache.push(function(){
                        //取策略名称
                        var strategy=strategyArr.shift();
                        //向数组头部添加校验元素的value
                        strategyArr.unshift(dom.value);
                        strategyArr.push(errorMsg);

                        return strategies[strategy]&&strategies[strategy].apply(dom,strategyArr);
                    })

                })(rule)
            }
        },
        //校验
        valid:function(){
            for(var i=0,validFunc;validFunc=this.cache[i++];){
                var errorMsg=validFunc();
                if(errorMsg){
                    return errorMsg;
                }

            }
        }
    }
    
    //校验函数
    var validateFunc=function(){
        var validator=new Validator();
        validator.add(regForm.name,
        [{
            strategy:'isNotEmpty',
            errorMsg:'用户名不为空'
        }]);
        validator.add(regForm.password,
        [{
            strategy:'isNotEmpty',
            errorMsg:'密码不为空'
        },{
            strategy:'minLength:6',
            errorMsg:'密码长度不小于6位'
        }]);
        validator.add(regForm.phone,
        [{
            strategy:'isMobile',
            errorMsg:'手机格式不正确'
        }]);

        return validator.valid();

    }

    regForm.onsubmit=function(){
        var errorMsg=validateFunc();
        if(errorMsg){
            console.log(errorMsg);
            return false;
        }
    }

</script>
```


总结：策略模式在实践中不仅可以封装算法，是要在分析过程中需要在不同时间应用不同的业务规则，就可以考虑是要策略模式来处理各种变化。
