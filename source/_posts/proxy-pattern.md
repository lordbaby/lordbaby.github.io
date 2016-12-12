title: 代理模式
date: 2016-04-15 12:01:46
tags: 
categories: JavaScript
---

代理模式是为一个对象提供一个代用品或占位符，以便控制对它的访问。

<!--more-->

## 虚拟代理之合并http请求

场景1：页面中有一文件列表，勾中列表上的checkbox即同步文件,但手速快的小伙伴一秒钟能点4个checkbox，可见频繁的网络请求会带来相当大的开销。

解决方案：通过一个代理函数proxy，来收集一段时间内的请求，然后一次性发送给服务器，比如等待2秒之后才把这2秒之内的全部文件id打包发送给服务器。大大减少服务器压力。
```javascript
var synchronFile=function(){
    console.log('开始同步文件，id为：'+id);
}
var proxy=(function(){
    var cache=[], //存一段时间内需要同步的id
        timer;    //定时器
    return function(id){
        cache.push(id);
        if(timer) return; //保证不会覆盖已启动的定时器
        timer=setTimeout(function(){
            synchronFile(cache.join(',')); //2秒后向服务器发送需要同步的定时器
            clearTimeout(timer)   //清空定时器
            timer=null;
            cache.length=0;   //清空id集合
        },2000)
    }
})()

var checkboxs=document.getElementByTagName('input');

for(var i=0,c;c=checkboxs[i++];){
    c.onclick=function(){
        if(this.checked===true){
            proxy(this.id);
        }
    }
}
```

## 虚拟代理之惰性加载js文件

场景2：有个js文件adConsole.js大概有1.5kb，也许一开始并不需要加载这么大的js文件，因为并不是每个用户都需要打印log，我们希望在有必要时才加载它，比如用户主动呼唤出浏览器控制台时，在打印出招聘广告，比如百度。

```javascript
// adConsole.js

var adConsole={
    log:function(ad){
        console.log(ad);
    }
} 

//通过按下F12在加载adConsole.js 执行已缓存的函数

var proxy=(function(){
    var cache=[];  //缓存将要被执行的函数
    var handler=function(ev){
        if(ev.keyCode===123){ //f12
            var script=document.createElement('script');
            script.onload=function(){
                for(var i=0,fn;fn=cache[i++];){
                    fn();
                }
            }
            script.src="adConsole.js";
            document.getElementsByTagName('head')[0].appendChild(script);
            //移除body的keydown监听事件，保证只加载一次
            document.body.removeEventListener('keydown',handler);
        }
    }
    //给页面body添加keydown监听事件
    document.body.addEventListener('keydown',handler,false);

    return {
        log:function(){
            var args=arguments;
            cache.push(function(){
                return adConsole.log.apply(adConsole,args)
            });
        }
    }
})()

proxy.log('一张网页，要经历怎样的过程，才能抵达用户面前？')
proxy.log('一位新人，要经历怎样的成长，才能站在技术之巅？');
proxy.log('探寻这里的秘密；')
proxy.log('体验这里的挑战；\n成为这里的主人；');

```

##  缓存代理

场景3：缓存计算结果，把开销大的一些运算结果进行缓存，在下次运算时，如果传入的参数在缓存中有计算结果则直接返回，比如计算乘积的例子

```javascript
//通常算法
var mult=function(){
    var a=1;
    for(var i=0,len=arguments.length;i<len;i++){
        a*=arguments[i];
    }
    return a;
}
mult(1,2,3,4); //24

//缓存代理算法
var proxyMult=(function(){
    var cache={};
    return function(){
        var args=Array.prototype.join.call(arguments,',');
        console.log(cache[args])
        if(cache[args]){
            return cache[args];
        }

        return cache[args]=mult.apply(this,arguments);
    }
})()

proxyMult(1,2,3,4,5)
proxyMult(1,2,3,4,5)

```

## 总结

代理模式包括许多小分类，在js中最常用的是虚拟代理和缓存代理，虽然代理模式非常有用，但编写业务代码时，往往不需要预先猜是否需要代理模式，当真正发现不方便访问某个对象的时候，在写代理模式也不迟。