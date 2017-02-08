---
title: JavaScript数组去重
date: 2017-02-08 18:00:08
tags: [Array,unique]
categories: JavaScript
---

数据去重的方法非常之多，以前我的办法就是利用把数组的元素当作对象的key来去重，发现其实这并不是个好办法。比如无法区分隐式转化成字符串后一样的值，比如1和'1',特殊的数据'__proto__'也不行，因为Object的"__proto__"无法被重写等等。但话又说回来，如果你单纯的一个数值型数组，这个方法也是可以的。所以说具体的业务场景适当选择不同的技术，没有什么一劳永逸的方法。

<!--more-->

# 定义重复

要去重，首先得定义什么是重复的，具体到代码中就是数组中两个元素在什么情况下算是相等。对于原始值而言，我们很容易想到1和1是相等的，'1'和'1'也是相等的。那么，1和'1'是相等的么？

如果这个问题还好说，只要回答“是”或者“不是”即可。那么下面这些情况就没那么容易了。

# NaN

初看NaN时，很容易把它当成和null、undefined一样的独立数据类型。但其实，它是数字类型

```javascript
console.log(typeof NaN); //number

```
根据规范，比较运算中只要有一个值为NaN，则比较结果为false，所以会有下面这些看起来略蛋疼的结论：
```javascript
// 全都是false
0 < NaN;
0 > NaN;
0 == NaN;
0 === NaN;
```
这意味着任何涉及到NaN的情况都不能简单地使用比较运算来判定是否相等。比较科学的方法只能是使用isNaN()：
```javascript
var a = NaN;
var b = NaN;
// true
console.log(isNaN(a) && isNaN(b));
```

# 原始值和包装对象

如果你研究过`'a'.trim()`这样的代码的话，不知道是否产生过这样的疑问：`'a'`明明是一个原始值（字符串），它为什么可以直接调用`.trim()`方法呢？当然，很可能你已经知道答案：因为JS在执行这样的代码的时候会对原始值做一次包装，让'a'变成一个字符串对象，然后执行这个对象的方法，执行完之后再把这个包装对象脱掉。可以用下面的代码来理解:
```javascript
// 'a'.trim();
var tmp = new String('a');
tmp.trim();
```
这段代码只是辅助我们理解的。但包装对象这个概念在JS中却是真实存在的。
```javascript
var a = new String('a');
var b = 'b';
```
a即是一个包装对象，它和b一样，代表一个字符串。它们都可以使用字符串的各种方法（比如trim()），也可以参与字符串运算（+号连接等）。

但他们有一个关键的区别：类型不同
```javascript
typeof a; // object
typeof b; // string
```
在做字符串比较的时候，类型的不同会导致结果有一些出乎意料：
```javascript
var a1 = 'a';
var a2 = new String('a');
var a3 = new String('a');
a1 == a2; // true
a1 == a3; // true
a2 == a3; // false
a1 === a2; // false
a1 === a3; // false
a2 === a3; // false
```
同样是表示字符串a的变量，在使用严格比较时竟然不是相等的，在直觉上这是一件比较难接受的事情，在各种开发场景下，也非常容易忽略这些细节

# ==和===

在一些文章中，看到某一些数组去重的方法，在判断元素是否相等时，使用的是==比较运算符。众所周知，这个运算符在比较前会先查看元素类型，当类型不一致时会做隐式类型转换。这其实是一种非常不严谨的做法。因为无法区分在做隐匿类型转换后值一样的元素，例如`0、''、false、null、undefined`等。

同时，还有可能出现一些只能黑人问号的结果，例如：

```javascript
[] == ![]; //true
```

# Array.prototype.indexOf()

在一些版本中，去重用到了该方法：

```javascript
function unique(arr){
    return arr.filter(function(item,idx){
        // indexOf返回第一个索引值，
        // 如果当前索引不是第一个索引，说明是重复值
        return arr.indexOf(item)===idx;
    })
}

function unique(arr){
    var ret=[];
    arr.forEach(function(item){
        if(ret.indexOf(item)===-1){
            ret.push(item)
        }
    });
}

```

