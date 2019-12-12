---
title: ES6知识点梳理
date: 2019-08-31 22:53:35
tags: [前端,JS,ES6]
categories: 前端
keywords: [前端,JS,ES6]
description: 
---

最近拜读了阮一峰老师的ES6详解，虽然ES9已经大摇大摆亮相，但是ES6依然是一个跨时代的版本，其中不乏很多目前使用率颇高，功能性能上大为改进的新语法，所以还是决定对ES6做一个学习总结，也对其中部分闪光语法做详细解释。

<!--more-->

思维导图
-------------
![Alt text][01]
[01]: ../assets/20190831/es6.png

Promise对象
-------------
### 含义:
Promise 是异步编程的一种解决方案，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从它可以获取异步操作的消息。

### 特点:
+ 对象的状态不受外界影响。
+ 一旦状态改变，就不会再变，任何时候都可以得到这个结果。

### 基本用法：
```
var promise = new Promise(function(resolve, reject) { 
  // ... some code 
  if (/* 异步操作成功 */){ 
    resolve(value); 
  } else { 
    reject(error); 
  } 
});
```
Promise 实例生成以后，可以用 then 方法分别指定 resolved 状态和 rejected 状态的回调函数。
```
promise.then(function(value) { 
  // success 
}, function(error) { 
  // failure 
});
```

### Promise.prototype.then()
Promise 实例具有 then 方法，也就是说， then 方法是定义在原型对 象 Promise.prototype 上的。
```
getJSON("/post/1.json").then(function(post) { 
    return getJSON(post.commentURL); 
}).then(function funcA(comments) { 
    console.log("resolved: ", comments); 
}, function funcB(err){ 
    console.log("rejected: ", err); 
});
```

### Promise.prototype.catch()
Promise.prototype.catch 方法是 .then(null, rejection) 的别名，用于指定发生错误时的回调函数。
```
getJSON('/post/1.json').then(function(post) { 
    return getJSON(post.commentURL); 
}).then(function(comments) { 
    // some code 
}).catch(function(error) { 
    // 处理前面三个Promise产生的错误 
});
```

### Promise.all()
```
var p = Promise.all([p1, p2, p3]);
```
+ Promise.all 方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。
+ Promise.all 方法接受一个数组作为参数。
+ 只有 p1 、 p2 、 p3 的状态都变成 fulfilled ， p 的状态才会变 成 fulfilled ，此时 p1 、 p2 、 p3 的返回值组成一个数组，传递给 p 的回调函数。
+ 只要 p1 、 p2 、 p3 之中有一个被 rejected ， p 的状态就变成 rejected ，此时第一个被 reject 的实例的返回值，会传递给 p 的回调函数。

```
var promises = [2, 3, 5, 7, 11, 13].map(function (id) { 
    return getJSON('/post/' + id + ".json"); 
}); 
Promise.all(promises).then(function (posts) { 
    // ... 
}).catch(function(reason){ 
    // ... 
});
```

### Promise.race()
```
var p = Promise.race([p1, p2, p3]);
```
+ Promise.race 方法同样是将多个Promise实例，包装成一个新的Promise实例。
+ 方法接受一个数组作为参数。
+ 只要 p1 、 p2 、 p3 之中有一个实例率先改变状态， p 的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给 p 的回调函数。

```
const p = Promise.race([ 
    fetch('/resource-that-may-take-a-while'), 
    new Promise(function (resolve, reject) { 
        setTimeout(() => reject(new Error('request timeout')), 5000) 
    }) 
]); 
p.then(response => console.log(response)); 
p.catch(error => console.log(error));
```

### Promise.resolve()
参数分成四种情况:
#### 参数是一个Promise实例：
不做任何修改、原封不动地返回这个实例。

#### 参数是一个 thenable 对象：
将这个对象转为Promise对象，然后就立即执行 thenable 对象的 then 方法。
```
let thenable = { 
  then: function(resolve, reject) { 
    resolve(42); 
  } 
};
let p1 = Promise.resolve(thenable); 
p1.then(function(value) { 
  console.log(value); // 42 
});
```

#### 参数不是具有 then 方法的对象，或根本就不是对象：
返回一个新的Promise对象，状态为resolved。
```
var p = Promise.resolve('Hello'); 
p.then(function (s){ 
  console.log(s) 
}); 
// Hello
```

#### 不带有任何参数：
直接返回一个 resolved 状态的Promise对象。
```
var p = Promise.resolve(); 
p.then(function () { 
  // ... 
});
```

#### 执行顺序
```
setTimeout(function () { 
  console.log('three'); 
}, 0); 
Promise.resolve().then(function () { 
  console.log('two'); 
}); 
console.log('one');

//one
//two
//three
```
+ setTimeout(fn, 0) 在下一轮“事件循环”开始时执行。
+ Promise.resolve() 在本轮“事件循环”结束时执行。
+ console.log('one') 则是立即执行，因此最先输出。

