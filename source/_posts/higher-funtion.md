title: 高阶函数及相关应用
date: 2016-04-12 15:29:39
tags: [AOP,柯理化,函数节流,分时函数,惰性函数]
categories: JavaScript
---

能成为高阶函数至少满足以下两点之一

* 函数可以作为参数被传递
* 函数可以作为返回值

<!--more-->

## 函数作为参数传递

把函数作为参数传递，这代表可以抽象出一部分容易变化的业务逻辑，形成单独的函数来作为参数传递，这样一来可以分开业务逻辑中变化与不变化的部分。

最常见的就是回调函数，比如ajax异步请求返回之后要做的一些事情，可以封装的回调函数中，等待请求完成之后执行callback

```javascript
var getUserInfo=function(userId,callback){
    $.ajax({
        url:'http://xxx.com/getUserInfo?'+userId,
        success:function(data){
            if(typeof callback=='function'){
                callback(data);
            }
        }
    })
}

getUserInfo(12345,funciton(data){
    console.log(data.userName);
})

```
除了异步请求，当一个函数不适合执行一些请求时，也可以把这些请求封装成一个函数，作为参数传递。
在页面上创建100个div

```javascript
var appendDiv=function(callback){
    for(var i=0;i<100;i++){
        var div=document.createElement('div');
        div.innerHTML=i;
        document.body.appendChild(div);
        if(typeof callback==='function'){
           callback(div);
        }
    }
}
```
想让100个div隐藏，则创建一个隐藏函数,或者其他一些操作

```javascript
append(function(node){
    node.style.display='none';

    //todo
});

```

总结一点就是抽离出业务逻辑中的不变部分与可变部分，可变部分封装成单独的函数作为参数来传递

## 函数作为返回值

函数作为返回值的场景也许在应用中会更多

### 判断数据的类型

通常判断数据类型调用`Object.prototype.toString.call(obj)`比如：

```javascript
var isArray=function(obj){
    return Object.prototype.toString.call(obj)==='[object Array]'
}
var isNumber=function(obj){
    return Object.prototype.toString.call(obj)==='[object Number]'
}
```

为了避免重复大部分代码

```javascript
var isType=function(type){
    
    return function(obj){
        return Object.prototype.toString.call(obj)==='[object '+type+']';
    }
}

var isArray=isType('Array');
isArray([1,23,4]) //true
```
## 高阶函数的应用

### 单例模式

单例模式中函数即作为参数，又作为返回值。

```javascript
var getSingle=function(fn){
    var ret;
    return function(){
        return ret||(ret=fn.apply(this,arguments));
    }
}

var getScript=getSingle(function(){
    return document.createElement('script');
});

var script1=getScript();
var script2=getScript();

console.log(script1===script2) //true

```

### 实现AOP

AOP即面向切面编程，主要独立于核心业务逻辑，通常是无关功能，包括日志统计、安全控制、异常处理。把这些功能抽离出来在通过“动态组织”的方式掺入到业务逻辑中，这样保持业务逻辑干净和高内聚性，其次是抽离出来的功能实现高复用。

通过扩展Function.prototype来做到实现AOP。

```javascript

//原函数执行前
Function.prototype.before=function(beforeFn){
    var _self=this;
    return function(){
        beforeFn.apply(this,arguments);
        return _self.apply(this,arguments);
    }
}
//原函数执行后
Function.prototype.after=function(afterFn){
    var _self=this;
    return function(){
        var ret=_self.apply(this,arguments);
        afterFn.apply(this,arguments);
        return ret;
    }
}
//比如一个用户登录，登录前先进行安全验证，登录后再打印登录日志

//登录模块
var login=function(){
    console.log('用户登录中ing....')
}
//安全验证模块
var safety=function(){
    console.log('安全验证完毕')
}
//日志统计模块
var log=function(){
    console.log('事件：登录，用户：xxx，时间：2016年4月12日15:09:10')
}

login= login.before(safety).after(log);
login();
/*
输出:
安全验证完毕
用户登录中ing....
事件：登录，用户：xxx，时间：2016年4月12日15:09:10
*/

```

其实在设计模式中这叫责任链模式。

```javascript
//利用JavaScript函数式的特性，可以很方便的实现责任链模式

Function.prototype.after=function(fn){
    var self=this;
    return function(){
        var ret=self.apply(this,arguments);
        if(ret==='nextSuccessor'){
            return fn.apply(this,arguments);
        }
    }
}
```

### 柯理化

又名currying，分部求值，一个currying的函数首先接受一些参数，接受后不会立即进行求值，而是返回另一个函数，传入的参数在闭包做作为私有静态变量被保存下来，待真正求值时，一次性求出。

比如我们统计每月的消费