既然==和===在元素相等的比较中是有巨大差别的，那么indexOf的情况又如何呢？indexOf()使用的是严格比较，也就是===

>>再次强调：按照前文所述，===不能处理NaN的相等性判断。

# Array.prototype.includes()

`Array.prototype.includes()`是ES2016中新增的方法，用于判断数组中是否包含某个元素，所以上面使用indexOf()方法的第二个版本可以改写成如下版本：

```javascript
function unique(arr){
    var ret=[];
    arr.forEach(function(item){
        if(!ret.includes(item)){
            ret.push(item)
        }
    });
}
```
那么，猜猜，includes()又是用什么方法来比较的呢？如果想当然的话，会觉得肯定跟indexOf()一样喽。但是，程序员的世界里最怕想当然。翻一翻规范，发现它其实是使用的另一种比较方法，叫作“SameValueZero”比较（https://tc39.github.io/ecma262/2016/#sec-samevaluezero）。

>>If Type(x) is different from Type(y), return false.
If Type(x) is Number, then
a. If x is NaN and y is NaN, return true.
b. If x is +0 and y is -0, return true.
c. If x is -0 and y is +0, return true.
d. If x is the same Number value as y, return true.
e. Return false.
Return SameValueNonNumber(x, y).

注意2.a，如果x和y都是NaN，则返回true！也就是includes()是可以正确判断是否包含了NaN的。我们写一段代码验证一下：

```javascript
var arr = [1, 2, NaN];
arr.indexOf(NaN); // -1
arr.includes(NaN); // true
```
可以看到indexOf()和includes()对待NaN的行为是完全不一样的。

# 其他方案

双重遍历

```javascript
function unique(arr) {
    var ret = [];
    var len = arr.length;
    var isRepeat;
    for(var i=0; i<len; i++) {
        isRepeat = false;
        for(var j=i+1; j<len; j++) {
            if(arr[i] === arr[j]){
                isRepeat = true;
                break;
            }
        }
        if(!isRepeat){
            ret.push(arr[i]);
        }
    }
    return ret;
}

//优化版本
//
function unique(arr) {
    var ret = [];
    var len = arr.length;
    for(var i=0; i<len; i++){
        for(var j=i+1; j<len; j++){
            if(arr[i] === arr[j]){
                j = ++i;
            }
        }
        ret.push(arr[i]);
    }
    return ret;
}
```
这种方案没什么大问题，用于去重的比较部分也是自己编写实现（arr[i] === arr[j]），所以相等性可以自己针对上文说到的各种情况加以特殊处理。唯一比较受诟病的是使用了双重循环，时间复杂度比较高，性能一般。

# 使用key来去重

```javascript
function unique(arr) {
    var ret = [];
    var len = arr.length;
    var tmp = {};
    for(var i=0; i<len; i++){
        if(!tmp[arr[i]]){
            tmp[arr[i]] = 1;
            ret.push(arr[i]);
        }
    }
    return ret;
}
```

这种方法是利用了对象（tmp）的key不可以重复的特性来进行去重。但由于对象key只能为字符串，因此这种去重方法有许多局限性

1. 无法区分隐式类型转换成字符串后一样的值，比如1和'1'
2. 无法处理复杂数据类型，比如对象（因为对象作为key会变成[object Object]）
3. 特殊数据，比如'__proto__'会挂掉，因为tmp对象的__proto__属性无法被重写

对于第一点，有人提出可以为对象的key增加一个类型，或者将类型放到对象的value中来解决：

```javascript
function unique(arr) {
    var ret = [];
    var len = arr.length;
    var tmp = {};
    var tmpKey;
    for(var i=0; i<len; i++){
        tmpKey = typeof arr[i] + arr[i];
        if(!tmp[tmpKey]){
            tmp[tmpKey] = 1;
            ret.push(arr[i]);
        }
    }
    return ret;
}
```

而第二个问题，如果像上文所说，在允许对对象进行自定义的比较规则，也可以将对象序列化之后作为key来使用。这里为简单起见，使用JSON.stringify()进行序列化。

