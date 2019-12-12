---
title: JS引用类型
date: 2019-06-17 21:35:13
tags: 前端
categories: 前端
keywords: [前端,JS,引用类型]
description: 
---

定义
-------------
引用类型的值（对象）是引用类型的一个实例。
引用类型是一种数据结构，用于将数据和功能组织在一起。

Object类型
-------------
### 创建方式：
使用new操作符后跟Object构造函数。
```
var person = new Object();   //与 var person = {} 等同
person.name = "Nicholas";
person.age = 29;
```

<!--more-->

使用对象字面量表示法。
```
var person = {
    name : "Nicholas",
    age : 29  //在最后一个属性后面添加逗号，会在IE7及更早版本的Opera中导致错误
}
```

### 访问属性：
使用点表示法：
```
alert(person.name);
```
使用方括号语法：
```
alert(person["name"]);
```
使用方括号访问的优点：
+ 可以通过变量来访问
+ 如果属性名中包含会导致错误语法的字符，或使用关键字，保留字，可以用方括号

除非必须使用变量来访问属性，否则我们建议使用点表示法。

Array类型
-------------
### 特点：
ECMAScript数组的每一项可以保存在任何类型的数据
ECMAScript数组的大小是可以动态调整的

### 创建方式：
使用Array构造函数
```
var colors = new Array(3);
var colors = new Array("Greg");
```
使用数组字面量
```
var colors = ["red","blue","green"];
```

### 读取方式和设置数组：
```
var colors = ["red","blue","green"];
alert(colors[[0]]);
colors[2] = "black";
```

### 检测数组：
```
if(Array.isArray(value)){}
```

### 转换方法：
```
var colors = ["red","blue","green"];
alert(colors.toString());   //red,blue,green
alert(colors.valueOf());    //red,blue,green
alert(colors);              //red,blue,green
alert(colors.join(","));    //red,blue,green
alert(colors.join("||"));   //red||blue||green
```

### 栈方法：
栈是一种LIFO(Last-In-First-Out)的数据结构
```
var colors = new Array();
var count = colors.push("red","green");
alert(count);   //2
count = colors.push("black");
alert(count);   //3
var item = colors.pop();
alert(item);    //"black"
```

### 队列方法：
队列是一种FIFO(First-in-First-Out)的数据结构
```
var colors = new Array();
var count = colors.push("red","green");
alert(count);     //2
count = colors.push("black");
alert(count);     //3
var item = colors.shift();
alert(item);      //red
var colors = new Array();
var count = colors.unshift("red","green");
count = colors.unshift("black");
alert(colors);    //black,red,green
var item = colors.pop();
alert(item);      //green
```

### 重排序方法：
#### function reverse:
```
var values= [1,2,3,4,5];
values.reverse();
alert(values);   //5,4,3,2,1
```

#### function sort:
在执行sort方法前，会先调用toString()转型，然后比较得到字符串
```
var values = [0,1,5,10,15];
values.sort();
alert(values);   //0,1,10,15,5
```
添加比较函数，如果第一个参数应该位于第二个参数前，返回负数，否则返回正数。
```
function compare(value1,value2){
    if(value1 < value2){
        return -1;
    }
    else if(value1 > value2){
        return 1;
    }
    else{
        return 0;
    }
}
values.sort(compare);  //0,1,5,10,15
```
数值类型可以用更简单的比较函数
```
function compare(value1,value2){   //升序
    return value1 - value2;
}
```

### 操作方法：
#### function concat:
```
var colors = ["red","green","blue"];
var colors = colors.concat("yellow",["black","brown"]);
alert(colors);    //red,green,blue
alert(colors2);   //red,green,blue,yellow,black,brown
```
这个方法会先创建当前数组的一个副本，然后将接收的参数添加到这个副本的末尾，最后返回新构建的数组

#### function slice:
```
var colors = ["red","green","blue","yellow","purple"];
var colors2 = colors.slice(1);
var colors3 = colors.slice(1,4);
alert(colors2);   //green,blue,yellow,purple
alert(colors3);   //green,blue,yellow
```
能够基于当前数组中的一或多个项创建一个新数组，slice方法不会影响原始数组

#### function splice:
删除、插入、替换，用于向数组中部插入项
```
var colors = ["red","green","blue"];
var removed = colors.splice(0,1);   //删除一项
alert(colors);    //green,blue
alert(removed);   //red
removed = colors.splice(1,0,"yellow","orange");   //插入两项
alert(colors);    //green,yellow,orange,blue
alert(removed);   //空数组
removed = colors.splice(1,1,"red","purple");   //替换一项，插入一项
alert(colors);   //green,red,purple,orange,blue
alert(removed);  //yellow
```

