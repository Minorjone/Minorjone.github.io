---
title: JS事件冒泡和事件委托
date: 2019-12-15 17:25:59
tags: [前端,JS]
categories: 前端
keywords: [前端,JS,事件,事件冒泡，事件委托]
description: 
---

最近在学习React的时候，看到了React实现合成事件的底层机制，事件委托一词赫然出现在眼前，于是我不由的想到在不久前的一次面试中，面试官曾让我写一个冒泡阶段的事件委托，今天借着对React的理解，再来深入探讨一下JS中的事件冒泡和事件委托。

<!--more-->

事件流
-------------
“DOM2级事件”规定的事件流包括三个阶段：事件捕获阶段，处于目标阶段和事件冒泡阶段。
![Alt text][01]
[01]: ../assets/20191215/1.png

|    阶段    |       描述       |    传播方向    |
|:--------:|:-------------|:-------:|
|   事件捕获  |     不太具体的节点应该更早接收到事件，而具体的节点应该最后接收到事件    |   DOM树依次向下  |
|   事件冒泡  |     由最具体的元素开始接收，逐级向上传播到较为不具体的节点    |   DOM树向上  |

在DOM2级事件处理程序操作 `addEventListener()` 和 `removeEventListener()` 方法中，其第三个参数接收一个布尔值，用于指定事件处理程序在何时进行。
+ 值为true：表示在捕获阶段调用事件处理程序
+ 值为false：表示在冒泡阶段调用事件处理程序

这里是我们处在level菜鸟阶段最先接触到有关事件冒泡和事件捕获的方法了，但是由于事件捕获在浏览器的兼容情况并不乐观，所以大多数情况我们不建议在事件捕获阶段注册事件处理程序。

事件对象
-------------
在正式开始介绍事件委托之前，我们先来了解一下事件对象。
> 触发DOM上的某个事件时，会产生一个事件对象event，这个对象中包含着所有与事件有关的信息。

这是红宝书里对事件对象的解释，下表列出了事件对象的部分属性和方法 。

|    属性 / 方法    |    类型    |    读 / 写    |    说明    |
|:--------:|:---------:|:-------:|:-------------|
|   type  |   String   |   只读  |   被触发的事件的类型  |
|   target  |   Element   |   只读  |   事件的目标  |
|   currentTarget  |   Element   |   只读  |   其事件处理程序当前正在处理事件的那个元素  |
|   preventDefault()  |   Function   |   只读  |   取消事件的默认行为  |
|   stopPropagation  |   Function   |   只读  |   取消事件的进一步捕获或冒泡  |
|   stopImmediatePropagation  |   Function   |   只读  |   取消事件的进一步捕获或冒泡，同时阻止任何事件处理程序被调用  |

### target 和 currentTarget
在事件处理程序内部，对象this始终等于currentTarget的值，而target则只包含事件的实际目标。

+ 当事件处理程序直接指定给了目标元素，this、currentTarget、target包含相同的值。

```
var btn = document.getElementById("myBtn");
btn.onclick = function(e) {
  console.log(e.currentTarget === this); //true
  console.log(e.target === this); //true
}
```

+ 当事件处理程序存在于按钮的父节点中，这三者的值不相同。

```
document.body.onclick = function(e) {
  console.log(this === document.body); //true
  console.log(e.currentTarget === document.body); //true
  console.log(e.target === document.getElementById("myBtn")); //true;
}
```

上述例子中，click事件的真正目标是target的按钮元素，但由于没有在按钮元素上注册事件处理程序，所以click事件一只冒泡到了document.body后才被处理。

由上，我们可以得出更通俗的对target 和 currentTarget的解释。
+ target：当前被点击的实际元素
+ currentTarget：注册了事件处理程序的元素

### peventDefault()
```
var link = document.getElementById("myLink");
link.onclick = function(e) {
  e.preventDefault();
}
```
上述例子阻止了链接默认跳转到其href指定的URL。

#### FastClick原理
说到阻止事件默认行为，不得不提移动端常用的FastClick。其实现原理是这样的：
+ 注册业务onclick事件
+ 在touchend阻止默认事件（屏蔽之后的click事件）
+ 合成一个click事件，并立即执行
+ 执行业务自己的click事件

下面可以看一个模拟FastClick的代码案例
```
  // 业务代码
  var $test = document.getElementById('test')
  $test.addEventListener('click', function () {
    console.log('1 click')
  })

  // FastClick简单实现
  var targetElement = null
  document.body.addEventListener('touchstart', function () {
    // 记录点击的元素
    targetElement = event.target
  })
  document.body.addEventListener('touchend', function (event) {
    // 阻止默认事件（屏蔽之后的click事件）
    event.preventDefault()
    var touch = event.changedTouches[0]
    // 合成click事件，并添加可跟踪属性forwardedTouchEvent
    var clickEvent = document.createEvent('MouseEvents')
    clickEvent.initMouseEvent('click', true, true, window, 1, touch.screenX, touch.screenY, touch.clientX, touch.clientY, false, false, false, false, 0, null)
    clickEvent.forwardedTouchEvent = true // 自定义的
    targetElement.dispatchEvent(clickEvent)
  })
```

