title: 发布订阅模式
date: 2016-04-26 15:02:40
tags:
categories: JavaScript
---

发布订阅模式在web应用中是最常见的模式，广泛应用于异步编程，是用来代替回调函数的最佳方案。主要场景是用户（订阅者）想知道某个事件发生的结果，而不知道何时发生，这是只订阅一次，到时事件发生时就会通知用户。

<!--more-->

## DOM监听事件

DOM监听事件是最典型的发布-订阅模式，比如给绑定一个click事件，当触发时click时 通知订阅用户完成一些事情。

```javascript
document.body.addEventListener('click',function(){
    console.log('发布订阅消息')
},false);

document.body.click();
```

## 自定义事件

自定事件就是有一个对象，可以注册自定义事件和触发已注册的自定义事件。跟发布订阅一样。主要要点有：

1. 指定谁充当发布者
2. 发布者中定义一个缓存列表（数组），来缓存订阅的函数，以便回调。
3. 发布消息时，遍历缓存列表，依次触发缓存列表中的函数。

场景：公司老板不在，员工在上班期间没有工作而做其他事情，老板突然回来看到员工在忙其他事情而大发雷霆，于是员工通知前台让其在老板回公司时通知他。

```javascript
//发布者
var frontGirl={
    eventCache:[],
    //订阅
    register:function(fn){
        this.eventCache.push(fn);
    },
    trigger:function(){
        for(var i=0,fn;fn=this.eventCache[i++];){
            fn.apply(this,arguments);
        }
    }

};

//小李
frontGirl.register(function(){
    //回调函数，即到时前台通知的内容
    console.log('小李:老板来了')
})
//小张
frontGirl.register(function(){
    console.log('小张:老板来了')
})
//小王
frontGirl.register(function(name){
    console.log('小王:老板来了')
})

setTimeout(function(){
    //前台女孩通知订阅者
    frontGirl.trigger();
},5000)

//五秒钟后老板来了
//小李:老板来了
//小张:老板来了
//小王:老板来了

```

由于公司又分各种部门，当一个部门的领导回来，他之约束他部门下的员工，比如小张属于BossA，小李属于BossB，小王属于BossC。他们要求前台监听各自的boss，当自己的boss来了的时候通知自己。这样就需要加一个key来区别

```javascript
var frontGirl={
    eventCache:{},
    register:function(key,fn){
        if(!this.eventCache[key]){
            this.eventCache[key]=[];
        }
        this.eventCache[key].push(fn)
    },
    trigger:function(){
        var key=Array.prototype.shift.call(arguments),
            fns=this.eventCache[key];

        if(!fns||fns.length===0){
            return false;
        }
        for(var i=0,fn;fn=fns[i++];){
            fn.apply(this,arguments)
        }
    }
}

frontGirl.register('BossA',function(data){
    console.log(data);
});
frontGirl.register('BossB',function(data){
    console.log(data);
});
frontGirl.register('BossC',function(data){
    console.log(data);
});
//2秒钟后BossA来了
setTimeout(function(){
    frontGirl.trigger('BossA','小张，BossA来了')
},2000);
//10秒钟后BossA来了
setTimeout(function(){
    frontGirl.trigger('BossB','小李，BossB来了')
},10000);
//20秒钟后BossA来了
setTimeout(function(){
    frontGirl.trigger('BossC','小王，BossC来了')
},20000);

//小张，BossA来了
//小李，BossB来了
//小王，BossC来了
```

## 发布-订阅模式的通用实现

```javascript
var eventPubSub={
    eventCache:{},
    register:function(key,fn){
        if(!this.eventCache[key]){
            this.eventCache[key]=[];
        }
        this.eventCache[key].push(fn)
    },
    trigger:function(){
        var key=Array.prototype.shift.call(arguments),
            fns=this.eventCache[key];

        if(!fns||fns.length===0){
            return false;
        }
        for(var i=0,fn;fn=fns[i++];){
            fn.apply(this,arguments)
        }
    }
}
```

突然frontGirl由于通风报信被领导发现，给辞退了，很遗憾，他们决定请保洁阿姨来代替前台，通知他们boss的到来

```javascript
//保洁员
var cleaner={};
//给保洁员安装发布订阅模式
var installPubSub=function(obj){
    for(var p in eventPubSub){
        obj[p]=eventPubSub[p]
    }
}
```