### 位置方法：
#### function indexOf:
从数组的开头开始向后查找，比较时会使用全等操作符
```
var numbers = [1,2,3,4,5,4,3,2,1];
alert(numbers.indexoOf(4));   //3
alert(numbers.indexOf(4,4))   //第一参数表示要查找的项，第二参数表示查找起始位置
var person = {name:"Nicholas"};
var people = [{name:"Nicholas"}];
var morePeople = [person];
alert(people.indexOf(person));   //-1
alert(morePeople.indexOf(person));  //0
```

#### function lastIndexOf:
从数组的末尾开始向前查找
```
var numbers = [1,2,3,4,5,4,3,2,1];
alert(numbers.lastIndexOf(4));   //5
alert(numbers.lastIndexOf(4,4));   //3
```

### 迭代方法：
#### function every:
对数组中的每一项运行给定函数，如果该函数对每一项都返回true，则返回true
```
var numbers = [1,2,3,4,5,4,3,2,1];
var everyResult = numbers.every(function(item,index,array){
    return (item>2);
});
alert(everyResult);  //false
```


#### function filter:
对数组中的每一项运行给定函数，返回该函数会返回true的项组成的数组
```
var numbers = [1,2,3,4,5,4,3,2,1];
var filterResult = numbers.filter(function(item,index,array){
    return (item>2);
});
alert(filterResult);  //[3,4,5,4,3]
```

#### function forEach:
对数组中的每一项运行给定函数，这个方法没有返回值
```
var numbers = [1,2,3,4,5,4,3,2,1];
var filterResult = numbers.filter(function(item,index,array){
    //...
});
```

#### function map:
对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组
```
var numbers = [1,2,3,4,5,4,3,2,1];
var mapResult = numbers.map(function(item,index,array){
    return item*2;
});
alert(mapResult);  //[2,4,6,8,10,8,6,4,2]
```

#### function some:
对数组中的每一项运行给定函数，如果该函数对任一项返回true，则返回true
```
var numbers = [1,2,3,4,5,4,3,2,1];
var someResult = numbers.some(function(item,index,array){
    return (item>2);
});
alert(someResult);  //true
```

### 缩小方法（归并方法）：
#### function reduce:
接收4个参数：前一个值，当前值，项的索引值和数组对象，这个函数中返回的任何值都会作为第一个参数自动传给下一项
```
var values = [1,2,3,4,5];
var sum - values.reduce(function(prev,cur,index,array){
    return prev + cur;
});
alert(sum);   //15
```

#### function reduceRight:
与reduce相似，只是从数组最后一项开始，向前遍历
```
var values = [1,2,3,4,5];
var sum = values.reduceRight(function(prev,cur,index,array){
    return prev + cur;
});
alert(sum);   //15
```

Date类型
-------------
### 构造函数：
```
var nom = new Date();
var now = new Date.parse("YYYY-MM-DD");   //2019-02-19
var now = new Date(Date.UTC(2015,4,5,17,55,55));  //2015/4/5 17:55:55
var start = Date.now();  //获取毫秒数
```

### 继承方法：
不同浏览器对toLocaleString(),toString()和valueOf()方法解释不同

### 日期格式化方法：
toDateString()：以特定于实现的格式显示星期几、月、日和年
toTimeString()：以特定于实现的格式显示时分秒和时区
toLocaleDateString()：以特定于地区的格式显示星期几、日、月、年
toLocaleTimeString()：以特定于地区的格式显示时分秒
toUTCString()：以特定于实现的格式完整的UTC日期

### 日期/时间组件方法：
getTime()：返回表示日期的毫秒数
getFullyYear()：取得4位数的年份
getMonth()：返回日期中的月份，0-11表示1月至12月
getDate()：返回日期月份中的天数(1-31)
getDay()：返回日期中星期的星期几(0-6)表示，星期日至星期六
getHours()：返回日期中的小时数(0-23)

RegExp类型
-------------
### 基础语法：
```
var expression = /pattern/flags;
```
pattern部分：可以是任何简单或复杂的正则表达式
flags部分：每个正则表达式都可带有1或多个标志
> g：表示全局模式，即模式将被应用于所有字符串
> i：标识区分大小写模式，即在确定匹配项时忽略模式与字符串的大小写
> m：表示多行模式，即在到达一行文本末尾时还会继续查找下一行是否存在与模式匹配的项

