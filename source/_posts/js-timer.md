---
title: setTimeout vs setInterval
date: 2021-03-10 19:44:12
tags: 
    - 定时器
thumbnail: /imgs/20210310-bg.jpg
---

日常开发中我们常常会遇到这样的场景：进入页面后间隔一定的时间调用某个方法或者是每隔一定的时间定时调用某个方法。这个时候，就可以用到setTimeout 或 setInterval。

setTimeout和setInteval是window对象上两个主要的定时方法，他们的语法基本相同，但完成功能的却是不同的

setTimeout(fn,time) 在 一定的时间 后将fn加入到待执行的任务队列，并返回一个定时器的序号（如果需要清除定时器，则需要此序号）

setInteval(fn,time) 每隔一定的时间间隔将fn加入到待执行的任务队列，并返回一个定时器的序号（如果需要清除定时器，则需要此序号）

其实呢，setTimeout 与 setInterval 的执行时间都并不准确，因为他们都属于宏任务，这就涉及到js的事件循环，这里就不在讲述，由于前一篇文章已经
写过，请感兴趣的看官移步至  [[浅谈Js事件循环](https://edw4rd.cn/2021/03/09/js-eventloop)]

定时器的清除：

clearTimeout(id) 
clearInterval(id)

注意：定时器需要手动清除，并且clearTimeout和clearInterval都可以清除setTimeout或setInterval，但并不建议这样做，容易造成混淆。

### 为什么尽量别用setInterval

#### 1. setInterval会丢失部分时间间隔
#### 2. setInterval可能导致多个定时器会连续执行 
因为setInterval是 每个固定时间将function推入任务队列中，如果主线程一直不空闲，异步任务队列里就会有大量的待执行的function，一旦主线程空闲，此时大量的fucntion会一次执行，这样就丢失了时间间隔，大量的定时器就会连续执行。

### setTimeout模拟setInterval
综上所述，在某些情况下，setInterval 缺点是很明显的，为了解决这些弊端，可以使用 settTimeout() 代替。

具体代码如下：
```
let timer = null
function interval(func, wait){
    let interv = function(){
        func.call(null);
        timer=setTimeout(interv, wait);
    };
    timer= setTimeout(interv, wait);
 }
```
调用interval
```
interval(() => console.log(1),1000)
```
清除定时器

```
clearTimeout(timer)
```

### 参考
[为什么要用setTimeout模拟setInterval?](https://blog.csdn.net/b954960630/article/details/82286486)