### Promise.reject()
返回一个新的 Promise 实例，该实例的状态为 rejected。
```
var p = Promise.reject('出错了'); 
// 等同于 
var p = new Promise((resolve, reject) => reject('出错了')) 
p.then(null, function (s) { 
  console.log(s) 
}); 
// 出错了
```

### done()
总是处于回调链的尾端，保证抛出任何可能出现的错误。
```
asyncFunc() 
  .then(f1) 
  .catch(r1) 
  .then(f2) 
  .done();
```

### finally()
+ finally 方法用于指定不管Promise对象最后状态如何，都会执行的操作。
+ 它与 done 方法的最大区别，它接受一个普通的回调函数作为参数，该函数不管怎样都必须执行。

```
server.listen(0) 
  .then(function () { 
      // run test 
  })
  .finally(server.stop);
```

### Promise.try()
+ 让同步函数同步进行，异步函数异步进行。
+ 可以更好的管理异常。

```
Promise.try(database.users.get({id: userId})) 
  .then(...) 
  .catch(...)
```

Iterator和for...of循环
-------------
### Iterator（遍历器）的概念
它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署Iterator接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。
遍历过程：
+ 创建一个指针对象，指向当前数据结构的起始位置。也就是说，遍历器对象本质上，就是一个指针对象。
+ 第一次调用指针对象的 next 方法，可以将指针指向数据结构的第一个成员。
+ 第二次调用指针对象的 next 方法，指针就指向数据结构的第二个成员。
+ 不断调用指针对象的 next 方法，直到它指向数据结构的结束位置。

```
var it = makeIterator(['a', 'b']); 
it.next() // { value: "a", done: false } 
it.next() // { value: "b", done: false } 
it.next() // { value: undefined, done: true } 
function makeIterator(array) { 
  var nextIndex = 0; 
  return { 
    next: function() { 
      return nextIndex < array.length ? 
      {value: array[nextIndex++], done: false} : 
      {value: undefined, done: true}; 
    } 
  }; 
}
```

### 调用 Iterator 接口的场合
#### 解构赋值
对数组和 Set 结构进行解构赋值时，会默认调用 Symbol.iterator 方法。
```
let set = new Set().add('a').add('b').add('c'); 
let [x,y] = set; 
// x='a'; y='b' 
let [first, ...rest] = set; 
// first='a'; rest=['b','c'];
```

#### 扩展运算符
扩展运算符（...）也会调用默认的 Iterator 接口。
```
var str = 'hello'; 
[...str] // ['h','e','l','l','o']
```

#### yield*
yield* 后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口。
```
let generator = function* () { 
  yield 1; 
  yield* [2,3,4]; 
  yield 5; 
};
var iterator = generator(); 
iterator.next() // { value: 1, done: false } 
iterator.next() // { value: 2, done: false } 
iterator.next() // { value: 3, done: false } 
iterator.next() // { value: 4, done: false } 
iterator.next() // { value: 5, done: false } 
iterator.next() // { value: undefined, done: true }
```

#### 其他场合
由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。
+ for...of 
+ Array.from() 
+ Map(), Set(), WeakMap(), WeakSet()（比如 new Map([['a',1], ['b',2]]) ） 
+ Promise.all() 
+ Promise.race()

### 字符串的 Iterator 接口
字符串是一个类似数组的对象，也原生具有 Iterator 接口。
```
var someString = "hi"; 
typeof someString[Symbol.iterator] 
// "function" 
var iterator = someString[Symbol.iterator](); 
iterator.next() // { value: "h", done: false } 
iterator.next() // { value: "i", done: false } 
iterator.next() // { value: undefined, done: true }
```

### 遍历器对象的return()，throw()
+ 遍历器对象除了具有 next 方法，还可以具有 return 方法和 throw 方法。
+ return 方法的使用场合是，如果 for...of 循环提前退出（通常是因为出错， 或者有 break 语句或 continue 语句），就会调用 return 方法。如果一个对象在完成遍历前，需要清理或释放资源，就可以部署 return 方法。

```
function readLinesSync(file) { 
  return { 
    next() { 
      return { done: false }; 
    },
    return() { 
      file.close(); 
      return { done: true }; 
    }, 
  }; 
}

//return会被调用的情况
// 情况一 
for (let line of readLinesSync(fileName)) { 
  console.log(line); 
  break; 
}

// 情况二 
for (let line of readLinesSync(fileName)) { 
  console.log(line); 
  continue; 
}

// 情况三 
for (let line of readLinesSync(fileName)) { 
  console.log(line); 
  throw new Error(); 
}
```

Generator函数
-------------
### 概述
+ Generator 函数是一个状态机，封装了多个内部状态。
+ 执行 Generator 函数会返回一个遍历器对象，也就是说，Generator 函数除了状态机，还是一个遍历器对象生成函数。
+ 调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象

