---
title: 浅谈Js事件循环
date: 2021-03-09 20:01:44
tags: 事件循环 
thumbnail: /imgs/20210309-bg.png
---

JavaScript有一个基于事件循环的并发模型，事件循环(Event Loop)负责执行代码、收集和处理事件以及执行队列中的子任务。

之所以称之为 事件循环，是因为它经常按照类似如下的方式来被实现

```
while (queue.waitForMessage()) {
  queue.processNextMessage()
}
```
不过EventLoop在浏览器中与node环境中还是有些许差异

### 浏览器中EventLoop

代码执行顺序：默认先执行主栈中的代码，主栈执行完成后；一次性清空微任务对列，微任务对列清空后，取出第一个宏任务到主栈中执行，执行完成后再去检查微任务对列是否有未执行任务，如果有清空，若没有取下一个宏任务到主栈中执行，执行完成后再去检查微任务对列是否有未执行任务，如果有清空...如此形成一个事件环

#### 宏任务

setTiemout setInterval setImmediate(非标准，请尽量不要在生产环境中使用它)

messageChannel(web worker中可用，用于线程通信)

#### 微任务

Promise.resolve().then()

MutationObserver(提供了监视对DOM树所做更改的能力。它被设计为旧的Mutation Events功能的替代品，该功能是DOM3 Events规范的一部分)

小测试：
```
console.log('start')
setTimeout(()=>{
    console.log(1);
    Promise.resolve().then(()=>{
        console.log(2)
    })
},0);
Promise.resolve().then(data=>{
    console.log(3);
    setTimeout(()=>{
        console.log(4);
    })
})
console.log('end');
// start end 3 1 2 4

```

进阶测试：

```
async function async1() {
  console.log('async1Start');
  await async2(); 
  console.log('async1End');
}
async function async2() {
  console.log('async2');
}
console.log('scriptStart');
setTimeout(function () {
  console.log('setTimeout');
}, 0)
async1();
new Promise(function (resolve) {
  console.log('promise1');
  resolve();
}).then(function () {
  console.log('promise2');
});
console.log('scriptEnd');
//scriptStart async1Start async2 promise1 scriptEnd async1End promise2 setTimeout

```

对于async函数中await 如上图中的async1函数可以转换成如下：
function async1(){
  console.log('async1Start');
  new Promise((resolve,reject) => {
    reolve(async2())
  }).then(() => {
    console.log('async1End');
  })
}

### Nodejs中的EventLoop

Node.js是运行在服务端的js，虽然他也用到了V8引擎，但是他的服务目的和环境不同，导致了他API与原生JS有些区别，他的Event Loop还要处理一些I/O，比如新的网络连接等，所以与浏览器Event Loop也是不一样的。Node的Event Loop是分阶段的，如下图所示：

{% fb_img /imgs/eventloop-0.png node事件循环阶段图 %}

#### 宏任务
timers：执行的是setTimeout()和setInterval()的回调,timer指定的是一个下限时间而非准确时间，在达到这个下限后执行回调。在指定的时间后timer会尽可能早的执行回调，但是呢，系统调度或者其他调度的执行可能会延迟timer的执行。


I/O callbacks：执行一些系统操作的回调

idle, prepare：仅系统内部的调用

poll：检索新的I/O事件;执行与I/O相关的回调(除了close回调、计时器调度的回调和setImmediat()之外，几乎所有回调都执行);节点将在适当的时候在这里阻塞

check：setImmediate回调在这里触发

close callbacks：比如socket.on('close', ...)

#### 微任务
promise.then  process.nextTick

process.nextTick()的优先级高于所有的微任务,每一次清空微任务列表的时候，都是先执行 process.nextTick()

test1:
~~~
setTimeout(()=>{
  console.log('setTimeout');
},0)
setImmediate(()=>{
  console.log('setImmediate');
})
~~~
多次在node环境下运行test1中的代码 你会发现setTimeout与setImmediate的打印顺序并不一致。
1. 第一种情况，也是最常见的情况，是先输出setTimeout 然后在输出 setImmediate
2. 第二种情况，先输出 setImmediate，在输出 setTimeout.之所以会这样输出，是因为setTimeout第二个参数即使设置为0，在node环境下也会被改为1，所以输出情况取决于当前主线程的阻塞时间是否大于1ms,在大于1ms的情况下,则setImmediate优先输出，反之则是setTimeout优先输出。
   
所以运行以下不在 I/O 周期（即主模块）内的脚本,setTimeout 与 setImmediate的执行顺序与进程性能有关，并不确定,但是，如果你把这两个函数放入一个 I/O 循环内调用，setImmediate 总是被优先调用,如下：

test2:
~~~
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {s
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
 // immediate
 // timeout
~~~

使用 setImmediate() 相对于setTimeout() 的主要优势是，如果setImmediate()是在 I/O 周期内被调度的，那它将会在其中任何的定时器之前执行，跟这里存在多少个定时器无关

### 参考链接

#### [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop]([https://](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop))

#### [https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/](https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/)