与其他语言中的正则表达式类似，模式中使用的所有元字符都必须转义:([(\^$|)?*+.])
```
var pattern = /[bc]at/i;
var pattern2 = new RegExp("[bc]at","i");
```
在ECMAScript中，正则表达式字面量始终会共享同一个RegExp实例，而使用构造函数创建的每一个新RegExp实例都是一个新实例

### RegExp实例属性：
global：布尔值，表示是否设置了g标志
ignoreCase：布尔值，表示是否设置了i标志
lastIndex：整数，表示开始搜索下一个匹配项的字符位置，从0算起
multiline：布尔值，表示是否设置了m标志
source：正则表达式的字符串表示，按照字面量形式而传入构造函数中的字符串模式返回
```
var pattern1 = /\[bc\]at/i;
alert(pattern1.global);   //false
alert(pattern1.ignoreCase);   //true
alert(pattern1.multiline);   //false
alert(pattern1.lastIndex);   //0
alert(pattern1.source);   //"\[bc\]at"
```

### RegExp实例方法：
#### function exec:
专门为捕获组而设计，返回包含第一个匹配信息的数组
```
var text = "mom and dad and baby";
var pattern = /mom(and dad(and baby)?)?/gi;
var matches = pattern.exec(text);
alert(matches.index);  //0
alert(matches.input);  //"mom and dad and baby"
alert(matches[0]);   //"mom and dad and baby"
alert(matches[1]);   //"and dad and baby"
alert(matches[2]);   //"and baby"
```

#### function test:
接受一个字符串参数，在模式与该参数匹配的情况下返回true，否则，返回false
```
var text = "000-000-0000";
var pattern = /\d{3}-\d{2}-\d{4}/;
if(pattern.test(text)){
    alert("The pattern was matched");
}
```

#### function toLocaleString()和toString():
返回正则表达式的字面量，与创建正则表达式的方式无关

### 构造函数属性：
input：返回最近一次要匹配的字符串
leftContext：返回input字符串中lastMatch之前的文本
lastMatch：返回最近一次的匹配项
lastParen：返回最近一次匹配的捕获组
multiline：布尔值，表示是否所有表达式都是用多行模式
rightContext：input字符串中lastMatch之后的文本
```
var text = "this has been a short summer";
var patern = /(:)hort/g;
if(pattern.test(text)){
    alert(RegExp.input);   //this has been a short summer
    alert(RegExp.rightContext);   //this has been a 
    alert(RegExp.lastMatch);   //short
    alert(RegExp.lastParen);   //s
    alert(RegExp.multiline);   //false
}
```

### 模式的局限性：
ECMAScript不支持的特性：
> 匹配字符串开始和结尾的\A和\Z锚
> 向后查找
> 并集和交集类
> 原子组
> Unicode支持
> 命名的捕获组
> s(single，单行)和x(free-spacing，无间隔)匹配模式
> 条件匹配
> 正则表达式注释

Function类型
-------------
### 定义方式：
```
function sum(num1,num2){
    return num1 + num2;
}
var sum = function(num1,num2){
    return num1 + num2;
}
```

### 没有重载：
```
function addNumber(num){
    return num + 100;
}
funciton addNumber(num){
    return num + 200;
}
var result = addNumber(100);   //300
```

### 函数声明与函数表达式：
解析器会率先读取函数声明，并使其在执行任何代码之前可用
```
alert(sum(0,10));
function sum(num1,num2){
    return num1 + num2;
}
```

### 作为值的函数：
可以像传递参数一样把一个函数传递给另一个函数
```
function callSomeFunction(someFunction,someArgument){
    return someFunction(someArgument);
}
function add10(num){
    return num + 10;
}
function greeting(name){
    return "Hello " + name;
}
var result = callSomeFunction(add10,10);
alert(result);   //20
var result2 = callSomeFunction(greeting,"Van");
alert(result2);   //"Hello Van"
```
可以将一个函数作为另一个函数的结果返回
```
function createComparisonFunction(propertyName){
    return function (object1,object2){
        var value1 = object1[propertyName];
        var value2 = object2[propertyName];
        if(value1 < value2){
            return -1;
        }else if(value1 > value2){
            return 1;
        }else{
            return 0;
        }    
    }
}
```