```
function* helloWorldGenerator() { 
  yield 'hello'; 
  yield 'world'; 
  return 'ending'; 
}
var hw = helloWorldGenerator();

hw.next() 
// { value: 'hello', done: false } 
hw.next() 
// { value: 'world', done: false } 
hw.next() 
// { value: 'ending', done: true } 
hw.next() 
// { value: undefined, done: true }
```

### yield 表达式
next 方法的运行逻辑:
+ 遇到 yield 表达式，就暂停执行后面的操作，并将紧跟在 yield 后面的那个表达式的值，作为返回的对象的 value 属性值。
+ 下一次调用 next 方法时，再继续往下执行，直到遇到下一个 yield 表达式。
+ 如果没有再遇到新的 yield 表达式，就一直运行到函数结束，直到 return 语句为止，并将 return 语句后面的表达式的值，作为返回的对象的 value 属性值。
+ 如果该函数没有 return 语句，则返回的对象的 value 属性值为 undefined 。

注意点：
+ yield 表达式只能用在 Generator 函数里面，用在其他地方都会报错。
+ yield 表达式如果用在另一个表达式之中，必须放在圆括号里面。
```
function* demo() { 
  console.log('Hello' + yield); // SyntaxError 
  console.log('Hello' + yield 123); // SyntaxError 
  console.log('Hello' + (yield)); // OK 
  console.log('Hello' + (yield 123)); // OK 
}
```

### next 方法的参数
yield 表达式本身没有返回值，或者说总是返回 undefined 。 next 方法可以带一个参数，该参数就会被当作上一个 yield 表达式的返回值。
```
function* f() { 
  for(var i = 0; true; i++) { 
    var reset = yield i; 
    if(reset) { 
      i = -1; 
    } 
  } 
}
var g = f(); 
g.next() // { value: 0, done: false } 
g.next() // { value: 1, done: false } 
g.next(true) // { value: 0, done: false }
```

### for...of 循环
for...of 循环可以自动遍历 Generator 函数时生成的 Iterator 对象，且此时不再需要调用 next 方法。
```
function* numbers () { 
  yield 1 
  yield 2 
  return 3 
  yield 4 
}

// 扩展运算符 
[...numbers()] // [1, 2] 

// Array.from 方法 
Array.from(numbers()) // [1, 2] 

// 解构赋值 
let [x, y] = numbers(); 
x // 1 
y // 2 

// for...of 循环 
for (let n of numbers()) { 
  console.log(n) 
}
// 1
// 2
```

### Generator.prototype.throw()
+ Generator 函数返回的遍历器对象，都有一个 throw 方法，可以在函数体外抛出错误，然后在 Generator 函数体内捕获。
+ Generator 函数体内抛出的错误，也可以被函数体外的 catch 捕获。

```
var g = function* () { 
  try { 
    yield; 
  } catch (e) { 
    console.log('内部捕获', e); 
  } 
};
var i = g(); 
i.next(); 
try { 
  i.throw('a'); 
  i.throw('b'); 
} catch (e) { 
  console.log('外部捕获', e); 
}
// 内部捕获 a 
// 外部捕获 b
```

### Generator.prototype.return()
Generator 函数返回的遍历器对象，还有一个 return 方法，可以返回给定的值，并且终结遍历 Generator 函数。
```
function* gen() { 
  yield 1; 
  yield 2; 
  yield 3; 
}
var g = gen(); 
g.next() // { value: 1, done: false } 
g.return('foo') // { value: "foo", done: true } 
g.next() // { value: undefined, done: true }
```

### yield* 表达式
+ 用来在一个 Generator 函数里面执行另一个Generator 函数。
+ yield* 命令可以很方便地取出嵌套数组的所有成员。

```
function* foo() { 
  yield 'a'; 
  yield 'b'; 
}
function* bar() { 
  yield 'x'; 
  yield* foo(); 
  yield 'y'; 
}
// "x" 
// "a" 
// "b" 
// "y"
```

### Generator 函数的 this
Generator 函数总是返回一个遍历器，ES6 规定这个遍历器是 Generator 函数的实例，也继承了 Generator 函数的 prototype 对象上的方法。
```
function* g() {} 
g.prototype.hello = function () { 
  return 'hi!'; 
};
let obj = g(); 
obj instanceof g // true 
obj.hello() // 'hi!'
```

### 使用
#### Generator 与状态机
```
//ES5
var ticking = true; 
var clock = function() { 
  if (ticking) 
    console.log('Tick!'); 
  else
    console.log('Tock!'); 
  ticking = !ticking;
}

//ES6
var clock = function* () { 
  while (true) { 
    console.log('Tick!'); 
    yield; 
    console.log('Tock!'); 
    yield; 
  } }
;
```

### 应用
#### 异步操作的同步化表达
```
function* main() { 
  var result = yield request("http://some.url"); 
  var resp = JSON.parse(result); 
  console.log(resp.value); 
}

function request(url) { 
  makeAjaxCall(url, function(response){ 
    it.next(response); 
  }); 
}

var it = main(); 
it.next();
```

