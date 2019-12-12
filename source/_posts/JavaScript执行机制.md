---
title: JavaScript执行机制
date: 2019-10-07 21:02:34
tags: [前端,JS]
categories: 前端
keywords: [前端,JS,执行机制]
description: 
---

> 众所周知，JavaScript是一门单线程语言，但这并不意味着JavaScript会逐行执行，因为JS中有诸如 `setTimeout` , `Promise` 这类语法的存在，所以JS得执行顺序便变得扑朔迷离起来，本文就从JS的执行机制一起来探究JS的执行顺序。

<!--more-->

同步和异步
-------------
因为JavaScript时单线程语言，所以JS的任务需要一个个顺序执行。但当我们的页面需要请求文件、图片等耗时较久的操作时，显然一个个执行，将影响页面的渲染和用户体验，所以我们提出了两个概念：`同步任务` 和 `异步任务` 。

### 同步任务
定义：如果在函数A返回的时候，调用者就能够得到预期结果(即拿到了预期的返回值或者看到了预期的效果)，那么这个函数就是同步的。
```
console.log("Hi");
```

### 异步任务
定义：如果在函数A返回的时候，调用者还不能够得到预期结果，而是需要在将来通过一定的手段得到，那么这个函数就是异步的。
```
fs.readFile('foo.txt', 'utf8', function(err, data) {
  console.log(data);
});
```

事件循环
-------------
在我们了解了同步任务和异步任务后，我们来看看他们具体是如何执行的。
![Alt text][01]
[01]: ../assets/20191007/1.jpg

+ 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入Event Table并注册函数。
+ 当指定的事情完成时，Event Table会将这个函数移入Event Queue。
+ 主线程内的任务执行完毕为空，会去Event Queue读取对应的函数，进入主线程执行。
+ 上述过程会不断重复，也就是常说的Event Loop(事件循环)。

js引擎存在monitoring process进程，会持续不断的检查主线程执行栈是否为空，一旦为空，就会去Event Queue那里检查是否有等待被调用的函数。
```
let data = [];
$.ajax({
    url:www.javascript.com,
    data:data,
    success:() => {
        console.log('发送成功!');
    }
})
console.log('代码执行结束');
```

+ ajax进入Event Table，注册回调函数success。
+ 执行console.log('代码执行结束')。
+ ajax事件完成，回调函数success进入Event Queue。
+ 主线程从Event Queue读取回调函数success并执行。

setTimeout
-------------
setTimeout 函数常用于延时执行。
```
setTimeout(() => {
    task();
},3000)
console.log('执行console');

//执行console
//3秒后，执行task()
```

而有时候，setTimeout 的延时时间会超出我们设定的时间，这是因为JS的同步任务可能占用了大量时间，setTimeout 得等到同步任务执行完毕后才能执行。
```
setTimeout(() => {
    task()
},3000)

sleep(10000000);
```

+ task()进入Event Table并注册,计时开始。
+ 执行sleep函数，很慢，非常慢，计时仍在继续。
+ 3秒到了，计时事件timeout完成，task()进入Event Queue，但是sleep也太慢了吧，还没执行完，只好等着。
+ sleep终于执行完了，task()终于从Event Queue进入了主线程执行。

所以由上可知， `setTimeout(fn,0)` 的含义是指定某个任务在主线程最早可得的空闲时间执行。

setInterval
-------------
+ setInterval 与 setTimeout 相近，setInterval 会每隔指定的时间将注册的函数置入Event Queue，如果前面的任务耗时太久，同样需要等待。
+ 对于setInterval(fn,ms)来说，我们已经知道不是每过ms秒会执行一次fn，而是每过ms秒，会有fn进入Event Queue。一旦setInterval的回调函数fn执行时间超过了延迟时间ms，那么就完全看不出来有时间间隔了。

process.nextTick(callback)
-------------
process.nextTick(callback)类似node.js版的"setTimeout"，在事件循环的下一次循环中调用 callback 回调函数。

宏任务和微任务
-------------
除了广义的同步任务和异步任务，我们对任务有更精细的定义：
+ macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
+ micro-task(微任务)：Promise，process.nextTick

