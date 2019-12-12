---
title: JS函数表达式（闭包）
date: 2019-05-27 22:27:49
tags: 前端
categories: 前端
keywords: [前端,JS,闭包]
description: 
---

基础
-------------
### 函数声明和函数表达式
函数声明：重要特征是函数声明提升，在执行代码之前会先读取函数声明。
```
sayHi();
function sayHi(){
    alert("Hi");
}
```

<!--more-->

函数表达式（匿名函数）：匿名函数的name属性是空字符串，有时也称函数拉姆达函数。
```
var sayHi = function(){
    alert("Hi");
};
```

递归
-------------
使用arguments.callee指向正在执行的函数指针，可以实现对函数的递归调用，且比直接使用函数名更加安全。
```
function facorial(num){
    if(num <= 1){
        return 1;
    } else {
        return num * arguments.callee(num-1); //使之与函数名解耦合
    }
}
```
以上方法在严格模式下会出错，在严格模式下，可使用命名函数表达式达成相同的结果。
```
var factorial = (function f(num){
    if(num <= 1){
        return 1;
    } else {
        return num * f(num-1);
    }
});
```

闭包
-------------
### 基础
闭包是指有权访问另一个函数作用域中的变量的函数。
```
function createComparesonFuncion(popertyName){
    return function(object1,object2){
        var value1 = object1[propertyName];
        var value2 = object2[propertyName];
        if(value1 < value2){
            return -1;
        } else if(value1 > value2){
            return 1;
        } else {
            return 0;
        }
    };
}
```
一般当函数执行完毕后，局部活动对象就会被销毁，内存中仅保存全局作用域，但闭包不同，在另一个函数内部定义的函数会将包含函数（即外部函数）的活动对象加到它的作用域链中。
```
\\创建函数
var compareName = createComparesonFunction("name");
var result = compareNames({name:"Nicholas"},{name:"Greg"});  //调用函数
compareNames = null;  //解除对匿名函数的引用
```

![Alt text][01]
[01]: ../assets/20190527/pic1.png

### 闭包与变量
闭包只能取得包含函数中任何变量的最后一个值，因为闭包保存的是整个变量对象，而不是某个特殊变量。
```
function createFunctions(){
    var result = new Array();
    for(var i = 0; i < 10; i++){
        result[i] = function() {
            return i;   //每次返回都为10
        };
    }
    return result;
}
```
以上代码，通过匿名函数强制让闭包行为符合预期。
```
function createFunctions(){
    var result = new Array();
    for(var i = 0; i < 10; i++){
        result[i] = function(num) {   //使用匿名函数，则立即执行匿名函数
            return function(num){
                return num;
            };   
        }(i);   //值传递，把i赋值给num
    }
    return result;
}
```

### this对象
匿名函数的执行环节具有全局性，因此其this对象通常指向window。
```
var name = "The window";
var object = {
    name : "MyObject",
    getNameFunc ： function(){
        return function(){
            return this.name;
        }
    }
};
alert(object.getNameFunc()());   //"The window"
```
每个函数在调用时都会自动取得两个特殊变量`this`和`arguments`内部函数只会搜索到其活动对象为止，所以不能直接访问外部函数中的这两个变量。


可以通过把外部作用域中的this对象保存在闭包能访问到的变量里，让闭包访问。
```
var name = "The window";
var object = {
    name : "MyObject";
    getNameFunc : function(){
        var that = this;
        return function(){
            return that.name;
        };
    }
};
alert(object.getNameFunc()());  //"MyObject"
```
特殊情况，this值可能会意外地改变。
```
var name = "The window";
var object = {
    name : "MyObject";
    getName : function(){
        treturn this.name;
    }
};
object.getName();   //"MyObject"
(object.getName)();   //"MyObject",相当于引用了一个函数，this得到了维持
(object.getName = object.getName)();   //"The window"，this值不能得到维持
```

### 内存泄漏
```
//该闭包导致element的引用数无法减少，element不会被回收
function assignHandler(){
    var element = document.getElementById("someElement");
    element.onlick = function(){
        alert(element.id);    
    }
}
```

### 模仿块级作用域
JavaScript没有块级作用域的概念
```
fucntion outputNumbers(count){
    for(var i = 0; i < count; i++){
        alert(i);
    }
    var i;    //重新声明变量也不会改变它的值
    alert(i);  //count值，从i定义开始，就可以在函数内部随处访问
    //而在Java，c++中，变量i只会在for循环的语句块中有定义
}
```
可以用匿名函数来模仿块级作用域
```
(function(){
    //这里是块级作用域
})();   //定义并立即调用了一个匿名函数，括号表示它实际上是一个函数表达式
var someFunction = function(){
    //这里是块级作用域
};
someFunction();
```
JavaScript将function当作一个函数声明的开始，而函数声明后面不能跟圆括号，而函数表达式可以。
```
function outputNumber(count){
    (function(){
        for(var i = 0; i < count; i++){
            alert(i);
        }
    })();
    alert(i);   //出错
}
```
私有作用域的i在执行结束后被销毁，私有作用域能访问count，因为其为闭包

### 私有变量
```
function MyObject(){
    //私有变量和私有函数
    var privateVariable = 10;
    function privateFunction(){
        return false;
    }
    //特权方法
    this.publicMethod = function(){
        privateVariable ++;
        return privateFunction();
    }
}
```
利用私有和特权成员，可以隐藏那些不应该被直接修改的数据
```
funcion Person(name){
    this.getName = function(){
        return name;
    };
    this.setName = funcion(value){
        name = value;
    };
}
var person = new Person("Nicholas");
alert(person.getName());   //"Nicholas"
person.setName("Greg");
alert(person.getName());  //"Greg"
```
在构造函数外部没有办法访问name

### 静态私有变量
```
(function(){
    //私有变量和私有函数
    var privateVar = 10;
    function privateFunc(){
        return false;
    }
    //构造函数
    MyObject = function(){
    };   //没有使用var，所以声明的是全局的类
    //公有/特权方法
    MyObject.prototype.publicMethod = function(){
        privateVar ++;
        return privateFunc();
    };
})();
```
该模式与构造函数定义区别在于，其私有变量和函数是由实例共享的
```
(function(){
    var name = "";
    Person = function(value){
        name = value;
    };
    Person.prototype.getName = function(){
        return name;
    };
    Person.prototype.setName = function(value){
        name = value;
    };
})();
var person2 = new Person("Van");
alert(person1.getName());  //"Van"
person1.setName("Greg");
alert(person1.getName());  //"Greg"
var person2 = new Person("John");
alert(person1.getName());  //"John"
alert(person2.getName());  //"John"
//name为共享属性
```

### 模块模式
```
var singleton = function(){
    var privateVar = 10;
    function privateFun(){
        return false;
    }
    //特权/公有方法和属性
    return{
        publicProperty : true,
        publicMethod ：function(){
            privateVar++;
            return privateFunc();
        }  //对象字面量是单例的公共接口
    };
}();
```

### 增强的模块模式
增强模块模式适合那些单例必须是某种类型的实例，同时还必须增加某些属性和方法对其加以增强的情况。
```
var singleton = function(){
    var privateVar = 10;
    function privateFun(){
        return false;
    }
    var object = new CustomType();
    object.publicProperty = true;
    object.publicMethod = function(){
        privateVar++;
        return privateFun();
    };
    return object;
}();
```