#### 控制流管理
```
function* longRunningTask(value1) { 
  try { 
    var value2 = yield step1(value1); 
    var value3 = yield step2(value2); 
    var value4 = yield step3(value3); 
    var value5 = yield step4(value4); 
    // Do something with value4 
  } catch (e) { 
    // Handle any error from step1 through step4 
  } 
}

scheduler(longRunningTask(initialValue)); 
function scheduler(task) { 
  var taskObj = task.next(task.value); 
  // 如果Generator函数未结束，就继续调用 
  if (!taskObj.done) { 
    task.value = taskObj.value scheduler(task); 
  } 
}
```

#### 部署 Iterator 接口
```
function* makeSimpleGenerator(array){ 
  var nextIndex = 0; 
  while(nextIndex < array.length){ 
    yield array[nextIndex++]; 
  } 
}
var gen = makeSimpleGenerator(['yo', 'ya']); 
gen.next().value // 'yo' 
gen.next().value // 'ya' 
gen.next().done // true
```

#### 作为数据结构
```
function *doStuff() { 
  yield fs.readFile.bind(null, 'hello.txt'); 
  yield fs.readFile.bind(null, 'world.txt'); 
  yield fs.readFile.bind(null, 'and-such.txt'); 
}

for (task of doStuff()) { 
  // task是一个函数，可以像回调函数那样使用它 
}
```

Generator函数的异步应用
-------------
### Thunk 函数
编译器的“传名调用”实现，往往是将参数放到一个临时函数之中，再将这个临时函数传入函数体。这个临时函数就叫做 Thunk 函数。
```
function f(m) { 
  return m * 2; 
}
f(x + 5); 
// 等同于 
var thunk = function () { 
  return x + 5; 
};
function f(thunk) { 
  return thunk() * 2; 
}
```

### JavaScript 语言的 Thunk 函数
在 JavaScript 语言 中，Thunk 函数替换的不是表达式，而是多参数函数，将其替换成一个只接受回调函数作为参数的单参数函数。
```
// 正常版本的readFile（多参数版本） 
fs.readFile(fileName, callback); 

// Thunk版本的readFile（单参数版本） 
var Thunk = function (fileName) { 
  return function (callback) { 
    return fs.readFile(fileName, callback); 
  }; 
};
var readFileThunk = Thunk(fileName); 
readFileThunk(callback);
```

### Thunkify 模块
确保回调函数只运行一次。
```
function f(a, b, callback){ 
  var sum = a + b; 
  callback(sum); 
  callback(sum); 
}
var ft = thunkify(f); 
var print = console.log.bind(console); 
ft(1, 2)(print); 
// 3
```

### Generator函数的流程管理
Thunk 函数真正的威力，在于可以自动执行 Generator 函数。下面就是一个基于 Thunk 函数的 Generator 执行器。
```
function run(fn) { 
  var gen = fn(); 
  function next(err, data) { 
    var result = gen.next(data); 
    if (result.done) return; 
    result.value(next); 
  }
  next(); 
}
function* g() { 
  // ... 
}
run(g);
```

### co 模块
co 模块可以让你不用编写 Generator 函数的执行器。
```
var gen = function* () { 
  var f1 = yield readFile('/etc/fstab'); 
  var f2 = yield readFile('/etc/shells'); 
  console.log(f1.toString()); 
  console.log(f2.toString()); 
};
var co = require('co'); 
co(gen);
```

async函数
-------------
### 含义
它就是 Generator 函数的语法糖。async 函数对 Generator 函数的改进，体现在以下四点。
+ 内置执行器。
+ 更好的语义。
+ 更广的适用性。
+ 返回值是 Promise。

### 基本用法
当函数执行的时候，一旦遇到 await 就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。
```
async function getStockPriceByName(name) { 
  var symbol = await getStockSymbol(name); 
  var stockPrice = await getStockPrice(symbol); 
  return stockPrice; 
}
getStockPriceByName('goog').then(function (result) { 
  console.log(result); 
});
```
async 函数内部抛出错误，会导致返回的 Promise 对象变为 reject 状态。抛出的错误对象会被 catch 方法回调函数接收到。
```
async function f() { 
  throw new Error('出错了'); 
}
f().then( 
  v => console.log(v), 
  e => console.log(e) 
)
// Error: 出错了
```

### Promise 对象的状态变化
async 函数返回的 Promise 对象，必须等到内部所有 await 命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到 return 语句或者抛出错误。

### await 命令
正常情况下， await 命令后面是一个 Promise 对象。如果不是，会被转成一个即 resolve 的 Promise 对象。
```
async function f() { 
  return await 123; 
}
f().then(v => console.log(v)) 
// 123
```
只要一个 await 语句后面的 Promise 变为 reject ，那么整个 async 函数都会中断执行。
```
async function f() { 
  await Promise.reject('出错了'); 
  await Promise.resolve('hello world'); // 不会执行 
}
```