```javascript
var monthlyCost=0;
var cost=function(money){
    monthlyCost+=money;
}
cost(100);
cost(200);
cost(300);

console.log(monthlyCost)
```

其实我关心的是月底该月总共消费多少，到月底统一计算，就像我们关心的工资，上一天班不是说当天就给你结算，而是到月底一起算。这就是说的月薪，有些牛x的直接来年薪。

思路：把每天的消费先存放到闭包中的私有静态变量中，到月底一起算。

```javascript
var cost=(function(){
    var costs=[];
    return function(){
        if(arguments.length===0){
            var money=0;
            for(var i=0;i<costs.length;i++){
                money+=costs[i]
            }
            return money;
        }else{
            [].push.apply(costs,arguments);
        }
    }
})();

cost(100) //没求值
cost(200) //没求值
cost(300) //没求值
cost(); //600
```

函数柯理化，当然将要被柯理化的函数要作为参数，通过一个函数将其柯理化

```javascript
var currying=function(fn){
    var costs=[];
    return function(){
        if(arguments.length===0){
            return fn.apply(this,costs)
        }else{
            [].push.apply(costs,arguments)
        }
    }
}

var cost=(function(){
    var money=0;
    return function(){
        for(var i=0;i<arguments.length;i++){
            money+=arguments[i];
        }
        return money;
    }
})()

var cost=currying(cost)

cost(100)
cost(200)
cost(300)

cost(); //600
```

先不求值，最后统一计算。

### 函数节流

在有些场景中，函数有可能被频繁的调用，而造成性能问题，比如`window.onresize`,`mousemove`等事件。

函数节流的原理：比我`window.onresize`事件，改变一次窗口大小，打印窗口大小1秒钟就执行了10次，实际上只打印1-2次就够了，这需要按时间段来忽略一些事件请求，比如在500ms内打印一次，很显然可以借助`setTimeout`函数.

```javascript
//throttle的意思是动词掐死，意思是让其慢一点执行
/**
 * [throttle description]
 * @param  {Function} fn       被延迟执行的函数
 * @param  {[type]}   interval 延迟执行的时间
 * @return {[type]}            [description]
 */
var throttle=function(fn,interval){
    var _self=fn,
        timer,
        firstTime=true;//是否是第一次调用

    return function(){
        var args=arguments,
            _me=this;
        if(firstTime){
            _self.apply(this,args);
            return fistTime=false;
        }
        if(timer){
            return false;
        }
        timer=setTimeout(function(){
            clearTimeout(timer);
            timer=null;
            _self.apply(_me,args);
        },interval||500)
    }
}

window.onresize=throttle(function(){
    console.log(1); //窗口变动一次执行2次函数
});

```

### 分时函数

比如在短时间内往页面中大量添加DOM节点，会让浏览器吃不消，会发生卡顿甚至假死，解决办法之一，节点分批创建，比如1秒创建1000个，改为200ms创建8个。、

```javascript
/**
 * [timeChunk description]
 * @param  {[type]}   arr   [创建节点用到的数据]
 * @param  {Function} fn    [创建节点的逻辑函数]
 * @param  {[type]}   count [每一批创建的节点数量]
 * @return {[type]}         [description]
 */
var timeChunk=function(arr,fn,count){
    var obj,
        t;
    var len=arr.length;

    var start=function(){
        for(var i=0;i<Math.min(count||1,arr.length);i++){
            obj=arr.shift();
            fn(obj);
        }
    }

    return function(){
        t=setInterval(function(){
            if(arr.length==0){
                return clearInterval(t)
            }
            start();
        },200)
    }
}

var arr=[];
for(var i=0;i<1000;i++){
    arr.push(i)
}

var renderDiv=timeChunk(arr,function(n){
    var div=document.createElement('div')
    div.innerHTML=n;
    document.body.appendChild(div);
},8)

renderDiv();

```

### 惰性加载函数

在各式各样的浏览器中，一些嗅探工作总是不可避免，比如在一个各个浏览器中都能够通用的事件绑定函数addEvent。

```javascript
//惰性载入函数实现方案

/**
 * [addEvent description]
 * @param {[type]} elem    [元素]
 * @param {[type]} type    [事件类型]
 * @param {[type]} handler [处理事件]
 */
var addEvent=function(elem,type,handler){
    if(window.addEventListener){
        addEvent=function(elem,type,handler){
            elem.addEventListener(type,handler,false)
        }
    }else if(window.attachEvent){
        addEvent=function(elem,type,handler){
            elem.attachEvent('on'+type,handler)
        }
    }
    addEvent(elem,type,handler);
}
```

## 总结

接下来的会系统学习一下JavaScript设计模式，基本都是通过高阶函数和闭包来实现的。但其实更关注在哪种场景适合哪种模式。