### stopPropagation 和 stopImmediatePropagation
```
var btn = document.getElementById("myBtn");
btn.onclick = function(e) {
  alert("Clicked");
  e.stopPropagation();
}
document.body.onclick = function(e) {
  alert("Body clicked");
}
```
上述例子，在点击事件在目标元素上被执行后就阻止了冒泡，防止其传播到body。

```
var btn = document.getElementById("myBtn");
btn.addEventListener("click", function(e) {
  alert("Clicked before");
  e.stopImmediatePropagation();
}, false);
btn.addEventListener("click", function(e) {
  alert("Clicked after");
}, false);
document.body.addEventListener("click", function(e) {
  alert("Body clicked");
}, false);
```
在上述例子中，使用了stopImmediatePropagation阻止冒泡，绑定在该按钮上的同类事件，没有执行完的也将被阻止。

事件委托
-------------
事件委托利用了事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。例如，click事件会一直冒泡到document层次，也就是是说我们可以为整个页面指定一个onclick事件处理程序，而不必给每个可单击元素分别添加事件处理程序。
现在我们回到那个面试题，来写一个事件委托的方法。
```
/**
 * @param type：String，事件类型
 * @param element：DOM element，目标元素
 * @param callback：Function，业务事件处理程序
*/ 
function eventHandler(type, element, callback) {
  document.addEventListener(type, function(e){
    if(e.target === element) {
      callback.call(this);
    }
  }, false);
}
```

### React 合成事件的实现原理
React 合成事件是把所有事件绑定到结构的最外层，使用一个统一的事件监听器，以队列的方式，从触发事件的组件向父组件回溯，调用它们在JSX中声明的callback。
其中涉及到几个重要的类如下：
+ ReactEventListener：负责事件注册和事件分发。React将DOM事件全都注册到document这个节点上。事件分发主要调用dispatchEvent进行，从事件触发组件开始，向父元素遍历。
+ ReactEventEmitter：负责每个组件上事件的执行。
+ EventPluginHub：负责事件的存储，合成事件以对象池的方式实现创建和销毁，大大提高了性能。
+ SimpleEventPlugin等plugin：根据不同的事件类型，构造不同的合成事件。如focus对应的React合成事件为SyntheticFocusEvent

#### 事件注册
组件创建和更新的入口方法mountComponent和updateComponent都会调用_updateDOMProperties方法。
```
_updateDOMProperties: function (lastProps, nextProps, transaction) {
    ... // 前面代码太长，省略一部分
    else if (registrationNameModules.hasOwnProperty(propKey)) {
        // 如果是props这个对象直接声明的属性，而不是从原型链中继承而来的，则处理它
        // nextProp表示要创建或者更新的属性，而lastProp则表示上一次的属性
        // 对于mountComponent，lastProp为null。updateComponent二者都不为null。unmountComponent则nextProp为null
        if (nextProp) {
          // mountComponent和updateComponent中，enqueuePutListener注册事件
          enqueuePutListener(this, propKey, nextProp, transaction);
        } else if (lastProp) {
          // unmountComponent中，删除注册的listener，防止内存泄漏
          deleteListener(this, propKey);
        }
    }
}
```
下面我们来看enqueuePutListener，它负责注册JSX中声明的事件。源码如下
```
/ inst: React Component对象
// registrationName: React合成事件名，如onClick
// listener: React事件回调方法，如onClick=callback中的callback
// transaction: mountComponent或updateComponent所处的事务流中，React都是基于事务流的
function enqueuePutListener(inst, registrationName, listener, transaction) {
  if (transaction instanceof ReactServerRenderingTransaction) {
    return;
  }
  var containerInfo = inst._hostContainerInfo;
  var isDocumentFragment = containerInfo._node && containerInfo._node.nodeType === DOC_FRAGMENT_TYPE;
  // 找到document
  var doc = isDocumentFragment ? containerInfo._node : containerInfo._ownerDocument;
  // 注册事件，将事件注册到document上
  listenTo(registrationName, doc);
  // 存储事件,放入事务队列中
  transaction.getReactMountReady().enqueue(putListener, {
    inst: inst,
    registrationName: registrationName,
    listener: listener
  });
}
```
enqueuePutListener方法主要做了两件事，一是将JSX声明的事件注册到document元素上，一是将事件存储放入事务队列，以供事件触发时回调。
listenTo代码虽然比较长，但逻辑很简单，调用trapCapturedEvent和trapBubbledEvent来注册捕获和冒泡事件。
```
trapBubbledEvent: function (topLevelType, handlerBaseName, element) {
    if (!element) {
      return null;
    }
    return EventListener.listen(
      element,   // 绑定到的DOM目标,也就是document
      handlerBaseName,   // eventType
      ReactEventListener.dispatchEvent.bind(null, topLevelType));  // callback, document上的原生事件触发后回调
  },

  listen: function listen(target, eventType, callback) {
    if (target.addEventListener) {
      // 将原生事件添加到target这个dom上,也就是document上。
      // 这就是只有document这个DOM节点上有原生事件的原因
      target.addEventListener(eventType, callback, false);
      return {
        // 删除事件,这个由React自己回调,不需要调用者来销毁。但仅仅对于React合成事件才行
        remove: function remove() {
          target.removeEventListener(eventType, callback, false);
        }
      };
    } else if (target.attachEvent) {
      // attach和detach的方式
      target.attachEvent('on' + eventType, callback);
      return {
        remove: function remove() {
          target.detachEvent('on' + eventType, callback);
        }
      };
    }
  }
```
在listen方法中，我们终于发现了熟悉的addEventListener这个原生事件注册方法。只有document节点才会调用这个方法，故仅仅只有document节点上才有DOM事件。这大大简化了DOM事件逻辑，也节约了内存。