### 错误处理
+ 最好将 await 放在 try...catch 代码块之中。

```
async function myFunction() { 
  try { 
    await somethingThatReturnsAPromise(); 
  } catch (err) { 
    console.log(err);
  } 
}
```

+ 多个 await 命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。

```
let [foo, bar] = await Promise.all([getFoo(), getBar()]);
```

+ await 命令只能用在 async 函数之中，如果用在普通函数，就会报错。

```
async function dbFuc(db) { 
  let docs = [{}, {}, {}]; 
  // 报错 
  docs.forEach(function (doc) { 
    await db.post(doc); 
  }); 
}
```

### 异步遍历器
#### for await...of
```
async function () { 
  try { 
    for await (const x of createRejectingIterable()) { 
      console.log(x); 
    } 
  } catch (e) { 
    console.error(e); 
  } 
}
```

Class 的基本语法
-------------
### 基本概念
```
//定义类 
class Point { 
  constructor(x, y) { 
    this.x = x; 
    this.y = y; 
  }
  toString() { 
    return '(' + this.x + ', ' + this.y + ')'; 
  } 
}
```
类的所有方法都定义在类的 prototype 属性上面。
```
// 等同于 
Point.prototype = { 
  constructor() {}, 
  toString() {}, 
};
```
Object.assign 方法可以很方便地一次向类添加多个方法。
```
Object.assign(Point.prototype, { 
  toString(){}, 
  toValue(){} 
});
```
类的内部所有定义的方法，都是不可枚举的。

### constructor 方法
constructor 方法是类的默认方法，通过 new 命令生成对象实例时，自动调用该方法。一个类必须有 constructor 方法，如果没有显式定义，一个空的 constructor 方法会被默认添加。
```
class Point { }
// 等同于 
class Point { 
  constructor() {} 
}
```

### 类的实例对象
+ 生成类的实例对象的写法，与 ES5 完全一样，也是使用 new 命令。

```
class Point { // ... }
// 报错 
var point = Point(2, 3); 
// 正确 
var point = new Point(2, 3);
```
+ 与 ES5 一样，实例的属性除非显式定义在其本身（即定义在 this 对象上），否则都是定义在原型上（即定义在 class 上）。

```
//定义类 
class Point { 
  constructor(x, y) { 
    this.x = x; 
    this.y = y; 
  }
  toString() { 
    return '(' + this.x + ', ' + this.y + ')'; 
  } 
}
var point = new Point(2, 3); 
point.toString() // (2, 3) 
point.hasOwnProperty('x') // true 
point.hasOwnProperty('y') // true 
point.hasOwnProperty('toString') // false 
point.__proto__.hasOwnProperty('toString') // true
```
+ 与 ES5 一样，类的所有实例共享一个原型对象。

```
var p1 = new Point(2,3); 
var p2 = new Point(3,2); 
p1.__proto__ === p2.__proto__ 
//true
```

### Class 表达式
+ 与函数一样，类也可以使用表达式的形式定义。
+ 采用 Class 表达式，可以写出立即执行的 Class。

```
let person = new class { 
  constructor(name) { 
    this.name = name; 
  }
  sayName() { 
    console.log(this.name); 
  } 
}('张三'); 
person.sayName(); // "张三"
```

### 不存在变量提升
类不存在变量提升（hoist），这一点与 ES5 完全不同。

### 私有方法
#### 方法一
将私有方法移出模块,内部调用 call 进行绑定。
```
class Widget { 
  foo (baz) { 
    bar.call(this, baz); 
  }
  // ... 
}
function bar(baz) { 
  return this.snaf = baz; 
}
```

#### 方法二
利用 Symbol 值的唯一性，将私有方法的名字命名为一个 Symbol 值。
```
const bar = Symbol('bar'); 
const snaf = Symbol('snaf'); 
export default class myClass{ 
  // 公有方法 
  foo(baz) { 
    this[bar](baz); 
  }
  // 私有方法 
  [bar](baz) { 
    return this[snaf] = baz; 
  }
  // ... 
};
```

### 私有属性
在属性名之前，使用 # 表示。
```
class Point { 
  #x; 
  constructor(x = 0) { 
    #x = +x; 
    // 写成 this.#x 亦可 
  }
  get x() { 
    return #x 
  } 
  set x(value) { 
    #x = +value
  } 
}
```

### this 的指向
类的方法内部如果含有 this ，它默认指向类的实例。
```
class Logger { 
  printName(name = 'there') { 
    this.print(`Hello ${name}`); 
  }
  print(text) { 
    console.log(text); 
  } 
}
const logger = new Logger(); 
const { printName } = logger; 
printName(); 
// TypeError: Cannot read property 'print' of undef ined
```

