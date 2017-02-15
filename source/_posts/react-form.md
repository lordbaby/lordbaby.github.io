---
title: React中的表单
date: 2017-01-15 10:00:21
tags: [React]
categories: React
---

表单在web中的作用很重要，本文总结于《深入React技术栈》

<!--more-->

## 文本框

单行文本框`input`，多行文本框`textarea`

示例：

```
import React, {Component} from 'react';

class App extends Component{
    constructor(props){
        super(props);

        this.handleInputChange = this.handleInputChange.bind(this);
        this.handleTextareaChange = this.handleTextareaChange.bind(this);

        this.state={
            inputValue:'',
            textAreaValue:''
        }
    }
    
    handleInputChange(e){
        this.setState({
            inputValue:e.target.value
        });
    }

    handleTextareaChange(e){
        this.setState({
            textAreaValue:e.target.value
        });
    }

    render(){
        const {inputValue,textAreaValue}=this.state;
        return (
            <div>
                <p>
                    单行输入框：
                   <input type="text" 
                          value={inputValue}
                          onChange={this.handleInputChange} />
                </p>
                <p>
                    多行输入框：
                    <textarea cols="30" rows="10"
                              value={textAreaValue}
                              onChange={this.handleTextareaChange}></textarea>
                </p>
            </div>
        );
    }

}

```

>注意：textarea的值在html中是通过children来表示的（`<textarea>value</textarea>`）,而在react中则和input标签一样通过value prop来表示其值。

## radio和checkbox

这两种表单的value值一般不会改变，而是通过一个boolean类型的checked prop（属性）来表示是否为选中状态。

### radio示例

```

import React, {Component} from 'react';

class App extends Component{
    constructor(props){
        super(props);

        this.handleChange = this.handleChange.bind(this);

        this.state={
            radioValue:''
        }
    }
    
    handleChange(e){
        this.setState({
            radioValue:e.target.value
        });
    }

    render(){
        const {radioValue}=this.state;
        return (
            <div>
                <p>性别：</p>
                <label><input name="gender" type="radio" value="male" checked={radioValue==='male'} onChange={this.handleChange} />男</label>
                <label><input name="gender" type="radio" value="female" checked={radioValue==='female'} onChange={this.handleChange} />女</label>
            </div>
        );
    }

}

```

### checkbox示例

```
import React, {Component} from 'react';

class App extends Component{
    constructor(props){
        super(props);

        this.handleChange = this.handleChange.bind(this);

        this.state={
            skills:[]
        }
    }
    
    handleChange(e){
       const { checked,value } =e.target;
       let { skills }=this.state;
       if(checked&&skills.indexOf(value)===-1){
        skills.push(value)
       }else{
        skills.filter(skill => skill!==value);
       }

       this.setState({skills});
    }

    render(){
        const { skills }=this.state;
        return (
            <div>
                <p>技能：</p>
                <label><input type="checkbox" 
                              value="JavaScript" 
                              checked={skills.indexOf('JavaScript')!==-1}
                              onChange={this.handleChange} />
                    JavaScript
                </label>
                <label><input type="checkbox" 
                              value="HTML" 
                              checked={skills.indexOf('HTML')!==-1}
                              onChange={this.handleChange} />
                    HTML
                </label>
                <label><input type="checkbox" 
                              value="CSS" 
                              checked={skills.indexOf('CSS')!==-1}
                              onChange={this.handleChange} />
                    CSS
                </label>
                <label><input type="checkbox" 
                              value="React" 
                              checked={skills.indexOf('React')!==-1}
                              onChange={this.handleChange} />
                    React
                </label>
            </div>
        );
    }

}

```

一个简单的复选框好像变得复杂了，确实，React对表单状态进行了控制，相应的多了一些处理onChange的代码。

## Selecte组件

这个存在多选或单选，通过设置标签属性multiple={true}或者multiple={false}来实现多选或单选

