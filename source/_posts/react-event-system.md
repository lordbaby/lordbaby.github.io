---
title: React事件系统
date: 2016-12-30 14:44:58
tags: [React]
categories: React
---
本文总结于《深入React技术栈》一书。

<!--more-->

## 绑定方式

React事件的绑定方式与原生的HTML事件监听器很相似

React事件属性名是驼峰式写法，事件处理则通过函数指针来绑定

```html
<button onClick={this.handleClick}>Test</button>
```

DOM0级事件，事件处理是js函数

```html
<button onclick="handleClick()">Test</button>
```

## 实现机制

### 事件委派

首先还是理解一下传统的事件代理机制。其原理也就是事件的冒泡机制，冒泡顾名思义是从水底下往上冒泡，所以事件的冒泡是从DOM树底部往顶部传递，只要顶部的DOM有对该事件类型的监听都能拦截到。

React也许根据这种冒泡机制，来实现自己的事件委派。


> React把所有的事件绑定到结构的最外层，使用一个统一的事件监听器，这个事件监听器上维持一个映射来保存所有组件内部的事件监听和处理函数。当组件挂载和卸载时，只是在这个统一的事件监听器上来插入或删除一些对象。当事件发生时，首先被这个统一的事件监听器处理，然后在映射里找到真正的事件处理函数并调用。这样简化了事件处理和回收机制，效率也有很大提升。

### 自动绑定
<<<<<<< Updated upstream

=======
>>>>>>> Stashed changes
在编写React组件时，每个方法的上下文都指向该组件的实例。即自动绑定this为当前组件，但使用es6写组件时需要手动绑定this。

#### 1 将函数在constructor中bind

```javascript
constructor(props) {
    super(props);
    this.state = {
        userInput: ''
    }
    this.handleChange = this.handleChange
    this.clearAndFocusInput = this.clearAndFocusInput.bind(this)
}
```

#### 2 在stage-0也可以通过箭头函数定义函数

```javascript
 handleChange = () => {
 }
```

#### 3 在传递props的时候bind

```html
<input onChange={this.handleChange.bind(this)}/>
or 双冒号语法
<input onChange={::this.handleChange}/>
```
<<<<<<< Updated upstream

### 在React中使用原生事件

=======
### 在React中使用原生事件
>>>>>>> Stashed changes
既然使用原生事件就必须在真是的DOM中绑定，React的生命周期函数componentDidMount会在组件安装并在浏览器中存在真实DOM后调用，此时可以完成对原生事件的绑定。

```javascript
import React ,{Component} from 'react';

class NativeEventDemo extends Component{
    componentDidMount(){
        this.refs.button.addEventListener('click',(e)=>{
            handleClick(e);
        });
    }
    componentWillUnmount(){
        this.refs.button.removeEventListener('click');
    }
    handleClick(e){
        console.log(e)
    }
    render(){
        return (
            <button ref="button">Test</button>
        ) 
    }
}

```

在React中使用原生事件，必须在组件卸载时手动移除，否则会发生内存泄漏的问题，合成事件则不需要。
<<<<<<< Updated upstream

### 原生事件与合成事件混用

=======
### 原生事件与合成事件混用
>>>>>>> Stashed changes
栗子：页面中有个按钮，点击按钮来切换二维码的显示与隐藏，点击按钮之外的区域同样可以隐藏二维码。第一步实现很简单，但是点击页面其他区域隐藏二维码，其他区域已经超出组件的范围，所以只能通过原生事件监听body的click事件。代码如下：

```javascript
import React ,{Component} from 'react';
import style from './qrcode.css';
export default class qrCode extends Component{
    constructor(props){
        super(props);
        this.state={
            active:false
        };
        this.handleClick=this.handleClick.bind(this);
        this.handleQrClick=this.handleQrClick.bind(this);
    }
    componentDidMount(){
        document.body.addEventListener('click',(e)=>{
            this.setState({
                active:false
            });
        });
    }
    componentWillUnmount(){
        document.body.removeEventListener('click');
    }
    handleClick(e){
        e.preventDefault();
        let active=!this.state.active;
        this.setState({
            active:active
        });  
    }
    handleQrClick(e){
        e.preventDefault();
    }
    render(){
        return (
            <div className="qr-wrap">
                <button className="qr" onClick={this.handleClick}>二维码</button>
                <div className="code" 
                     style={{display:this.state.active?'block':'none'}}
                     onClick={this.handleQrClick}>
                    <img src="img/webpack.png" alt="qr"/>
                </div>
            </div>
        )
    }
}
```

逻辑似乎很简单，但结果似乎并不如人意，问题如下：

1. 二维码显示/隐藏，显示后再次点击无法隐藏。
2. 点击二维码区域二维码依然隐藏

第一个问题，React的合成事件冒泡行为只能阻止合成事件。原生事件无法阻止，再就是原生事件监听优先级比合成的高，所以才导致显示后无法隐藏。原书作者并没有提及该问题。

第二个问题，React合成事件系统的委托机制，在合成事件内部仅仅对最外层的容器进行了绑定，并依赖事件的冒泡机制完成了委派，也就是说事件并没有直接绑定到div.qr元素上，所以e.preventDefault()并没什么卵用。

```javascript
import React ,{Component} from 'react';
import style from './qrcode.css';

export default class qrCode extends Component{
    constructor(props){
        super(props);
        this.state={
            active:false
        };
        this.handleClick=this.handleClick.bind(this);
    }
    componentDidMount(){
        document.body.addEventListener('click',(e)=>{
            console.log(e.target);
            if((e.target&&e.target.matches('button.qr'))||(e.target&&(e.target.matches('div.code')||e.target.matches('img')))){
                return;
            }
            this.setState({
                active:false
            });
        });
    }
    componentWillUnmount(){
        document.body.removeEventListener('click');
    }
    handleClick(e){
        e.preventDefault();
        let active=!this.state.active;
        console.log(active);
        this.setState({
            active:active
        });
    }
    render(){
        return (
            <div className="qr-wrap">
                <button className="qr" onClick={this.handleClick}>二维码</button>
                <div className="code" 
                     style={{display:this.state.active?'block':'none'}}>
                    <img src="img/webpack.png" alt="qr"/>
                </div>
            </div>
        )
    }
}
```

### React合成事件VS原生DOM事件

#### 事件传播与阻止事件传播

原生DOM事件的传播分3个阶段：

>事件捕获阶段 => 目标对象事件处理阶段 => 事件冒泡

事件捕获在低于IE9的浏览器无法使用。意义并不大，React事件系统并没有实现它。

阻止原生事件传播需要使用e.preventDefault() 或者e.cancelBubble=true来阻止。在React中只需要e.preventDefault()即可。

#### 事件类型

React合成事件类型是JavaScript原生事件类型的一个子集。

#### 事件绑定方式

原生的有DOM0，DOM2级事件。

React合成事件则简单的多了。

#### 事件对象

原生的有兼容性问题，而在React合成事件中，则不存在。

## 总结

所以尽量避免在React中混用合成事件和原生DOM事件，React事件的冒泡行为只能用于React合成事件系统，它不能阻止原生事件的冒泡，反之，原生事件中阻止冒泡行为，却可以阻止React合成事件的传播。另一个就是原生DOM事件的优先级高于React合成事件。