### Class 的取值函数（getter）和存值函数（setter）
在“类”的内部可以使用 get 和 set 关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。
```
class MyClass { 
  constructor() { 
    // ... 
  }
  get prop() { 
    return 'getter'; 
  }
  set prop(value) { 
    console.log('setter: '+value); 
  } 
}
let inst = new MyClass(); 
inst.prop = 123; 
// setter: 123 
inst.prop 
// 'getter'
```

### Class 的 Generator 方法
如果某个方法之前加上星号（ * ），就表示该方法是一个 Generator 函数。
```
class Foo { 
  constructor(...args) { 
    this.args = args; 
  }
  * [Symbol.iterator]() { 
    for (let arg of this.args) { 
      yield arg; 
    } 
  } 
}

for (let x of new Foo('hello', 'world')) { 
  console.log(x); 
}
// hello 
// world
```

### Class 的静态方法
+ 如果在一个方法前，加上 static 关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

```
class Foo { 
  static classMethod() { 
    return 'hello'; 
  } 
}
Foo.classMethod() // 'hello' 
var foo = new Foo(); 
foo.classMethod() 
// TypeError: foo.classMethod is not a function
```

+ 如果静态方法包含 this 关键字，这个 this 指的是类，而不是实例。

```
class Foo { 
  static bar () { 
    this.baz(); 
  }
  static baz () { 
    console.log('hello'); 
  }
  baz () { 
    console.log('world'); 
  } 
}
Foo.bar() // hello
```

+ 父类的静态方法，可以被子类继承，也可以从 super 对象上调用。

```
class Foo { 
  static classMethod() { 
    return 'hello'; 
  } 
}

class Bar extends Foo { }
Bar.classMethod() // 'hello'

//或
class Bar extends Foo { 
  static classMethod() { 
    return super.classMethod() + ', too'; 
  } 
}
Bar.classMethod() // "hello, too"
```

### Class 的静态属性和实例属性
静态属性指的是 Class 本身的属性。
```
class Foo { }
Foo.prop = 1; 
Foo.prop // 1
```

#### 类的实例属性
类的实例属性可以用等式，写入类的定义之中。
```
class MyClass { 
  myProp = 42; 
  constructor() { 
    console.log(this.myProp); // 42 
  } 
}
```

#### 类的静态属性
类的静态属性只要在上面的实例属性写法前面，加上 static 关键字就可以了。
```
class MyClass { 
  static myStaticProp = 42; 
  constructor() { 
    console.log(MyClass.myStaticProp); // 42 
  } 
}
```

### new.target属性
+ 该属性一般用在构造函数之中，返回 new 命令作用于的那个构造函数。如果构造函数不是通过 new 命令调用的， new.target 会返回 undefined ，因此这个属性可以用来确定构造函数是怎么调用的。

```
function Person(name) { 
  if (new.target === Person) { 
    this.name = name; 
  } else { 
    throw new Error('必须使用 new 生成实例'); 
  } 
}
var person = new Person('张三'); // 正确 
var notAPerson = Person.call(person, '张三'); // 报错
```
+ 子类继承父类时， new.target 会返回子类。
+ 在函数外部，使用 new.target 会报错。

Class 的继承
-------------
### 简介
+ 子类必须在 constructor 方法中调用 super 方法，否则新建实例时会报错。这是因为子类没有自己的 this 对象，而是继承父类的 this 对象，然后对其进行加工。

```
class Point { /* ... */ } 
class ColorPoint extends Point { 
  constructor() { } 
}
let cp = new ColorPoint(); // ReferenceError
```

+ 如果子类没有定义 constructor 方法，这个方法会被默认添加。

```
class ColorPoint extends Point { }
// 等同于 
class ColorPoint extends Point { 
  constructor(...args) { 
    super(...args); 
  } 
}
```

### Object.getPrototypeOf()
Object.getPrototypeOf 方法可以用来从子类上获取父类。
```
Object.getPrototypeOf(ColorPoint) === Point 
// true
```

### super 关键字
super 这个关键字，既可以当作函数使用，也可以当作对象使用。
#### super 作为函数调用
super 作为函数调用时，代表父类的构造函数。
```
class A {} class B extends A { 
  constructor() { 
    super(); 
  } 
}
```
作为函数时， super() 只能用在子类的构造函数之中，用在其他地方就会报错。
```
class A {} class B extends A { 
  m() { 
    super(); 
    // 报错 
  } 
}
```

#### super 作为对象时
super 作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。
```
class A { 
  p() { 
    return 2; 
  } 
}
class B extends A { 
  constructor() { 
    super(); 
    console.log(super.p()); // 2 
  } 
}
let b = new B();
```
由于 super 指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过 super 调用的。
```
class A { 
  constructor() { 
    this.p = 2; 
  } 
}
class B extends A { 
  get m() { 
    return super.p; 
  } 
}
let b = new B(); 
b.m // undefined
```
通过 super 调用父类的方法时， super 会绑定子类的 this 。
```
class A { 
  constructor() { 
    this.x = 1; 
  }
  print() { 
    console.log(this.x); 
  } 
}
class B extends A { 
  constructor() { 
    super(); 
    this.x = 2; 
  }
  m() { 
    super.print(); 
  } 
}
let b = new B(); 
b.m() // 2
```

