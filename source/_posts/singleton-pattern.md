title: 通用的惰性单例
date: 2016-04-13 17:10:15
tags: 
categories: JavaScript
---

在全局保证一个类仅有一个实例，并提供一个访问它的全局访问点。

别的语言不说，先谈谈在JavaScript中全局只有一个，并且直接可用，那么直接定义即可，比如`var o={}`,同时满足以上2点，但全局变量问题很多，比如易造成命名空间的污染，全局变量容易被覆盖等。

<!--more-->

## 使用命名空间

使用命名空间可以有效的减少全局变量的数量，方式就是用字面量对象。

```javascript
var Namespace={
    A:{
      AA:{
        AAA:{}
      }  
    },
    B:{
        BB:{
            BBB:{}
        }
    }
}
```
也可以动态的创建命名空间

```javascript
var Namespace={};
    Namespace.register=function(names){
    if(!names){ return;}
    var parts=names.split('.'),
        cur=Namespace;
    for(var i=0,len=parts.length;i<len;i++){
        if(!cur[parts[i]]){
            cur[parts[i]]={};
        }
        cur=cur[parts[i]];
    }
}

Namespace.register('A.AA');
Namespace.register('A.AA.AAA');
Namespace.register('B');
Namespace.register('B.BB');
/*
等价于
var Namespace={
    A:{
        AA:{
            AAA:{}
        }
    },
    B:{
        BB:{}
    }
}
    
*/

```

## 使用闭包封装私有变量

```javascript
var user=(function(){
    var __name='aaa',
        __age=22;
    return {
        getUserInfo:function(){
            return __name+'-'+__age;
        }
    }
})()
```

静态私有变量`__name`,`__age`在外面是无法直接访问的，也同样避免了全局命名空间的污染。

## 惰性单例

惰性单例是在需要时才会被创建。
比如登录框，全局唯一，用时弹出，不用时隐藏

```html
<<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
    <button id="loginBtn">登录</button>
    <script type="text/javascript">

        var loginLayer=(function(){
            var div=document.createElement('div');
            div.innerHTML='登录窗口';
            div.style.display='none';
            document.body.appendChild(div);
            return div;
        })();

        document.getElementById('loginBtn').onclick=function(){
           loginLayer.style.display='block';
        }

    </script>
</body>
</html>

```

这种方式问题是页面加载时`loginLayer`对象就被创建了，有时用户可能不登录就浏览一下页面，这样无形中增加了DOM操作。
惰性单例就是用时才执行

```javascript
var createLoginLayer=(function(){
    var div;
    return function(){
        if(!div){
            div=document.createElement('div');
            div.innerHTML='登录窗口';
            div.style.display='none';
            document.body.appendChild(div);
        }
        return div
    }
})();

document.getElementById('loginBtn').onclick=function(){
    var loginLayer=createLoginLayer();
    loginLayer.style.display='block';
}
```

这种方式虽然满足单例模式，但是比如我要创建一个唯一的iframe，就要需把`createLoginLayer`方法炮制一遍，违反职责单一的原则，这样就需要把单例中的逻辑方法抽离出来，作为参数传入单例函数`getSingle`中

```javascript

//管理单例职责
var getSingle=function(fn){
    var instance;
    return function(){
        return instance|| (instance=fn.apply(this,arguments));
    }
}
//业务职责
var creatLoginLayer=function(){
    var div=document.createElement('div');
    div.innerHTML='登录窗口';
    div.style.display='none';
    document.body.appendChild(div);
    return div;
}

var createSingleLayer=getSingle(creatLoginLayer);

document.getElementById('loginBtn').onclick=function(){
    var loginLayer=createSingleLayer();
    loginLayer.style.display='block';
}

```

单例模式的用途远不止于此，比如页面渲染完一个列表之后（通过ajax往列表中追加数据），接下来给列表绑定click事件,只需要第一次渲染时绑定一次，但是又不想去判断是否是第一次去渲染，如果借助于JQuery，给节点绑定one事件

```javascript

var bindEvent=function(){
    $('#list').one('click','li',function(){
        //doSomething
    })
}

var renderList=function(){
    console.log('开始渲染列表');
    bindEvent();
}

renderList();
```

如果用getSingle函数也能达到一样的效果

```javascript

var bindEvent=getSingle(function(){
     docuent.getElementById('list').onclick=function(event){
        var target=event.target;
        if(target.tagName.toUpperCase()=='li'){
            //do some thing
        }
     }
     return true;
})

var renderList=function(){
    console.log('开始渲染列表');
    bindEvent();
}

renderList();
renderList();
renderList();

```

可以看到renderList，bindEvent分别执行了3次，但列表实际只绑定了一次click事件。

## 总结

惰性单例注意点是单例管理职责，业务逻辑职责（创建对象）等要分开，这两个组合起来才是真正的单例模式