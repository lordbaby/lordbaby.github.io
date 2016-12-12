title: 享元模式
date: 2016-05-06 11:33:25
tags:
categories: JavaScript
---

享元（flyweight）模式是一种用于性能优化的模式。尤其是在移动端浏览器分配内存不算多的情况下，如何节省内存则是一件很重要的事情。

<!--more-->

# 初识享元模式

比如厂里有50种不同款式的男内衣和50种女内衣，需要每种男女内衣进行试穿拍照，以程序思维正常情况下需要50个男模特和50个女模特（塑料的），不适用享元模式的情况下：

```javascript
var Model=function(sex,underwear){
    this.sex=sex;
    this.underwear=underwear;
}

Model.prototype.takePhoto=function(){
    console.log('sex= '+this.sex+' underwear= '+this.underwear);
}

for(var i=1;i<=50;i++){
    var maleModel=new Model('male','underwear'+i);
    maleModel.takePhoto();
}
for(var j=1;j<=50;j++){
    var femaleModel=new Model('female','underwear'+j);
    femaleModel.takePhoto();
}
```

这样会产生100个对象，如果更多的内衣的话这样对内存需求也会更大。这种方案自然而然的无法解决内存的问题。既然是性能优化则需要尽可能少的创建对象，比如男女模特各一个，分别穿不同的内衣来拍照。再以面向对象的思想看模特，显然`underwear`不属于模特的自有属性，放到外面作为外部属性进行附加。代码如下：

```javascript
var Model=function(sex){
    this.sex=sex;
}

Model.prototype.takePhoto=function(){
    console.log('sex= '+this.sex+' underwear= '+this.underwear);
}
for(var i=1;i<=50;i++){
    var maleModel=new Model('male');
    maleModel.underwear='underwear'+i
    maleModel.takePhoto();
}
for(var j=1;j<=50;j++){
    var femaleModel=new Model('female');
    femaleModel.underwear='underwear'+j
    femaleModel.takePhoto();
}
```

# 内部属性与外部属性

万物皆对象，面向对象编程要求我们编写类的时候，颗粒细度要适应程序的需求，每一个类的属性应是该类的自然属性，而享元模式要求内部属性即某一个对象的自然属性，外部属性独立于具体的场景，根据不同的场景而变化。
这样一来可以根据把所有内部属性相同的对象指定为同一个共享对象，而外部属性从对象身上剥离出来，并存储在外部。

享元模式的问题：

1. 通过构造函数new出了男女两个model对象，在其他系统中，也许并不是一开始就需要所有的共享对象。
2. 给model对象手动设置underwear外部属性，在更新复杂系统中，这不是个最好的办法，因为外部属性可能相当复杂，他们与共享对象的联系会变的困难。

解决方案：

1. 通过工厂函数，只有当某种共享对象被真正需要时，才从工厂中创建出来。
2. 可以通过一个管理器来记录对象相关的外部属性，在使用这些外部属性时通过某个钩子和共享对象联系起来。

# 文件上传例子

文件上传方式有好几种，比如浏览器插件，Flash，表单上传等，总之当选择文件之后，都会调用一个startUpload方法。