### 类的 prototype 属性和__proto__属性
+ 子类的 __proto__ 属性，表示构造函数的继承，总是指向父类。 
+ 子类 prototype 属性的 __proto__ 属性，表示方法的继承，总是指向父类的 prototype 属性。

```
class A { }
class B extends A { }
B.__proto__ === A // true 
B.prototype.__proto__ === A.prototype // true
```

### 原生构造函数的继承
ECMAScript 的原生构造函数大致有下面这些。
+ Boolean()
+ Number()
+ String() 
+ Array() 
+ Date() 
+ Function() 
+ RegExp() 
+ Error() 
+ Object()

ES6 允许继承原生构造函数定义子类，因为 ES6 是先新建父类的实例对象 this ，然后再用子类的构造函数修饰 this ，使得父类的所有行为都可以继承。下面是一个继承 Array 的例子。
```
class MyArray extends Array { 
  constructor(...args) { 
    super(...args); 
  } 
}
var arr = new MyArray(); 
arr[0] = 12; 
arr.length // 1 
arr.length = 0; 
arr[0] // undefined
```

### Mixin 模式的实现
Mixin 模式指的是，将多个类的接口“混入”（mixin）另一个类。
```
function mix(...mixins) { 
  class Mix {} 
  for (let mixin of mixins) { 
    copyProperties(Mix, mixin); 
    copyProperties(Mix.prototype, mixin.prototype); 
  }
  return Mix; 
}

function copyProperties(target, source) { 
  for (let key of Reflect.ownKeys(source)) { 
    if ( key !== "constructor" && key !== "prototype" && key !== "name" ) {
      let desc = Object.getOwnPropertyDescriptor(source, key); 
      Object.defineProperty(target, key, desc); 
    } 
  } 
}

//使用
class DistributedEdit extends mix(Loggable, Serializable) {}
```

Module 的语法
-------------
### 概述
ES6 模块不是对象，而是通过 export 命令显式指定输出的代码，再通过 import 命令输入。

ES6 模块还有以下好处。
+ 不再需要 UMD 模块格式了，将来服务器和浏览器都会支持 ES6 模块格式。目前，通过各种工具库，其实已经做到了这一点。 
+ 将来浏览器的新 API 就能用模块格式提供，不再必须做成全局变量或 者 navigator 对象的属性。 
+ 不再需要对象作为命名空间（比如 Math 对象），未来这些功能可以通过模块提供。

### export 命令
如果你希望外部能够读取模块内部的某个变量，就必须使用 export 关键字输出该变量。
```
// profile.js 
export var firstName = 'Michael'; 
export var lastName = 'Jackson'; 
export var year = 1958;

//or
var firstName = 'Michael'; 
var lastName = 'Jackson'; 
var year = 1958; 
export {firstName, lastName, year};
```
export 命令除了输出变量，还可以输出函数或类（class）。
```
export function multiply(x, y) { 
  return x * y; 
};
```
通常情况下， export 输出的变量就是本来的名字，但是可以使用 as 关键字重命名。
```
function v1() { ... } 
function v2() { ... } 
export { 
  v1 as streamV1, 
  v2 as streamV2, 
  v2 as streamLatestVersion 
};
```
export 语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。
```
export var foo = 'bar'; 
setTimeout(() => foo = 'baz', 500);
```

### import 命令
+ 如果想为输入的变量重新取一个名字， import 命令要使用 as 关键字，将输入的变量重命名。
+ import 命令接受一对大括号，里面指定要从其他模块导入的变量名。

```
import { lastName as surname } from './profile';
```
+ import 命令具有提升效果，会提升到整个模块的头部，首先执行。
+ 由于 import 是静态执行，所以不能使用表达式和变量

```
// 报错 
import { 'f' + 'oo' } from 'my_module'; 

// 报错 
let module = 'my_module'; 
import { foo } from module; 

// 报错 
if (x === 1) { 
  import { foo } from 'module1'; 
} else { 
  import { foo } from 'module2'; 
}
```
+ 如果多次重复执行同一句 import 语句，那么只会执行一次，而不会执行多次。
```
import 'lodash'; 
import 'lodash';
```

### 模块的整体加载
用星号（ * ）指定一个对象，所有输出值都加载在这个对象上面。
```
// circle.js 
export function area(radius) { 
  return Math.PI * radius * radius; 
}
export function circumference(radius) { 
  return 2 * Math.PI * radius; 
}

// main.js
import * as circle from './circle'; 
console.log('圆面积：' + circle.area(4)); 
console.log('圆周长：' + circle.circumference(14));
```

### export default 命令
+ 为模块指定默认输出，其他模块加载该模块时， import 命令可以为该匿名函数指定任意名字。