### 函数内部属性：
#### callee:
一个指针，指向拥有这个arguments对象的函数
```
//函数执行与函数名"factorial"紧耦合在一起
function factorial(num){
    if(num <= 1){
        return 1;
    }else{
        return num * factorial(num-1);
    }
}

//无论函数取什么名字都能正常递归
function factorial(num){
    if(num <= 1){
        return 1;
    }else{
        return num * arguments.callee(num-1);
    }
}
```

#### this：
引用函数当前执行的环境对象
```
window.color = "red";
var o = {color:"blue"};
function sayColor(){
    alert(this.color);
}
sayColor();  //"red"，this为全局对象
o.sayColor = sayColor;
alert(0.sayColor());  //"blue"，this为对象o
```

#### caller:
ES5引入，保存调用当前函数的函数引用，如果实在全局调用，其值为null
```
function outer(){
    inner();
}
function inner(){
    alert(inner.caller); //可解耦合写成alert(arguments.callee.caller)
}
outer();  //outer
```

### 函数的属性和方法：
#### length属性:
表示函数希望接收的命名参数个数
```
function sayName(name){}
function sum(num1,num2){}
function sayHi(){}
alert(sayName.length);  //1
alert(sum.length);  //2
alert(sayHi.length);  //0
```

#### prototype属性:
保存它们所有实例方法的真正所在

#### function apply:
在特定的作用域中调用函数
```
function sum(num1,num2){
    return num1 + num2;
}
function callSum1(num1,num2){
    return sum.apply(this,arguments);
}
function callSum2(num1,num2){
    return sum.apply(this,[num1,num2]);
}
```

#### function apply:
与apply方法的作用相同，区别在于除了第一个参数this,其余参数都须直接传给函数
```
function sum(num1,num2){
    return num1 + num2;
}
function callSum(num1,num2){
    return sum.call(this,num1,num2);
}
```

apply()和call()真正强大是能够扩充函数赖以运行的作用域
```
window.color = "red";
var o = {color:"blue"};
function sayColor(){
    alert(this.color);
}
sayColor();  //"red"
sayColor.call(this);  //"red"
sayColor.call(window);  //"red"
sayColor.call(o);   //"blue"
```
用这种方法来扩充作用域对象不需要与方法有任何耦合关系

基本包装类型
-------------
### Boolean类型：
```
var falseObject = new Boolean(false);
var result = falseObjet && true;
alert(result);  //true
var falseValue = false;
result = falseValue && true;
alert(result);  //false
```

### Number类型：
Number类型重写了valueOf,toLocalString和toString方法
```
var num = 10;
alert(num.toString());   //"10"，默认十进制表示
alert(num.toString(2));  //"1010"，二进制表示
alert(num.toFixed(2));   //"10.00"，按照指定小数位返回
alert(num.toExponential(1));   //"1.0e+1"，按照指定位数返回指数表示法
```

### String类型：
#### 字符方法：
```
var stringValue = "hello world";
alert(stringValue.charAt(1));  //"e"
alert(stringValue.charCodeAt(1));  //"101",小写字母e字符编码
```

#### 字符串操作方法：
```
var stringValue = "hello";
var result = stringValue.concat(" world","!");  //返回新字符串
alert(result);  //"hello world!"
alert(stringValue);  //"hello"

var stringValue = "hello world";
alert(stringValue.slice(3));  //"lo world"
alert(stringValue.subString(3));  //"lo world"
alert(stringValue.subStr(3));  //"lo world"

alert(stringValue.slice(3,7));  //"lo w"，起始位置，结束位置
alert(stringValue.subString(3,7));  //"lo w"，同上
alert(stringValue.subStr(3,7));  //"lo worl"，起始位置，返回字符个数

alert(stringValue.slice(-3));  //"rld"
alert(stringValue.subString(-3));  //"hello world"，把所有负数转换成0
alert(stringValue.subStr(-3));  //"rld"，负数加上数组长度

alert(stringValue.slice(3,-4));  //"lo w"
alert(stringValue.subString(3,-4));  //"hel"，把负数变成0，从小的开始
alert(stringValue.subStr(3,-4));  //" "，把负数变成0
```

#### 字符串位置方法：
```
var stringValue = "hello";
alert(stringValue.indexOd("o"));  //4
alert(stringValue.lastIndexOf("o"));  //7
```
可以通过循环调用，来找到所有匹配的子字符串
```
var stringValue = "Lorem ipsum color sit amet, consectetur adipisicing elit";
var positions = new Array();
var pos = stringValue.indexOf("e");
while(pos>-1){
    positions.push(pos);
    pos = stringValue.indexOf("e",pos+1);
}
alert(positions);  //"3,24,32,35,52"
```