```javascript

/*
  上传对象
  它们实际上是根据上传方式的不同而创建对象，则上传方式为内部属性
  无论采取什么上传方式，fileName，fileSize都是根据场景而变化的，被规划为外部属性
*/
var Upload=function(uploadType){
    this.uploadType=uploadType;
}
Upload.prototype.delFile=function(id){
    
    uploadManager.setExteralProperty(id,this);

    if(this.fileSize<3000){
        return this.dom.parentNode.remove(this.dom);
    }
    if(window.confirm('确定要删除该文件吗？'+this.fileName)){
        return this.dom.parentNode.remove(this.dom);
    }
}

//工厂函数
var UploadFacory=(function(){
    var createFlyWeightObj=[];
    return {
        create:function(uploadType){
            if(createFlyWeightObj[uploadType]){
                return createFlyWeightObj[uploadType];
            }
            return createFlyWeightObj[uploadType]=new Upload(uploadType);
        }
    }
})()
//外部属性管理器
var uploadManager=(function(){
    var uploadDatabase={};

    return {
        add:function(id,uploadType,fileName,fileSize){
            var flyWeightObj=UploadFactory.create(uploadType);

            var dom=document.createElement('div');
            dom.innerHTML='<span>文件名称：'+fileName+' ,文件大小：'+fileSize+'</span>'+
                            '<button class="delFile">删除<button>';
            dom.querySelector('.delFile').onclick=function(){
                flyWeightObj.delFile(id);
            }
            document.body.appendChild(dom);

            uploadDatabase[id]={
                fileSize:fileSize,
                fileName:fileName,
                dom:dom
            }

            return flyWeightObj;
        },
        //什么时候需要什么时候添加，比如删除的时候需要查看一下文件大小，则就在删除的时候附加
        setExteralProperty:function(id,flyWeightObj){
            var uploadData=uploadDatabase[id];
            for(var p in uploadData){
                flyWeightObj[p]=uploadData[p]
            }
        }
    }
})()
//starUpload
var id=0;
var startUpload=function(uploadType,files){
    for(var i=0,file;file=files[i++];){
        var uploadObj=uploadManager.add(++id,uploadType,file.fileName,file.fileSize);
    }
}

```

# 享元模式的使用场景

作为一个性能优化的解决方案，但也带来了Factory对象和manager对象的开销及维护。

一般以下情况发生时可以使用享元模式

* 程序中使用了大量的相似对象，造成很大的内存开销
* 对象的大多数属性可以变为外部属性
* 剥离出对象的外部属性之后，可以用相对较少的共享对象取代大量对象

# 对象池

对象池维护一个装载空闲对象的池子，需要对象时不是直接new，而是转从对象池中获取，如果对象池中没有空闲则在创建一个对象，当完成其职责后再进入池子等待被下次获取。

```javascript
//对象池工厂
var objectPoolFactory=function(createObjFn){
    var objectPool=[];

    return {
        //获取
        get:function(){
            var obj=objectPool.length===0?createObjFn.apply(this,arguments):objectPool.shift();
            return obj;
        },
        //回收
        recover:function(obj){
            objectPool.push(obj)
        }
    }
}
```

比如在地图搜索时，地图上会出现一些标志地名的红色气泡，我在我家附近搜索招商银行ATM出现了2个气泡，然后再搜索69路公交站出现了4个气泡，按照对象池的思想，在搜索69路公交站之前，并不会把第一次创建的2个气泡删除掉，而是放到对象池中，在第二次搜索结果则只需要创建2个气泡即可。

```javascript
var createTooltip=function(obj){
    var tooltip=document.createElement('div');
    tooltip.style.width='20px';
    tooltip.style.height='20px;'
    tooltip.style.backgroundColor='red';
     tooltip.style.color='#fff';
    document.body.appendChild(tooltip);
    return tooltip;
}
toolTipFactory=objectPoolFactory(createTooltip);

var search=function(msg){
    var ary=[];
    if(msg==='招商银行ATM'){
        for(var i=0,str;str=['ATM1','ATM2'][i++];){
            var tooltip=toolTipFactory.get();
            tooltip.innerHTML=str;
            ary.push(tooltip);
        }
    }
    if(msg==='69路公交站'){
        for(var i=0;i<ary.length;i++){
            toolTipFactory.recover(ary[i]);
        }
        for(var i=0,str;str=['A','B','C','D'][i++];){
            var tooltip=toolTipFactory.get();
            tooltip.innerHTML=str;
            ary.push(tooltip);
        }
    }
}

search('招商银行ATM')
search('69路公交站')
```

# 总结

享元模式是为解决性能而生的模式。