```
// export-default.js 
export default function () { 
  console.log('foo'); 
}
// import-default.js 
import customName from './export-default'; 
customName(); // 'foo'
```
+ 一个模块只能有一个默认输出，因此 export default 命令只能使用一次。
+ export default 命令其实只是输出一个叫做 default 的变量，所以它后面不能跟变量声明语句。

### export 与 import 的复合写法
如果在一个模块之中，先输入后输出同一个模块， import 语句可以与 export 语句写在一起。
```
export { foo, bar } from 'my_module';
// 接口改名 
export { foo as myFoo } from 'my_module'; 
// 整体输出 
export * from 'my_module';
```

### 模块的继承
```
// circleplus.js
export * from 'circle'; 
export var e = 2.71828182846; 
export default function(x) { 
  return Math.exp(x); 
}

// main.js 
import * as math from 'circleplus'; 
import exp from 'circleplus'; 
console.log(exp(math.e));
```

### 跨模块常量
如果想设置跨模块的常量（即跨多个文件），或者说一个值要被多个模块共享，可以采用下面的写法。
```
// constants.js 模块 
export const A = 1; 
export const B = 3; 
export const C = 4; 

// test1.js 模块 
import * as constants from './constants'; 
console.log(constants.A); // 1 
console.log(constants.B); // 3
```

### import()
#### 简介
+ require 是运行时加载模块， require 到底加载哪一个模块，只有运行时才知道。 import 语句做不到这一点。
+ import() 函数，完成动态加载。
+ import() 类似于 Node 的 require 方法，区别主要是前者是异步加载，后者是 同步加载。
+ import() 返回一个 Promise 对象。

```
const main = document.querySelector('main'); 
import(`./section-modules/${someVariable}.js`) 
  .then(module => { 
    module.loadPageInto(main); 
  })
  .catch(err => { 
    main.textContent = err.message; 
  });
```

#### 适用场合
+ 按需加载: import() 可以在需要的时候，再加载某个模块。
+ 条件加载：import() 可以放在 if 代码块，根据不同的情况，加载不同的模块。
+ 动态的模块路径：import() 允许模块路径动态生成。

Module 的加载实现
-------------
### 加载规则
```
<script type="module" src="foo.js"></script>
```

### ES6 模块与 CommonJS 模块的差异
+ CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
+ CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

### Node 加载
+ 在 Node 环境中，使用 import 命令加载 CommonJS 模块，Node 会自动将 module.exports 属性，当作模块的默认输出，即等同于 export default 。

```
// a.js 
module.exports = { 
  foo: 'hello', 
  bar: 'world' 
};
// 等同于 
export default { 
  foo: 'hello', 
  bar: 'world' 
};
```
+ 采用 require 命令加载 ES6 模块时，ES6 模块的所有输出接口，会成为输入对象的属性。

```
// es.js 
export let foo = {bar:'my-default'}; 
export {foo as bar}; 
export function f() {}; 
export class c {}; 

// cjs.js 
const es_namespace = require('./es'); 
// es_namespace = { 
//   get foo() {return foo;} 
//   get bar() {return foo;} 
//   get f() {return f;} 
//   get c() {return c;} 
// }
```

### 循环加载
“循环加载”（circular dependency）指的是， a 脚本的执行依赖 b 脚本，而 b 脚本的执行又依赖 a 脚本。
#### CommonJS
CommonJS 模块的重要特性是加载时执行，即脚本代码在 require 的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。
```
// a.js
exports.done = false; 
var b = require('./b.js'); 
console.log('在 a.js 之中，b.done = %j', b.done); 
exports.done = true; 
console.log('a.js 执行完毕');

// b.js
exports.done = false; 
var a = require('./a.js'); 
console.log('在 b.js 之中，a.done = %j', a.done); 
exports.done = true; 
console.log('b.js 执行完毕');

// main.js
var a = require('./a.js'); 
var b = require('./b.js'); 
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.do ne);

$ node main.js 

在 b.js 之中，a.done = false 
b.js 执行完毕 
在 a.js 之中，b.done = true 
a.js 执行完毕 
在 main.js 之中, a.done=true, b.done=true
```

#### ES6 模块
ES6加载的变量，都是动态引用 其所在的模块。只要引用存在，代码就能执行。
```
// even.js
import { odd } from './odd' 
export var counter = 0; 
export function even(n) { 
  counter++; 
  return n == 0 || odd(n - 1); 
}
// odd.js 
import { even } from './even'; 
export function odd(n) { 
  return n != 0 && even(n - 1); 
}

$ babel-node > import * as m from './even.js'; 
> m.even(10); 
true 
> m.counter 
6
> m.even(20) 
true 
> m.counter 
17
```

### ES6模块的转码
它是一个垫片库（polyfill），可以在浏览器内加载 ES6 模块、AMD 模块和 CommonJS 模块，将其转为 ES5 格式。
```
<script src="system.js"></script>
<script> 
  System.import('./app.js'); 
</script>
```