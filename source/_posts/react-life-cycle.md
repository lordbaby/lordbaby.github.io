---
title: React的生命周期
date: 2017-02-13 15:56:21
tags: [React]
categories: React
---

React的生命周期可分为挂载、渲染和卸载几个阶段，当然组件更新时会重新渲染组件。本文总结于《深入React技术栈》

<!--more-->

# 初探React生命周期

在自定义组件时，需要在组件生命周期的不同阶段实现不同的逻辑，（为了方便查看组件的生命周期顺序，推荐使用react-lifecycle-mixin）

下面是一张流程图，画图工具：[ASCIIFlow](http://asciiflow.com/)，纯文本，Geek风格，很不幸它不支持中文。

```

+-----------+                                +-----------------------+
|   start   +-----+ReactDOM.render()+------->+       constructor     |
+-----------+                                +-----------+-----------+
                                                         |
                                                         v
                                             +-----------+-----------+                  +----------------------------+
                                             |   componentWillMount  | <----------------+ can use this.setState()    |
+------------------------------+             +-----------+-----------+                  +----------------------------+
|    componentWillUnmount      |                         |                                                           |
+----+-------------------------+                         v                                                           |
     ^                                       +-----------+-----------+                                               |
     |                                       |         render        |                                               |
     |                                       +-----------+-----------+                                               |
     |                                                   |                                                           |
     |                                                   v                                                           |
     |                                       +-----------+-----------+                                               |
     |                                       |   componentDidMount   | <---------------------------------------------+
     |                                       +-----------+-----------+                                               |
     |                                                   |                                                           |
     |                                       +-----------v-----+  props change   +-----------------------------+     |
     +------ReactDOM.------------------------+     Running     +---------------> | componentWillReceiveProps   | <---+
            unmountComponentAtNode()         |                 |                 +---------------+-------------+     |
                                             +-+----+-+--+-----+                                 v                   |
                                               ^    | ^  |     setState()        +---------------+-------------+     |
                                               |    | |  +---------------------> |    shouldComponentUpdate    |     |
                                               |    | |                          +-----------------------------+     |
                                               |    | +-------------- false ---------------------+                   |
                                               |    |                                           true                 |
                                               |    |                                            +                   |
                                               |    |                            +---------------v-------------+     |
                                               |    +-------forceUpdate()------> |    componentWillUpdate      |     |
                                               |                                 +---------------+-------------+     |
                                               |                                                 |                   |
                                               |                                 +---------------v-------------+     |
                                               |                                 |            render           |     |
                                               |                                 +---------------+-------------+     |
                                               |                                                 |                   |
                                               |                                 +---------------v-------------+     |
                                               +---------------------------------+      componentDidUpdate     | <---+
                                                                                 +-----------------------------+


```

1. **首次挂载组件**，执行顺序`getDefaultProps` => `getInitialState` => `componentWillMount` => `render` => `componentDidMount`
2. **卸载组件**，执行`componentWillUnmount`
2. **重新挂载组件**，执行 `getInitialState` => `componentWillMount` => `render` => `componentDidMount`，并没有执行`getDefaultProps`
3. **更新组件**，执行顺序 `componentReceiveProps` => `shouldComponentUpdate` => `componentWillUpdate` => `render` => `componentDidUpdate`.

ES6 class构建组件时，`static defaultProps={}`其实就是调用的内部方法`getDefaultProps`，`contructor`中的`this.state={}`调用的是`getInitialState()`。

生命周期执行顺序图

```
+-------------------------------+   +-----------------------------------+  +--------------------------------+
|        First Render           |   |            Props Change           |  |          State Change          |
+-------------------------------+   +-----------------------------------+  +--------------------------------+
|                               |   |                                   |  |                                |
|  +-------------------------+  |   |   +---------------------------+   |  |   +------------------------+   |
|  |   getDefaultProps       |  |   |   | componentWillReceiveProps |   |  |   |    shouldComponent     |   |
|  +-------------------------+  |   |   +---------------------------+   |  |   +------------------------+   |
|                               |   |                                   |  |                                |
|  +-------------------------+  |   |   +---------------------------+   |  |   +------------------------+   |
|  |   getInitialState       |  |   |   |   shouldComponentUpdate   |   |  |   |   componentWillUpdate  |   |
|  +-------------------------+  |   |   +---------------------------+   |  |   +------------------------+   |
|                               |   |                                   |  |                                |
|  +-------------------------+  |   |   +---------------------------+   |  |   +------------------------+   |
|  |   componentWillMount    |  |   |   |    componentWillUpdate    |   |  |   |         render         |   |
|  +-------------------------+  |   |   +---------------------------+   |  |   +------------------------+   |
|                               |   |                                   |  |                                |
|  +-------------------------+  |   |   +---------------------------+   |  |   +------------------------+   |
|  |         render          |  |   |   |          render           |   |  |   |   componentDidUpdate   |   |
|  +-------------------------+  |   |   +---------------------------+   |  |   +------------------------+   |
|                               |   |                                   |  |                                |
|  +-------------------------+  |   |   +---------------------------+   |  |                                |
|  |    componentDidMout     |  |   |   |    componentDidUpdate     |   |  |                                |
|  +-------------------------+  |   |   +---------------------------+   |  |                                |
|                               |   |                                   |  |                                |
+-------------------------------+   +-----------------------------------+  +--------------------------------+

+-------------------------------+   +-----------------------------------+
|           Unmount             |   |          Second Render            |
+-------------------------------+   +-----------------------------------+
|  +-------------------------+  |   |   +---------------------------+   |
|  |  componentWillUnmount   |  |   |   |      getInitialState      |   |
|  +-------------------------+  |   |   +---------------------------+   |
+-------------------------------+   |   +---------------------------+   |
                                    |   |     componentWillMount    |   |
                                    |   +---------------------------+   |
                                    |   +---------------------------+   |
                                    |   |           render          |   |
                                    |   +---------------------------+   |
                                    |   +---------------------------+   |
                                    |   |     componentDidMount     |   |
                                    |   +---------------------------+   |
                                    |                                   |
                                    +-----------------------------------+


```

- 为何 React 会按上述顺序执行生命周期？
- 为何 React 多次 render 时，会执行生命周期的不同阶段？
- 为何 getDefaultProps 只执行了1次？

# 详解React生命周期

自定义组件(ReactCompositeComponent)的生命周期主要通过三个阶段进行管理：**MOUNTING**，**RECIEVE_PROPS**，**UNMOUNTING**,他们负责通知组件当前所处的阶段，应该执行生命周期中的哪个步骤，三个阶段对应三种方法，分别是：

- MOUNTING -> mountComponent
- RECIEVE_PROPS -> updateComponent
- UNMOUNTING -> unmoutComponent

每个方法都提供了几种处理方法，其中带will前缀的方法在进入状态之前调用，带did在进入状态之后调用，3个阶段共包含5中处理方法，还有两种特殊状态的处理方法。

```
+--------------+--------------------------------------------------------------------+--------------+
|              |                                                                    |              |
|              |                                                                    |              |
|              |                                                                    |              |
|  UNMOUNTED   |                              MOUNTED                               |  UNMOUNTED   |
|              |                                                                    |              |
|              |                                                                    |              |
|              |                                                                    |              |
|              |                                                                    |              |
+--------------------------------------------------------------------------------------------------+
|              |                                                                    |              |
|              |                                                                    |              |
|              |   +----------------+    +-----------------+    +---------------+   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|  +-------->  |   |    MOUNTING    |    |   RECIEVE_PROPS |    |  UNMOUNTING   |   | +--------->  |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   |                |    |                 |    |               |   |              |
|              |   +----------------+    +-----------------+    +---------------+   |              |
|              |                                                                    |              |
|              |                                                                    |              |
|              |                                                                    |              |
+--------------+--------------------------------------------------------------------+--------------+


```

## 使用createClass创建自定义组件

createClass是创建自定义组件的入口方法，负责管理生命周期中的 **getDefaultProps** 方法，该方法在整个生命周期中只执行一次。这样所有实例初始化的props将会被共享。

当使用es6编写组件时，`class MyComponent extends React.Component`其实就是调用内部的方法`createClass`。
相关代码路径：https://github.com/facebook/react/blob/master/src/isomorphic/classic/class/ReactClass.js#L768

```javascript
/**
 * Module for creating composite components.
 *
 * @class ReactClass
 */
var ReactClass = {

  /**
   * Creates a composite component class given a class specification.
   * See https://facebook.github.io/react/docs/react-api.html#createclass
   *
   * @param {object} spec Class specification (which must define `render`).
   * @return {function} Component constructor function.
   * @public
   */
  createClass: function(spec) {
    // To keep our warnings more understandable, we'll use a little hack here to
    // ensure that Constructor.name !== 'Constructor'. This makes sure we don't
    // unnecessarily identify a class without displayName as 'Constructor'.
    var Constructor = identity(function(props, context, updater) {

      // Wire up auto-binding
      if (this.__reactAutoBindPairs.length) {
        bindAutoBindMethods(this);
      }

      this.props = props;
      this.context = context;
      this.refs = emptyObject;
      this.updater = updater || ReactNoopUpdateQueue;

      this.state = null;

      // ReactClasses doesn't have constructors. Instead, they use the
      // getInitialState and componentWillMount methods for initialization.

      var initialState = this.getInitialState ? this.getInitialState() : null;

      invariant(
        typeof initialState === 'object' && !Array.isArray(initialState),
        '%s.getInitialState(): must return an object or null',
        Constructor.displayName || 'ReactCompositeComponent'
      );

      this.state = initialState;
    });
    //原型继承父类
    Constructor.prototype = new ReactClassComponent();
    Constructor.prototype.constructor = Constructor;
    Constructor.prototype.__reactAutoBindPairs = [];
    //合并mixin
    mixSpecIntoComponent(Constructor, spec);
    
    // Initialize the defaultProps property after all mixins have been merged.
    // 初始化defaultProps。注意defaultProps没绑定到this上。
    // 在整个生命周期中.getDefaultProps只执行一次
    if (Constructor.getDefaultProps) {
      Constructor.defaultProps = Constructor.getDefaultProps();
    }

    // Reduce time spent doing lookups by setting these on the prototype.
    for (var methodName in ReactClassInterface) {
      if (!Constructor.prototype[methodName]) {
        Constructor.prototype[methodName] = null;
      }
    }

    return Constructor;
  },

};
```

## 阶段一：MOUNTING

mountComponent负责管理生命周期中 `getInitialState`，`componentWillMount`，`render`，`componentDidMount`。

由于getDefaultProps是通过构造函数进行管理的，所以是整个生命周期最先开始的执行，所以mountComponent无法调用到getDefaultProps，这就解释了为何getDefaultProps只执行一次了。

首先通过 mountComponent 装载组件，此时，将状态设置为 MOUNTING，利用 getInitialState 获取初始化 state，初始化更新队列。

若存在 componentWillMount，则执行；如果此时在 componentWillMount 中调用 setState，是不会触发 reRender，而是进行 state 合并。

其实，mountComponent 本质上是通过 递归渲染 内容的，由于递归的特性，父组件的 componentWillMount 一定在其子组件的 componentWillMount 之前调用，而父组件的 componentDidMount 肯定在其子组件的 componentDidMount 之后调用。

当渲染完成之后，若存在 componentDidMount 则触发。这就解释了 componentWillMount - render - componentDidMount 三者之间的执行顺序。

```
               mountComponent   <----------------------+
                                                       |
             +-----------------------------+           |
             |      getInitialState        |           |
             +---+---------------+---------+           |
                 | Y             |                     |
                 |               |                     |
+----------------v---------+     |                     |
|    componentWillMount    |     |                     |
+----------------+---------+     |                     |
                 |               | N                   |
                 |               |                     |
+----------------v---------+     |                     |
|       merge state        |     |                     |
+----------------+---------+     |                     |
                 | Y             |                     |
                 |               |                     |
             +---v---------------v---------+           |
             |  create Component instance  |           |
             +----------------+------------+           |
                              |                        |
                              |                        |
             +----------------v-----------+            |
             |      recursive render      +------------+
             +----------------+-----------+
                              |
             +----------------v------------+
             |      componentDidMount      |
             +-----------------------------+


```

## 阶段二 RECIEVE_PROPS

updateComponent负责管理生命周期中的componentWillReceiveProps，shouldComponentUpdate，componentWillUpdate，render，componentDidUpdate。

首先通过updateComponent更新组件，如果前后元素不一致，说明需要进行组件更新。

若存在componentWillReceiveProps，则执行，如果此时在componentWillReceiveProps中调用setState，是不会触发re-render的，而是会进行state的合并。且在componentWillReceiveProps，shouldComponentUpdate，componentWillUpdate中无法获取更新后的state，即this.state仍然是未更新的数据，因此只有在render和componentDidUpdate中才能获取到更新后的this.state。

调用shouldComponentUpdate判断是否需要进行组件更新，如果存在componentWillUpdate则执行。

updateComponent本质也是通过递归来渲染内容的，由于递归的特性，父组件的componentWillUpdate是在其子组件的componentWillUpdate之前调用的，而父组件的componentDidUpdate是在子组件的componentDidUpdate之后调用的。

当渲染完成之后，若存在componentDidUpdate，则触发，这就解释了componentWillReceiveProps，shouldComponentUpdate，componentWillUpdate，render，componentDidUpdate之间的的执行顺序

    >注意，禁止在shouldComponentUpdate和componentWillUpdate中调用setState，这会造成循环调用，直至耗光浏览器内存后崩溃。

updateComponent的执行顺序

```
               updateComponent  <---------------------+
                                                      |
                                                      |
             +------------------------------+         |
             |   componentWillReceiveProps  |         |
             +------+-------------+---------+         |
                Y   |             |                   |
                    |             |                   |
+-------------------v-------+     |                   |
|  shouldComponentUpdate    |     |                   |
+-------------------+-------+     |                   |
               Y    |             | N                 |
                    |             |                   |
+-------------------v-------+     |                   |
|  componentWillUpdate      |     |                   |
+-------------------+-------+     |                   |
                    |             |                   |
                    |             |                   |
              +-----v-------------v---------+         |
              |  create component instance  |         |
              +------------+----------------+         |
                           |                          |
                           |                          |
              +------------v----------------+         |
              |    recursive render         +---------+
              +------------+----------------+
                           |
                           |
              +------------v----------------+
              |    componentDidUpdate       |
              +-----------------------------+


```

## 阶段三 UNMOUNTING

unmountComponent负责管理生命周期中的componentWillUnmount。

如果存在componentWillUnmount，则执行并重置所有相关参数，更新队列以及状态，如果此时在componentWillUnmount调用setState，是不会触发re-render的，因为所有更新队列和更新状态都被重置为null，并清除了公共类，完成了组件的卸载。

我们经常编写一些无状态，只从父组件接受props，并根据这些属性进行渲染的简单操作组件，这不仅让组件的开发变得简单，高效，也便于对状态进行统一管理，因此，在React开发中，一个重要的原则就是让组件尽可能的是无状态的。

## 无状态组件

无状态组件只是一个render方法，并没有组件化的实例化过程，也没有实例返回，比如

```
const Helloworld= props => <div>{props.name}</div>

RenderDOM.render(<Helloworld name="hello world" />,app);

```

无状态组件没有状态，没有生命周期，只是简单的接受props渲染生成DOM解构，是一个纯粹的为渲染而生的组件。由于无状态组件有简单、便捷、高效等有点，所以有可能的话，尽量用无状态组件。

最后一张图再次归纳一下生命周期

```
+                                                                                         +                        +
|  <-------MOUNTING---------------------------------------------------------------------> | <----MOUNTED---------> |
+                                                                                         |                        +
                                                                                          |
      no  this.setState        no this.setState                           no this.setState|
                                                                                          |     +-------------+
       +-----------------+    +----------------+   +------------------+    +--------+     |     | componentDid|
       |getDefaultProps  |    |getInitialState |   |componentWillMount|    |render  |     |     | Mount       |
       +-----------------+    +----------------+   +------------------+    +--------+     |     +-------------+
                                                                                          |
+                          +                                                              |                        +
| <------RECEIVE_PROPS---> | <-------------------RECEIVE STAGE------------------------->  | <---MOUNTED----------> |
+                          +                                                              |                        |
                               no this.setState     no this.State         no this.setState|                        +
       +-----------------+     +--------------+     +----------------+      +--------+    |     +-------------+
       |componentWill    |     | shouldComponent    | componentWill  |      | render |    |     | componentDid|
       | Recei^eProps    |     | Update       |     | Update         |      +--------+    |     | Update      |
       +-----------------+     +--------------+     +----------------+                    |     +-------------+
                                                                                          |                         +
+                           +                                                             |                         |
| <-----RECEIVE_PROPS-----> | <-----------------------------------UNMOUNTED---------------------------------------> |
+                           +                                                             +                         +
       +-----------------+
       |componentWill    |
       |Unmount          |
       +-----------------+
       no this.setState


```

# 总结

- React 通过三种状态：MOUNTING、RECEIVE_PROPS、UNMOUNTING，管理整个生命周期的执行顺序
- setState 会先进行 _pendingState 更新队列的合并操作，不会立刻 reRender，因此是异步操作，且通过判断状态（MOUNTING、RECEIVE_PROPS）来控制 reRender 的时机
- 不建议在 getDefaultProps、getInitialState、shouldComponentUpdate、componentWillUpdate、render 和 componentWillUnmount 中调用 setState，特别注意：不能在 shouldComponentUpdate 和 componentWillUpdate 中调用 setState，会导致循环调用