---
title: JavaScript中的异步：Event Loop及其他
date: 2017-03-15 17:31:28
tags: [JavaScript]
categories: JavaScript
---

简单地说，JavaScript 是单线程执行的语言，但在使用中有很多异步执行的情况。异步的本质是用其他方式（相对同步）控制程序的执行顺序，这与其他语言中的多线程模型不同，所以常常有人对非顺序 JavaScript 代码的运行结果感到困惑不解。
<!--more-->

# 一段简单的代码

任何使用过 JavaScript 的程序员都能说出下面这段代码的输出

```javascript
console.log("A");

setTimeout(() => {
  console.log("B");
}, 100);

console.log("C");
```

先后顺序是 A、C、B，因为第二个参数的作用是指定延迟的毫秒数，这段代码只有一个 setTimeout，所以不会让人迷惑。

对类似程序的解释通常是由 setTimeout 设置一个定时器，在指定毫秒数后调用回调函数。然而，它的执行机制并不是这么简单。**实际上，setTimeout 的作用是在指定的毫秒数之后，在得到机会时，将 callback 放入 Event Loop Queue**。

# Event Loop

首先要抛出一些概念，通常所说的 JavaScript Engine 是指负责执行一个一个 chunk 的程序，它依赖宿主环境的调度，也需要通过宿主环境与操作系统产生关联并得到支持。JavaScript Engine 是 JavaScript Runtime(Hosting Environment) 的一部分。

每个 chunk 通常是以 function 为单位，一个 chunk 执行完成后，才会执行下一个 chunk。下一个 chunk 是什么呢？取决于当前 Event Loop Queue 中的队首。Event Loop Queue 中存放的都是消息，每个消息关联着一个函数，JavaScript Engine 就按照队列中的消息顺序执行它们，也就是执行 chunk。

所以上面的 setTimeout 实际执行起来更接近这样：

* chunk1执行：由 setTimeout 启动定时器（100毫秒）
* chunk2执行：得到机会，将 callback 放入 Event Loop Queue
* chunk3执行：此 callback 执行

不难发现，**得到机会** 很重要！这也就可以解释用 setTimeout 延迟 1000 不一定是准确的，而是会至少延迟一秒。因为如果还有其他的任务在前面，它要等待那些任务对应的消息都出队，也就是程序都执行完成，它才能将 callback 放入队列。也就是实际延迟会大于或等于一秒。

通常所说的触发了一个事件，就是指这个 event listener 得到了执行。与 setTimeout 这个例子中的概念一样，这也是一次 chunk 的执行。像这样一个一个执行 chunk 的过程就叫 **Event Loop**。

还有一个经常提到的概念叫「无阻塞」，JavaScript 中的无阻塞就是指这种 Event Loop 模型。除去 alert 或同步 Ajax 请求等历史原因造成的问题，程序总是不会出现阻塞；也就是说 JavaScript Engine 总是可以处理下一个任务，如处理用户对浏览器的操作。

# 一些简单的小例子

将 setTimeout 加入 try 语句之中，结果会如何？

```javascript

try {
  setTimeout(() => {
    throw new Error("Error - from try statement");
  }, 0);
} catch (e) {
  console.error(e);
}

```

try catch 与 setTimeout 不在同一个 chunk，所以……你懂的。

再看下一个。

下面的堆栈信息会输出 C - B - A 吗？

```javascript

setTimeout(function A() {
  setTimeout(function B() {
    setTimeout(function C() {
      throw new Error("Error - from function C");
    }, 0);
  }, 0);
}, 0);

```

它们并不对应同一条 Event Loop Queue 中的消息，分别有各自的调用栈，所以错误栈里面只有 C。

# Job Queue

Job 是 ES6 中新增的概念，它与 Promise 的执行有关，可以理解为等待执行的任务；Job Queue 就是这种类型的任务的队列。JavaScript Runtime 对于 Job Queue 与 Event Loop Queue 的处理有所不同。

相同点：

* 都用作先进先出队列

相异点：

* 每个 JavaScript Runtime 可以有多个 Job Queue，但只有一个 Event Loop Queue
* 当 JavaScript Engine 处理完当前 chunk 后，优先执行所有的 Job Queue，然后再处理 Event Loop Queue

ES6 中，一个 Promise 就是一个 PromiseJob，一种 Job。

再来观察一段小程序：

```javascript

console.log("A");

setTimeout(() => {
  console.log("A - setTimeout");
}, 0);

new Promise((resolve) => {
  resolve();
})
.then(() => {
  return console.log("A - Promise 1");
})
.then(() => {
  return console.log("B - Promise 1");
});

new Promise((resolve) => {
  resolve();
})
.then(() => {
  return console.log("A - Promise 2");
})
.then(() => {
  return console.log("B - Promise 2");
})
.then(() => {
  return console.log("C - Promise 2");
});

console.log("AA");

```

在原生支持 Promise 的环境，输出是这样：

```
A
AA
A - Promise 1
A - Promise 2
B - Promise 1
B - Promise 2
C - Promise 2
A - setTimeout
```

理解这个输出:

* A 与 AA 最先输出，因为它们不是异步任务，属于第一个 chunk。
* Promise 1 与 Promise 2 先于 setTimeout 执行，因为 Job Queue 的执行优先于 Event Loop Queue。
* Promise 1 与 Promise 2 各自的输出都是顺序的，因为 Job Queue 是先进先出队列，同一 Job Queue 中的任务顺序执行。
* Promise 1 与 Promise 2 的后续任务是交错的，因为 Promise 1 与 Promise 2 都是独立的 PromiseJob（job 的其中一种），属于不同的 Job Queue，它们之间的顺序规范中没有规定。

# 并发

文章开头，我说「简单地说，JavaScript 是单线程执行的语言」，现在可以说得稍微复杂一点了：JavaScript Engine 对 JavaScript 程序的执行是单线程的，但是 JavaScript Runtime（整个宿主环境）并不是单线程的；而且，几乎所有的异步任务都是并发的，例如多个 Job Queue、Ajax、Timer、I/O(Node)等等。

上面说的是 JavaScript Runtime 层面，JavaScript 执行本身，也有一些特殊情况，例如：一个 Web Worker 或者一个跨域的 iframe，也是独立的线程，有各自的内存空间（栈、堆）以及 Event Loop Queue。要与这些不同的线程通信，只能通过 postMessage。一次 postMessage 就是在另一个线程的 Event Loop Queue 中加入一条消息。