不同类型的任务会进入对应的Event Queue，比如setTimeout和setInterval会进入相同的Event Queue。

事件循环的顺序，决定js代码的执行顺序。进入整体代码(宏任务)后，开始第一次循环。接着执行所有的微任务。然后再次从宏任务开始，找到其中一个任务队列执行完毕，再执行所有的微任务。

```
setTimeout(function() {
    console.log('setTimeout');
})

new Promise(function(resolve) {
    console.log('promise');
}).then(function() {
    console.log('then');
})

console.log('console');
```

+ 这段代码作为宏任务，进入主线程。
+ 先遇到setTimeout，那么将其回调函数注册后分发到宏任务Event Queue。(注册过程与上同，下文不再描述)
+ 接下来遇到了Promise，new Promise立即执行，then函数分发到微任务Event Queue。
+ 遇到console.log()，立即执行。
+ 整体代码script作为第一个宏任务执行结束，看看有哪些微任务？我们发现了then在微任务Event Queue里面，执行。
+ 第一轮事件循环结束了，我们开始第二轮循环，当然要从宏任务Event Queue开始。我们发现了宏任务Event Queue中setTimeout对应的回调函数，立即执行。
+ 结束。

![Alt text][02]
[02]: ../assets/20191007/2.jpg

复杂案例
-------------
```
console.log('1');

setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
})
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})

setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
})
```
第一轮事件循环流程分析如下：
+ 整体script作为第一个宏任务进入主线程，遇到console.log，输出1。
+ 遇到setTimeout，其回调函数被分发到宏任务Event Queue中。我们暂且记为setTimeout1。
+ 遇到process.nextTick()，其回调函数被分发到微任务Event Queue中。我们记为process1。
+ 遇到Promise，new Promise直接执行，输出7。then被分发到微任务Event Queue中。我们记为then1。
+ 又遇到了setTimeout，其回调函数被分发到宏任务Event Queue中，我们记为setTimeout2。

|    宏任务Event Queue    |       微任务Event Queue       |
|:-------:|:-------------:|
|   setTimeout1  |     process1    |
|   setTimeout2  |     then1    |

+ 上表是第一轮事件循环宏任务结束时各Event Queue的情况，此时已经输出了1和7。
+ 我们发现了process1和then1两个微任务。
+ 执行process1,输出6。
+ 执行then1，输出8。

第二轮时间循环从setTimeout1宏任务开始：
+ 首先输出2。接下来遇到了process.nextTick()，同样将其分发到微任务Event Queue中，记为process2。new Promise立即执行输出4，then也分发到微任务Event Queue中，记为then2。

|    宏任务Event Queue    |       微任务Event Queue       |
|:-------:|:-------------:|
|   setTimeout2  |     process2    |
|    |     then2    |

+ 第二轮事件循环宏任务结束，我们发现有process2和then2两个微任务可以执行。
+ 输出3。
+ 输出5。
+ 第二轮事件循环结束，第二轮输出2，4，3，5。

第三轮事件循环开始，此时只剩setTimeout2了，执行。

+ 直接输出9。
+ 将process.nextTick()分发到微任务Event Queue中。记为process3。
+ 直接执行new Promise，输出11。
+ 将then分发到微任务Event Queue中，记为then3。

|    宏任务Event Queue    |       微任务Event Queue       |
|:-------:|:-------------:|
|     |     process3    |
|    |     then3    |

+ 第三轮事件循环宏任务执行结束，执行两个微任务process3和then3。
+ 输出10。
+ 输出12。
+ 第三轮事件循环结束，第三轮输出9，11，10，12。

整段代码，共进行了三次事件循环，完整的输出为1，7，6，8，2，4，3，5，9，11，10，12。

总结
-------------
以上就是JavaScript的执行机制，最后在强调一下JavaScript的两个基本特征。
+ javascript是一门单线程语言
+ Event Loop是javascript的执行机制

> 参考链接：[这一次，彻底弄懂 JavaScript 执行机制（ssssyoki）](https://juejin.im/post/59e85eebf265da430d571f89)