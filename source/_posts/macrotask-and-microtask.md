---
title: macrotask和microtask
date: 2017-03-04 11:37:45
tags: [macrotask,microtask]
categories: JavaScript
---

学习一下macrotask和microtask 。
<!--more-->

# Event Loop

浏览器的Event Loop至少包含两个队列，macrotask队列和microtask队列。

Macrotasks包含生成dom对象、解析HTML、执行主线程js代码、更改当前URL还有其他的一些事件如页面加载、输入、网络事件和定时器事件。从浏览器的角度来看，macrotask代表一些离散的独立的工作。当执行完一个task后，浏览器可以继续其他的工作如页面重渲染和垃圾回收。

Microtasks则是完成一些更新应用程序状态的较小任务，如处理promise的回调和DOM的修改，这些任务在浏览器重渲染前执行。Microtask应该以异步的方式尽快执行，其开销比执行一个新的macrotask要小。Microtasks使得我们可以在UI重渲染之前执行某些任务，从而避免了不必要的UI渲染，这些渲染可能导致显示的应用程序状态不一致。

> ECMAScript规范并没有提及到event loop。实际上event loop是定义在HTML规范里的，这里也讨论了microtask和macrotask。ECMAScript规范在谈及处理promise回调时提到了jobs（其和microtask类似）。即使event loop是定义在HTML规范里的，其他的宿主环境如Node.js也使用了相同的概念。

Event loop的实现应该至少使用一个队列用于处理macrotasks，至少一个队列处理microtasks。Event loop的实际实现通常分配几个队列用于处理不同类型的macrotasks和microtasks。这使得可以对不同的任务类型进行优先级排序。例如优先考虑一些性能敏感的任务如用户输入。另一方面，因为实际上存在很多JavaScript宿主环境，所以有的event loop使用一个队列处理这两种任务也不应该感到奇怪。

Event loop基于两个基本原则：

* 同一时间只能执行一个任务。
* 任务一直执行到完成，不能被其他任务抢断。

![](/images/macrotask-microtask.png)

如上图所示，在单次的迭代中，event loop首先检查macrotask队列，如果有一个macrotask等待执行，那么执行该任务。当该任务执行完毕后（或者macrotask队列为空），event loop继续执行microtask队列。如果microtask队列有等待执行的任务，那么event loop就一直取出任务执行知道microtask为空。这里我们注意到处理microtask和macrotask的不同之处：在单次循环中，一次最多处理一个macrotask（其他的仍然驻留在队列中），然而却可以处理完所有的microtask。

当microtask队列为空时，event loop检查是否需要执行UI重渲染，如果需要则重渲染UI。这样就结束了当次循环，继续从头开始检查macrotask队列。

上图还包含了一些细节：

* 两个任务队列都放置在event loop外，这表明将任务添加和任务处理行为分离。在event loop内负责执行任务（并从队列里删除），而在event loop外添加任务。如果不是这样，那么在event loop里执行代码时，发生的任何事件都被忽略，这显然不合需求，因此我们将添加任务的行为和event loop分开进行。
* 两种类型的任务同时只能执行一个，因为JavaScript基于单线程执行模型。任务一直执行到完成而不能被其他任务中断。只有浏览器可以停止任务的执行;例如如果某个任务消耗了太多的内存和时间的话，浏览器可以中断其执行。
* 所有的microtasks都应该在下次渲染前执行完，因为其目的就是在渲染前更新应用状态。
* 浏览器通常每秒尝试渲染页面60次，以达到每秒60帧（60 fps），这个帧速率通常被认为是平滑运动的理想选择。这意味着浏览器尝试每16ms渲染一帧。上图中“update rendering”操作在事件循环中进行，这是因为在呈现页面时，页面内容不应该被另一个任务修改。这意味着如果我们应用实现平滑的效果，单个事件循环中不能占据太多时间。单个任务和由该任务生成的所有microtasks应该在16毫秒内完成。

当浏览器完成页面渲染后，下次event loop循环迭代中可能发生三种情况：

1. event loop在另一个16 ms之前执行的“is rendering needed”的判断处。因为更新UI是一个复杂的操作，如果没有明确要求渲染页面，浏览器可能在本次迭代中不执行UI渲染。
2. event loop 在上次渲染后约16 ms处达到“Is rendering needed”判断处。在这种情况下，浏览器更新UI，用户会认为应用比较流畅。
3. 执行下次任务（及其所有相关的microtask）花费时间大大超过16ms。这样浏览器将无法按照目标的帧速率重新渲染页面，UI也将不会更新。如果运行任务代码不占用太多时间（超过几百毫秒），这种延迟甚至可能感知不到，尤其是对于没有太多动画的页面。另一方面，如果我们花费太多时间，或者页面中含有动画，用户可能认为网页缓慢和没有响应。在最坏的情况下，如果任务执行超过几秒钟，用户的浏览器会显示“无响应脚本”消息。

> 处理事件时应注意其发生的频率和处理所需时间。如在处理鼠标移动事件时应该格外小心。移动鼠标会导致大量的事件排队，因此在该鼠标移动处理程序中执行任何复杂的操作都可能导致应用变得很不流畅。