#### trim方法：
喊出前置和后缀的所有空格
```
var stringValue = "  hello world";
var trimedString = stringValue.trim();
alert(stringValue);  //"  hello world"
alert(trimedString);  //"hello world"
```

#### 字符串大小写转换方法：
```
var stringValue = "hello world";
alert(stringValue.toLocaleUpperCase());   //"HELLO WORLD"
alert(stringValue.toUpperCase());   //"HELLO WORLD"
alert(stringValue.toLocaleLowerCase());   //"hello world"
alert(stringValue.toLowerCase());   //"hello world"
```

#### 字符串模式匹配方法：
```
var text = "cat,bat,sat,fat";
var pattern = /.at/;
//与pattern.exec()相同
var matches = text.match(pattern);
alert(matches.index);  //0
alert(matches[0]);  //"cat"，返回第一个匹配的字符串
alert(pattern.lastIndex);  //0

var pos = text.search(/at/);
alert(pos);  //1，返回第一个匹配项的索引，没有找到则返回-1
var result = text.replace("at","ond");  //只替换第一个子字符串
alert(result);  //"cond,bat,sat,fat"
result = text.replace(/at/g,"ond");
alert(result);  //"cond,bond,sond,fond"
```
使用replace第二个参数，可以实现更加精细的替换
```
function htmlEscape(text){
    return text.replace(/[<>"&]/g,function(match,pos,orignalText)){
        switch(match){
            case "<":
                return "&lt";
            case ">":
                return "&gt";
            case "&":
                return "&amp";
            case "\"":
                return "&quot";
        }
    }
}
```

#### split方法：
```
var colorText = "red,blue,green,yellow";
var colors1 = colorText.split(",");  //["red","blue","green","yellow"]
var colors2 = colorText.split(",",2);  //["red","blue"]，只包含两项
var colors2 = colorText.split(/[^\,]t/);  //["","","",""]
```

#### LocaleCompare方法：
比较两个字符串，如果字符串在字母表中应该排在字符串参数之前，则返回负数，排在之后，则返回一个正数，等于则返回0
```
var stringValue = "yellow";
alert(stringValue.localeCompare("black"));  //1
alert(stringValue.localeCompare("yellow"));  //0
alert(stringValue.localeCompare("zoo"));  //1
```

#### fromCharCode方法：
接收一或多个字符编码，将它们转换成一个字符串
```
alert(String.fromCharCode(104,101,108,108,111));  //"hello"
```

单体内置对象
-------------
开发人员不必显示的实例化内置对象
### Global对象
#### URI编码方法
```
var uri = "http://www.wrox.com/illegal.value.html#start";
alert(encodeURI(url);  
//https://www.wrox.com/illegal%20value.html#start
alert(encodeURIComponent(url));  
//https%3A%2F%2Fwww.wrox.com%2Fillegal%20value.html%23start
```
encodeURI不会对URI本身进行编码
```
var uri2 = "https%3A%2F%2Fwww.wrox.com%2Fillegal%20value.html%23start";
alert(decodeURI(uri)); 
//https%3A%2F%2Fwww.wrox.com%2Fillegal%20value.html%23start
alert(decodeURIComponent(uri));
//https://www.wrox.com/illegal%20value.html#start
```
decodeURI只能对encodeURI转换后的字符进行解码，而decodeURIComponent会将所有特殊字符解码

#### eval方法
通过eval执行的代码被认为是包含该次调用的执行环境的一部分，因此被执行的代码具有与该执行环境相同的作用域链
```
var msg = "hello world";
eval("alert(msg)");  //"hello world"
```

#### window对象
```
var color = "red";
function sayColor(){
    alert(window.color);
}
window.sayColor();  //"red"
```

### Math对象
#### min和max方法
```
var max = Math.max(3,54,32,16);
alert(max);  //54
var min = Math.min(3,54,32,16);
alert(min);  //3
```
可以使用apply方法
```
var values = [1,2,3,4,5,6,7,8];
var max = Math.max.apply(Math,values);
```

#### 四舍五入方法
```
alert(Math.ceil(25.9));  //总是将数值向上舍入为最近的整数，26
alert(Math.floor(25.9));  //总是将数值向下舍入为最近的整数，25
alert(Math.round(25.9));  //四舍五入为最近的整数，26
```

#### random方法
值：Math.floor(Math.random()*可能的值的总数+第一个可能的值)
```
var num = Math.floor(Math.random()*9+2);  //返回2-10的值
```