有时被警告过的员工改过自新，取消订阅的事件，于是需要添加一个订阅事件移除功能

```javascript
eventPubSub.remove=function(key,fn){
    var fns=this.eventCache[key];
    if(!fns){
        return false;
    }
    if(!fn){
        fns&&(fns.length=0); //如果没有传入具体的函数，对应的key中所有函数都清除
    }else{
        for(var i=fns.length-1;i>=0;i--){
            var _fn=fns[i];
            if(_fn===fn){
                fns.splice(i,1); //移除
            }
        }
    }


}

eventPubSub.register('BossA',function(data){
    console.log(data);
});
eventPubSub.register('BossB',fnB=function(data){
    console.log(data);
});
//移除对bossB的监听
eventPubSub.remove('BossB',fnB)
eventPubSub.register('BossC',function(data){
    console.log(data);
});
//2秒钟后BossA来了
setTimeout(function(){
    eventPubSub.trigger('BossA','小张，BossA来了')
},2000);
//10秒钟后BossA来了
setTimeout(function(){
    eventPubSub.trigger('BossB','小李，BossB来了')
},10000);
//20秒钟后BossA来了
setTimeout(function(){
    eventPubSub.trigger('BossC','小王，BossC来了')
},20000);

//小张，BossA来了
//小王，BossC来了
```

## 栗子-网站登录

一个电商网站，有header头部，nav导航，消息列表，购物车模块等。这几个模块渲染都需要ajax异步请求登录后获取用户信息后，至于ajax什么时候成功返回信息这个不得所知。你可能写道登录成功后的success回调函数中

```javascript
login.succ(function(data){
    header.render(data);
    nav.render(data);
    message.render(data);
    cart.render(data);
})
```

登录模块是A写的，但A得知道header NAV message cart等的模块名称以及对外暴漏的渲染接口，有一天A去度假，突然项目加了个收货地址管理的模块，打电话让A加上，A又得翻开好几个月前的代码来加上。这些突如其来的业务让A烦不胜烦，提出离职。

但通过发布-订阅模式重构后，模块耦合性降低，模块只需要对自己感兴趣的模块注册即可。

```javascript
//登录模块
login.succ(function(data){
    //发布
    eventPubSub.trigger('loginSucc',data);
})
//header模块
var header=(function(){
    //订阅
    login.register('loginSucc',function(data){
        header.render(data)
    })
    
    return {
        render:function(data){
            console.log(data)
        }
    }
})()

//nav模块
var nav=(function(){
    //订阅
    login.register('loginSucc',function(data){
        header.render(data)
    })
    
    return {
        render:function(data){
            console.log(data)
        }
    }
})()

//nav模块
var address=(function(){
    //订阅
    login.register('loginSucc',function(data){
        header.render(data)
    })
    
    return {
        render:function(data){
            console.log(data)
        }
    }
})()

```

作为登录模块的开发永远不需要关心其他模块的行为了。

## 全局的发布订阅模式

很显然上面的都是针对某个对象的发布订阅，像前台，保洁员，登录等。同样需要一个全局的发布订阅对象，实现订阅者不需要了解信息来自哪个发布者，发布者不需要关心信息推送给哪个订阅者。

```javascript
var PubSub=(function(){
    
    var eventCache={},
        listen,
        trigger,
        remove;
    
    listen=function(key,fn){
        if(!eventCache[key]){
            eventCache[key]=[];
        }
        eventCache[key].push(fn)
    }
    
    trigger=function(){
        var key=Array.prototype.shift.call(arguments),
            fns=eventCache[key];
        if(!fns||fns.length===0){
            return false;
        }
        for(var i=0,fn;fn=fns[i++];){
            fn.apply(this,arguments);
        }
    }
    remove=function(key,fn){
        var fns=eventCache[key];

        if(!fns){
            return false;
        }
        if(!fn){
            fns&&(fns.length=0);
        }else{
            for(var l=fns[length]-1;l>=0;l--){
                var _fn=fns[l];
                if(_fn===fn){
                    fns.splice(l,1);
                }
            }
        }
    }
    
    return {
        listen:listen,
        trigger:trigger,
        remove:remove
    }

})()

```

