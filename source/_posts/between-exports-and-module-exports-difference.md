---
title: exports和module.exports的区别
date: 2015-6-24 13:39:43
tags: [Node.js]
categories: JavaScript
---

require是用来加载代码，它们两个则是用来导出代码，一开始学习node.js时会迷惑它们区别是什么。
<!--more-->

# js基础

先来看一段代码

```javascript
var a={name:1};
var b=a;

console.log(a);
console.log(b);

b.name=2;

console.log(a);
console.log(b);

b={name:3};

console.log(a);
console.log(b);

//运行结果
/*
{name:1}
{name:1}
{name:2}
{name:2}
{name:2}
{name:3}
 */

```

a是一个对象，b是对a的引用，即ab都指向同一内存地址。当修改b时，ab指向同一内存地址的内容发生变化，所以a也体现出来了。当b被赋值另一个对象时 会在内存中重新分配地址，b会指向新的内存地址，而a则不会变。

# 区别

明白上面的例子，只需要知道3点就知道exports和module.exports的区别了：

1. module.exports的初始值为一个空对象。
2. exports是指向module.exports的引用。
3. require()返回的是module.exports而不是exports。

Node.js官网文档

```javascript
//the exports variable that is available within a module starts as a reference to module.exports. As with any variable, //if you assign a new value to it,it is no longer bound to the previous value

function require(...){
    ((module,exports) =>{
        exports=some_func;

        module.exports=some_func;
    })(module,module.exports);
    return module;
}

//as a guideline ,if the relationship between exoirts and moudule.exports semms like magic to you , ignore exports and only use module.exports
```

经常看见这样的写法：

```exports=module.exports={....}```

等价于

```
module.exports={....}

exports=module.exports
```

原理很简单：module.exports指向新对象时，exports就断开了与module.exports的引用，那么通过exports=module.exports让exports重新指向了module.exports