```
//单选
import React, {Component} from 'react';

class App extends Component{
    constructor(props){
        super(props);

        this.handleChange = this.handleChange.bind(this);

        this.state={
            area:''
        }
    }
    
    handleChange(e){
       this.setState({area:e.target.value})
    }

    render(){
        const { area }=this.state;
        return (
            <div>
               <select value={area} onChange={this.handleChange} >
                    <option value="bj">北京</option>
                    <option value="sh">上海</option>
                    <option value="gz">广州</option>
               </select>
            </div>
        );
    }

}

//多选
import React, {Component} from 'react';

class App extends Component{
    constructor(props){
        super(props);

        this.handleChange = this.handleChange.bind(this);

        this.state={
            areas:['bj','sh']
        }
    }
    
    handleChange(e){
       const {options} =e.target;
       //注意这里options返回来的是一个对象 并非数组
       const areas=Object.keys(options)
                    .filter( i => options[i].selected ===true)
                    .map(i => options[i].value);
        this.setState({areas});
    }

    render(){
        const { areas }=this.state;
        return (
            <div>
               <select multiple={true} value={areas} onChange={this.handleChange} >
                    <option value="bj">北京</option>
                    <option value="sh">上海</option>
                    <option value="gz">广州</option>
               </select>
            </div>
        );
    }

}

```

## 受控组件

上面的例子中每个表单的状态发生变化时，都会被写入到组件的state中，这种组件在React中被称为受控组件，在受控组件中，组件渲染出的状态与它的value或checked prop相对应，React通过这种方式消除了组件额拒不状态，使得应用的整个状态更加可控，同样官方也推荐使用受控表单组件。

总结受控表单更新state的流程：

1. 可以通过初始state中设置表单的默认值。
2. 每当表单的值发生变化时，调用onChange事件处理器
3. 事件处理器通过合成事件对象e拿到改变后的状态，并更新应用的state
4. setState触发视图的重新渲染，完成表单组件值的更新。

在React中数据时单向的，上面的例子中通过onChange事件处理器setState重新更新表单状态，并完成了数据额双向绑定。

## 非受控组件 

并非React不支持非受控组件，如果一个表单组件没有value checked等props，可以使用defaultValue、defaultChecked来表示组件的默认状态。

```
import React, {Component} from 'react';

class App extends Component{
    constructor(props){
        super(props);

        this.handleSubmit = this.handleSubmit.bind(this);
    }
    
    handleSubmit(e){
       e.preventDefault();
       const {value} =this.refs.name;//也可以用document.querySelector
       console.log(value)
    }

    render(){
        const { areas }=this.state;
        return (
            <form onSubmit={this.handleSubmit}>
                <input type="text" ref="name" defaultValue="admin" />
                <button type="submit">提交</button>
            </form>
        );
    }

}

```

非受控组件是一种反模式，它的值不受组件自身的state或props控制。上面的defaultValue和defaultChecked只是默认值，要想改变只能通过操作DOM。

## 小结

1. input(type=text)、textarea、select等表单组件通过value属性来展示应用的状态。
2. radio、checkbox通过checked属性来展示状态
3. selected属性可作用域select组件下面的option上，不过React并不推荐，推荐使用value的方式。
4. 以上几个组件状态发生改变时，通过触发onChange事件属性。
5. 表单域没有必要为所有组件的onChange事件属性指定多个处理器。可以集中处理，代码示例：

```
import React, {Component} from 'react';

class App extends Component{
    constructor(props){
        super(props);

        this.handleChange = this.handleChange.bind(this);

        this.state={
            name:'',
            age:20
        }
    }
    
    handleChange(name,e){
       const { value } = e.target;
       this.setState({
        [name]:value
        })
    }

    render(){
        const { name,age }=this.state;
        return (
            <div>
                <input type="text" value={name} onChange={this.handleChange('name')} />
                <input type="text" value={age} onChange={this.handleChange('age')} />
            </div>
        );
    }

}

```