有关事件存储的过程我们就不是我们本文的重点，就跳过，有兴趣的可以去看一下参考链接中的文章。

#### 事件执行
在trapBubbledEvent中，我们看到listen最后执行的回调函数是dispatchEvent，这个方法就是React执行事件分发的入口。
```
// topLevelType：带top的事件名，如topClick。不用纠结为什么带一个top字段，知道它是事件名就OK了
// nativeEvent: 用户触发click等事件时，浏览器传递的原生事件
dispatchEvent: function (topLevelType, nativeEvent) {
    // disable了则直接不回调相关方法
    if (!ReactEventListener._enabled) {
      return;
    }

    var bookKeeping = TopLevelCallbackBookKeeping.getPooled(topLevelType, nativeEvent);
    try {
      // 放入批处理队列中,React事件流也是一个消息队列的方式
      ReactUpdates.batchedUpdates(handleTopLevelImpl, bookKeeping);
    } finally {
      TopLevelCallbackBookKeeping.release(bookKeeping);
    }
}
```
可见我们仍然使用批处理的方式进行事件分发，handleTopLevelImpl才是事件分发的真正执行者，它是事件分发的核心，体现了React事件分发的特点。
```
// document进行事件分发,这样具体的React组件才能得到响应。因为DOM事件是绑定到document上的
function handleTopLevelImpl(bookKeeping) {
  // 找到事件触发的DOM和React Component
  var nativeEventTarget = getEventTarget(bookKeeping.nativeEvent);
  var targetInst = ReactDOMComponentTree.getClosestInstanceFromNode(nativeEventTarget);

  // 执行事件回调前,先由当前组件向上遍历它的所有父组件。得到ancestors这个数组。
  // 因为事件回调中可能会改变Virtual DOM结构,所以要先遍历好组件层级
  var ancestor = targetInst;
  do {
    bookKeeping.ancestors.push(ancestor);
    ancestor = ancestor && findParent(ancestor);
  } while (ancestor);

  // 从当前组件向父组件遍历,依次执行注册的回调方法. 我们遍历构造ancestors数组时,是从当前组件向父组件回溯的,故此处事件回调也是这个顺序
  // 这个顺序就是冒泡的顺序,并且我们发现不能通过stopPropagation来阻止'冒泡'。
  for (var i = 0; i < bookKeeping.ancestors.length; i++) {
    targetInst = bookKeeping.ancestors[i];
    ReactEventListener._handleTopLevel(bookKeeping.topLevelType, targetInst, bookKeeping.nativeEvent, getEventTarget(bookKeeping.nativeEvent));
  }
}
```
事件处理由_handleTopLevel完成。
```
// React事件调用的入口。DOM事件绑定在了document原生对象上,每次事件触发,都会调用到handleTopLevel
handleTopLevel: function (topLevelType, targetInst, nativeEvent, nativeEventTarget) {
  // 采用对象池的方式构造出合成事件。不同的eventType的合成事件可能不同
  var events = EventPluginHub.extractEvents(topLevelType, targetInst, nativeEvent, nativeEventTarget);
  // 批处理队列中的events
  runEventQueueInBatch(events);
}
```
React的合成事件机制还是比较复杂的，这里只是部分，但是也可是大致感受到它从自动委托到分发执行的过程，希望对大家理解有所帮助。

> 参考链接：[React源码分析7 — React合成事件系统](https://blog.csdn.net/u013510838/article/details/612247609)