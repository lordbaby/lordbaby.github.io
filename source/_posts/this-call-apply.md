title: this，call和apply
date: 2016-04-07 10:41:25
tags: [this,call,apply]
categories: JavaScript
---

## this

javascript中的this始终指向一个对象，具体指向哪个是在运行时基于函数的执行环境动态绑定的。

<!--more-->

## this的指向

* 作为对象的方法调用
* 作为普通函数调用
* 构造器调用
* Function.prototype.call或Function.prototype.apply调用

### 1.做为对象的方法调用

当函数作为对象的方法调用，this指向该对象

```javascript
var obj={
    a:1,
    getA:function(){
        console.log(this===obj); //输出:true 
        console.log(this.a);     //输出:1
    }
}
obj.getA();

```

### 2.作为普通函数调用

当普通函数调用，this指向的是全局对象，在浏览器中则是window对象。

```javascript
window.name='globalName';

var getName=function(){
    return this.name;
}
console.log(getName()); //=>globalName 
console.log(window.getName()); //getName方法中的this指向调用getName方法的对象（window）

//或者

var myObj={
    a:'myObj',
    getName:function(){
        return this.name;
    }
}
var getName=myObj.getName;
console.log(getName()); //=>globalName 
```

不过在es5的strict模式下 this不会指向全局对象，而是undefined。

```javascript
var func=function(){
    'use strict'
    console.log(this);
}
func(); //=>undefined
```

### 3.构造器调用

当用new调用构造函数时，该函数总返回一个对象，构造函数里面的this就指向返回的这个对象。

```javascript
var MyClass=function(){
    this.name='MyClass';
};
var myObj=new MyClass();
console.log(myObj.name); //=> MyClass;

//如果构造函数中显示的返回了一个对象，那么new之后返回的对象为return中的，而不是之前期待的那个this

var MyClass2=function(){
    this.name='myClass';
    return {
        name:'MyClass2'
    }
}
var myObj2=new MyClass2();
console.log(myObj2.name);//=>MyClass2

//若构造器不显示的返回任何数据，或者返回一个非对象类型的数据（即5大基本类型），就不会造成上述问题。
```

### 4.Function.prototype.call或Function.prototype.apply调用

这两个方法可以动态的改变传入函数的this

```javascript
var obj={
    name:'obj',
    getName:function(){
        return this.name;
    }
}
var obj2={
    name:'obj2'
}
console.log(obj.getName()) //=>obj
console.log(obj.getName.call(obj2)) //=>obj2
```

call和apply能够很好的体现js函数式语言特性。

### 丢失this

```javascript
var getId=function(id){
    return document.getElementById(id);
}
getId('div1');

//错误的方式
var getId=document.getElementById;
getId('div1');
//在浏览器中会抛出异常错误，因为document.getElementById方法里面用到this，this指向的是document，用getId来引用document.getElementById，再调用时this指向可就是window了。

document.getElementById=(function(func){
    return function(){
        return func.apply(document,arguments);
    }
})(document.getElementById)
var getId=document.getElementById;
getId('div1');//正常
```

### call和apply的区别

功能完全一样，区别在于传入参数的形式不同。

* 第一个参数指定了函数体内this对象的指向。
* 第二个参数为一个带下表的集合，这个集合可以是数组，也可以是伪数组。call传入的参数数量不固定。

当调用一个函数时，js解释器不会计较形参与实参的数量，类型以及顺序上的区别，js的参数在内部就是用一个数组来表示，从这个意义上说，apply比call的使用率更高。

当使用call或apply的时候，如果传入的第一个参数为null，函数体内的this会指向默认的宿主对象。
有时使用call或apply的目的不在于指定this的指向，而是另有用途，比如借用其他对象的方法，那么可以传入null来代替某个具体的对象。

```javascript
Math.max.apply(null,[1,23,4,5,6]); //=>23
```

### call和apply的用途

1. 改变this的指向

```javascript
window.name='windowName'
var obj1={
    name:'obj1'
}
var obj2={
    name:'obj2'
}
var getName=function(){
    return this.name;
}
console.log(getName()) //=>windowName
console.log(getName.call(obj1)) //=>obj1
console.log(getName.call(obj2)) //=>obj2
```

在实际开发中经常会遇到this指向被不经意改变的场景

```javascript
document.getElementById('div1').onclick=function(){
    console.log(this.id) //=>div1
    var callback=function(){
        console.log(this) 
    }
    callback();//callback直接调用则输出window对象
    callback.call(this) //this为div1元素
}
```

2. Function.prototype.bind

Function.prototype.bind用来指定函数内部的this指向，大部分浏览器都实现了内置的bind。

```javascript
//这是简化版的
Function.prototype.bind=function(context){
    var self=this;  //保存原函数
    return function(){  //返回一个新函数
        return self.apply(context,arguments); //返回原函数的执行结果
    }
}
var obj={
    name:'obj'
}
var func=function(){
    console.log(this.name);
}.bind(obj);

console.log(func()) //=>obj

//复杂点的

Function.prototype.bind=function(){
    var self=this,
        context=[].shift.call(arguments), //取context ,shift取数组头部，原数组会减少
        args=[].slice.call(arguments); //取除context剩余参数
    return function(){
        return self.apply(context,[].concat.call(args,[].slice.call(arguments)));
    }
}

var obj={
    name:'obj'
}
var func=function(a,b,c,d){
    console.log(this.name);
    console.log([a,b,c,d]);
}.bind(obj,1,2);

func(3,4); //=> obj , [1,2,3,4]

```


3. 借用其他对象的方法

第一种场景就是借用构造函数，可以实现继承效果
```javascript
var A=function(name){
    this.name=name;
}
var B=function(){
    A.apply(this,arguments)
}
B.prototype.getName=function(){
    return this.name;
}
var b=new B('name1');
console.log(b.getName()) //=>name1

```

给伪数组添加新元素

```javascript
(function(){
    
    Array.prototype.push.call(arguments,3);
    console.log(arguments) //=>[1,2,3]

})(1,2)
```

将伪数组变为真正的数组，可以借用Array.prototype.slice方法，想截去arguments列表的头一个元素，可借用Array.prototype.shift方法

以Array.prototype.push为例，看看V8引擎的具体实现：

```javascript
function ArrayPush(){
    var n=TO_UINT32(this.length); //被push对象的length
    var m=%_ArgumentsLength(); //push的参数个数
    for(var i=0;i<m;i++){
        this[i+n]=%_Arguments(i);  //复制元素
    }
    this.length=n+m;
    return this.length;
}
```

以Array.prototype.push实际上就是一个属性的复制过程，顺便修改这个对象的length属性，至于被修改的对象是谁，到底是数组还是类数组对象，这一点并不重要。

```javascript
var o={
    length:0
}
Array.prototype.push.call(o,'first');
console.log(o); //=>Object {0: "first", length: 1}

```

能借用Array.prototype.push方法的对象要满足2个条件

1. 对象本身要可以存取属性
2. 对象的length属性可读写