```javascript
function unique(arr) {
    var ret = [];
    var len = arr.length;
    var tmp = {};
    var tmpKey;
    for(var i=0; i<len; i++){
        tmpKey = typeof arr[i] + JSON.stringify(arr[i]);
        if(!tmp[tmpKey]){
            tmp[tmpKey] = 1;
            ret.push(arr[i]);
        }
    }
    return ret;
}
```

# Map key

可以看到，使用对象key来处理数组去重的问题，其实是一件比较麻烦的事情，处理不好很容易导致结果不正确。而这些问题的根本原因就是因为key在使用时有限制。

那么，能不能有一种key使用没有限制的对象呢？答案是——真的有！那就是ES2015中的Map。

>>Map是一种新的数据类型，可以把它想象成key类型没有限制的对象。此外，它的存取使用单独的get()、set()接口。

```javascript
var tmp = new Map();
tmp.set(1, 1);
tmp.get(1); // 1
tmp.set('2', 2);
tmp.get('2'); // 2
tmp.set(true, 3);
tmp.get(true); // 3
tmp.set(undefined, 4);
tmp.get(undefined); // 4
tmp.set(NaN, 5);
tmp.get(NaN); // 5
var arr = [], obj = {};
tmp.set(arr, 6);
tmp.get(arr); // 6
tmp.set(obj, 7);
tmp.get(obj); // 7
```

由于Map使用单独的接口来存取数据，所以不用担心key会和内置属性重名（如上文提到的__proto__）。使用Map改写一下我们的去重方法：

```javascript
function unique(arr) {
    var ret = [];
    var len = arr.length;
    var tmp = new Map();
    for(var i=0; i<len; i++){
        if(!tmp.get(arr[i])){
            tmp.set(arr[i], 1);
            ret.push(arr[i]);
        }
    }
    return ret;
}
```

# Set

既然都用到了ES2015，数组这件事情不能再简单一点么？当然可以。

除了Map以外，ES2015还引入了一种叫作Set的数据类型。顾名思义，Set就是集合的意思，它不允许重复元素出现，这一点和数学中对集合的定义还是比较像的

```javascript
var s = new Set();
s.add(1);
s.add('1');
s.add(null);
s.add(undefined);
s.add(NaN);
s.add(true);
s.add([]);
s.add({});
```

如果你重复添加同一个元素的话，Set中只会存在一个。包括NaN也是这样。于是我们想到，这么好的特性，要是能和数组互相转换，不就可以去重了吗？

```javascript
function unique(arr){
    var set = new Set(arr);
    return Array.from(set);
}
```

讨论了这么久的事情，居然两行代码搞定了，简直不可思议。

然而，不要只顾着高兴了。有一句话是这么说的“不要因为走得太远而忘了为什么出发”。我们为什么要为数组去重呢？因为我们想得到不重复的元素列表。而既然已经有Set了，我们为什么还要舍近求远，使用数组呢？是不是在需要去重的情况下，直接使用Set就解决问题了？这个问题值得思考。

# 小结

最后，用一个测试用例总结一下文中出现的各种去重方法：

```javascript
var arr = [1,1,'1','1',0,0,'0','0',undefined,undefined,null,null,NaN,NaN,{},{},[],[],/a/,/a/]
console.log(unique(arr));
```

测试中没有定义对象的比较方法，因此默认情况下，对象不去重是正确的结果，去重是不正确的结果。

|方法 |结果  |说明|
|------|:------:|-----:|
|indexOf#1|NaN被去掉 | |
|indexOf#2|NaN重复   | |
|includes |正确  | |
|双重循环#1|NaN重复 | |  
|双重循环#2|NaN重复   | |
|对象#1|字符串和数字无法区分，对象、数组、正则表达式被去重   | |
|对象#2|对象、数组、正则表达式被去重  | |
|对象#3|对象、数组被去重，正则表达式被消失   |JSON.stringify(/a/)结果为{}，和空对象一样|
|Map| 正确  　| |
|Set| 正确| |

任何脱离场景谈技术都是妄谈，本文也一样。去重这道题，没有正确答案，请根据场景选择合适的去重方法。