## 模块间通信

比如模块a中有个按钮，模块b中有个div，点击a中的按钮，模块b中div显示就自动+1,利用全局发布订阅模式对象

```javascript
//a模块
var a=(function(window,document,undefined){
    var count=0;
    var btn=document.getElementById('count')
    btn.onclick=function(){
        PubSub.trigger('add',count++);
    }
})(window,document)
//b模块
var b=(function(window,document,undefined){
    var div=document.getElementById('div1');
    PubSub.listen('add',function(data){
        div.innerHTML=data;
    })
})(window,document)
```

## 先发布后订阅

```javascript
/*
发布订阅模式，可现实先发布后订阅
比如商城中用户登录之后获取用户信息，再渲染用户导航，有时会出现用户信息获取完了（开始发布事件），导航模块代码还没加载完（即还没订阅事件）
特别是用了一些模块化惰性加载技术后
*/
var Event2 = (function() {
    var global = this,
        Event2,
        _default = 'default';
    Event2 = (function() {
        var _listen,
            _trigger,
            _remove,
            _slice = Array.prototype.slice,
            _shift = Array.prototype.shift,
            _unshift = Array.prototype.unshift,
            namespaceCache = {},
            _create,
            find,
            each = function(arr, fn) {
                var ret;
                for (var i = 0; i < arr.length; i++) {
                    var n = arr[i];
                    ret = fn.call(n, i, n);
                }
                return ret;
            }

        _listen = function(key, fn, cache) {
            if (!cache[key]) {
                cache[key] = [];
            }
            cache[key].push(fn);
        }

        _remove = function(key, cache, fn) {
            if (cache[key]) {
                if (fn) {
                    for (var i = cache[key].length; i >= 0; i--) {
                        if (cache[key][i] === fn) {
                            cache[key].splice(i, 1)
                        }
                    };
                } else {
                    cache[key] = [];
                }
            }
        }

        _trigger = function() {
            var cache = _shift.call(arguments),
                key = _shift.call(arguments),
                args = arguments,
                _self = this,
                ret,
                stack = cache[key];
            if (!stack || !stack.length) {
                return;
            }
            return each(stack, function() {
                return this.apply(_self, args);
            })
        }

        _create = function(ns) {
            var namespace = ns || _default;
            var cache = {},
                offlineStack = [],
                ret = {
                    listen: function(key, fn, last) {
                        _listen(key, fn, cache);
                        if (offlineStack === null) {
                            return;
                        }
                        if (last === 'last') {
                            offlineStack.length && offlineStack.pop()();
                        } else {
                            each(offlineStack, function() {
                                this();
                            })
                        }
                        offlineStack = null;
                    },
                    one: function(key, fn, last) {
                        _remove(key, cache);
                        this.listen(key, fn, last);
                    },
                    remove: function(key, fn) {
                        _remove(key, cache, fn);
                    },
                    trigger: function() {
                        var fn, args, _self = this;
                        _unshift.call(arguments, cache);
                        args = arguments;
                        fn = function() {
                            return _trigger.apply(_self, args);
                        }
                        if (offlineStack) {
                            return offlineStack.push(fn);
                        }
                        return fn();
                    }
                }
            return namespace ? (namespaceCache[namespace] ? namespaceCache[namespace] : namespaceCache[namespace] = ret) : ret;
        }
        return {
            create: _create,
            one: function(key, fn, last) {
                var event = this.create();
                event.one(key, fn, last);
            },
            remove: function(key, fn) {
                var event = this.create();
                event.remove(key, fn);
            },
            listen: function(key, fn, last) {
                var event = this.create();
                event.listen(key, fn, last);
            },
            trigger: function() {
                var event = this.create();
                event.trigger.apply(this, arguments);
            }
        }
    })();
    return Event2;
})()


//先触发
Event2.create('ns1').trigger('click',1);
//后订阅
setTimeout(function(){
    Event2.create('ns1').listen('click',function(a){
        console.log(a); //output :1
    })
},2000)

Event2.create('ns2').trigger('click',2);

setTimeout(function(){
    Event2.create('ns2').listen('click',function(a){
        console.log(a);//output :2
    })
},3000)

```

## 总结

发布订阅模式有点，一时间上解耦，二对象间解耦。