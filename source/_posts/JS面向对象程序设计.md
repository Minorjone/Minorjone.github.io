---
title: JS面向对象程序设计
date: 2019-03-24 11:41:03
tags: 前端
categories: 前端
keywords: [前端,JS,面向对象]
description: 
---

属性类型
-------------
### 数据属性
|    属性名   |    解释   |
|:-------|:-----------|
|   [[Configurable]]  |     表示能否通过delete删除属性从而重新定义属性，默认为true   | 
|   [[Enumerable]]  |     表示能否通过for-in循环返回属性，默认为true    | 
|   [[Writable]]  |     表示能否修改属性的值，默认为true    | 
|   [[Value]]  |     包含这个属性的数值，默认为undefined    | 

```
var person = {};
Object.defineProperty(person,"name",{
    writable: false,
    value: "Nicholas"
});
alert(person.name);  // "Nicholas"
person.name = "Greg";
alert(person.name);  // "Nicholas",通过修改属性使之不能被修改
```

<!--more-->

### 访问器属性
|    属性名   |    解释   |
|:-------|:-----------|
|   [[Configurable]]  |     表示能否修改属性的特性，或者能否把属性修改为数据属性，默认为true   | 
|   [[Enumerable]]  |     表示能否通过for-in循环返回属性，默认为true    | 
|   [[Get]]  |     在读取属性时调用的函数，默认为undefined    | 
|   [[Set]]  |     在写入属性时调用的函数，默认为undefined    | 

```
var book = {
    _year: 2004,
    edition: 1
};
Object.defineProperty(book,"year",{
    get: function(){
        return this._year;
    }
    set: function(newValue){
        if(newVale > 2004){
            this._year = newValue;
            this.edition += newValue - 2004;
        }
    }
});
book.year = 2005;
alert(book.edition);  // 2,访问属性只能用Object.defineProperty定义
```

### 定义多个属性
使用Object.defineProperty方法定义多个属性

```
var book = {};
Object.defineProperty(book,{
    _year: {
        writable: true,
        value: 2004
    },
    edition: {
        wirtable: true,
        value: 1
    }
});
```

### 读取属性的特性

```
var descriptor = Object.getOwnPropertyDescriptor(book,"_year");
alert(descriptor.value);  //2004
alert(descriptor.configurable);   //false
alert(typeof descriptor.get);    //"undefined"
```

创建对象
-------------
### 工厂模式
抽象了创建具体对象的过程

```
function createPerson(name,age,job){
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function(){
        alert(this.name);
    };
    return o;
}
```

### 构造函数模式

```
function Person(name,age,job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function(){
        alert(this.name);
    };
}
var person1 = new Person("Nicholas",29,"Software Engineer");

// 使用call实现在特殊作用于中调用Person函数
var o = new Object();
Person.call(o,"Kristen",25,"Nurse");
o.sayName();   //"Kristen"
```

此方法特点：
+ 没有显示地创建对象
+ 直接将属性方法赋给了his对象
+ 没有return语句
+ 不同实例上的同名函数每次都是新创建初始化，他们之间不相等
最好将函数定义转移到构造函数外

```
function Person(name,age,job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = sayName;
}
function sayName(){
    alert(this.name);
}
var person1 = new Person("Nicholas",29,"Software Engineer");
```

### 原型模式
创建的每个函数都有一个prototype属性，这个属性指向一个对象，该对象的用途是包含可以由特定类型的所有实例共享的属性和方法。

```
function Person(){
}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function(){
    alert(this.name);
};
var person1 = new Person();
person1.sayName();   //"Nicholas",来自原型
```

![Alt text][01]
[01]: ../assets/jsoop/20190324153411.png

可以通过isPrototypeOf来确定对象之间是否存在这种关系

```
alert(Person.prototype.isPrototypeOf(person1));   //true
alert(Person.getPrototypeOf(person) == Person.prototype);   //true
alert(Obeject.getPrototypeOf(person1).name);   //"Nicholas"
```

可以通过hasOwnProperty来检测一个属性是否存在于实例中，只有存在于实例中，才会返回true

```
alert(person1.hasOwnPrototype("name"));   //false
person1.name = "Greg";
alert(person1.name);   //"Greg",来自实例
alert(person1.hasOwnPrototype("name"));   //true
delete person1.name;
alert(person1.name);   //"Nicholas",来自原型
alert(person1.hasOwnPrototype("name"));   //false
```

in操作符会在通过对象能够访问给定属性时返回true，无论其在实例还是原型中

```
alert("name" in person1);   //true
```

for-in循环，返回的是所有能够通过对象访问、可枚举的属性

```
var o = {
    toString: function(){
        return "My Object";
    }
};
for(var prop in o){
    if(pop == "toString"){
        alert("Found toString");   //IE中不会显示，认为原型中的toString被屏蔽
    }
}
```

使用Object.key可以取得所有可枚举的实力属性

```
var keys = Object.keys(Person.prototype);
alert(keys);   //"name,age,job,sayName"
var p1 = new Person();
p1.name = "Rob";
p1.age = 31;
var p1Keys = Object.keys(p1);
alert(p1Keys);   //"name,age"
```

更简单的原型创建语法

```
function Person(){
}
Person.prototype = {
    name: "Nicholas",
    age: 29,
    job: "Software Engineer",
    sayName: function(){
        alert(this.name);
    }
}
// 此时constructor不再指向Person了，如果需要可以将constructor的值设为Person
```

对原型对象所做的修改可以立即从实例上反映出来

```
var friend = new Person();
Person.prototype.sayHi = function(){
    alert("Hi");
}
friends.sayHi();   //"Hi"
```

但重写整个原型对象就不可以了，此时constructor指向新创建的对象

```
var Person(){
}
var friend = new Person();
Person.prototype = {
    constructor: Person,
    name: "Nicholas",
    age: 29,
    job: "Software Engineer",
    sayName: function(){
        alert(this.name);
    }
};
friends.sayName();   //error
```

通过原生对象原型，可以取得所有默认方法的引用，而且也可以定义新方法

```
String.prototype.startsWith = function(text){
    return this.indexOf(text) == 0;
};
var msg = "Hello world";
alert(msg.startsWith("Hello"));   //true
```

原生对象存在缺点，包含引用类型值的属性将被所有实例共享，例如属性中包含数组

### 组合使用构造函数和原型模式
构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性

```
function Person(name,age,job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ["Shelly","Court"];
}
Person.prototype = {
    constructor: Person,
    sayName: function(){
        alert(this.name);
    }
};
var person1 = new Person("Nicholas",29,"SoftWare Engineer");
var person2 = new Person("Greg",27,"Doctor");
person1.friends.push("Van");
alert(person1.friends);   //"Shelly,Court,Van"
alert(person2.friends);   //"Shelly,Court"
```

### 动态原型模式
通过检查某个应该存在的方法是否有效，来决定是否需要初始化原型

```
function Person(name,age,job){
    this.name = name;
    this.age = age;
    this.job = job;
    if(typeof this.sayName != "function"){
        Person.prototype.sayName = function(){
            alert(this.name);
        }
    }
}
```

### 寄生构造函数模式
该模式用来为对象创建构造函数，以此创建一个具有额外方法的特殊对象

```
function SpecialArray(){
    var values = new Array();
    values.push.apply(values,arguments);
    values.toPipedString = function(){
        return this.join("|);
    };
    return values;
}
var colors = new SpecialArray("red","blue","green");
alert(colors.toPipedString());    //"red|blue|green"
```

### 稳妥构造函数模式
稳妥对象最适合在一些安全的环境中（这些环境中会禁止使用this和new）或者防止数据被其他应用程序（如Mashup程序）改动时使用

```
function Person(name,age,job){
    var o = new Object();
    o.sayName = function(){
        alert(name);    //name只能通过sayName方法访问
    };
    return 0;
}
```

继承
-------------
### 原型链
实现原型链的基本模式

```
function SuperType(){
    this.property = true;
}
SuperType.prototype.getSuperValue = function(){
    return this.property;
};
function SubType(){
    this.subproperty = false;
}
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function(){
    return this.subproperty;
};
var instance = new SubType();
alert(instance.getSuperValue());   //true
```

![Alt text][02]
[02]: ../assets/jsoop/20190324225152.png

以上代码中，可用instanceof和isPrototypeOf来确定原型和实例之间的关系

```
alert(instance instanceOf Object);   //true
alert(instance instanceOf SuperType);   //true
alert(instance instanceOf SubType);   //true
alert(Object.prototype.isPrototypeOf(instance));   //true
```

通过原型链实现继承时，不能使用对象字面量创建原型方法，因为这样会重写原型链，原型链中存在问题，包含引用类型值的原型属性会被所有实例共享

### 借用构造函数

```
function SuperType(){
    this.colors: ["red","blue","green"];
}
function SubType(){
    SuperType.call(this);   //执行了SuperType中定义的所有对象初始化代码
}
var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors);   //"red,blue,green,black"
var instance2 = new SubType();
alert(instance2.colors);   //"red,blue,green"

// 还可以传递参数
function SuperType(name){
    this.name = name;
}
function SubType(){
    SuperType.call(this,"Nicholas");
    this.age = 29;
}
var instance = new SubType();
alert(instance.name);   //"Nicholas"
alert(instance.age);   //29
```

如同仅仅借用构造函数，则无法函数复用

### 组合继承
使用原型链实现对原型属性和方法的继承，通过借用构造函数实现对实例的继承

```
function SuperType(name){
    this.name = name;
    this.colors = ["red","blue","green"];
}
SuperType.prototype.sayName = function(){
    alert(this.name);
}
function SubType(name,age){
    SuperType.call(this,name);    //继承属性
    this.age = age;
}
SubType.prototype = new SuperType();   //继承方法
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function(){
    alert(this.age);
};
var instance1 = new SubType("Nicholas",29);
instance1.colors.push("black");
alert(instance1.colors);   //"red,blue,green,black"
instance1.sayName();   //"Nicholas"
instance1.sayAge();   //29
var instance2 = new SubType("Greg",27);
alert(instance2.colors);   //"red,blue,green"
instance2.sayName();   //"Greg"
instance2.sayAge();   //27
```

### 原型式继承

```
var person = {
    name: "Nicholas",
    friends: ["Shelby","Court","Van"]
};
var anotherPerson  Object(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");
var getAnotherPerson = Object(person);
getAnotherPerson.name = "Linda";
getAnotherPerson.friends.push("Barbie");
alert(person.friends);   //"Shelby,Court,Van,Rob,Barbie"
```

ES5新增Object.create()方法规范化了原型式继承，在传入一个参数时与Object方法相同

```
var person = {
    name: "Nicholas",
    friends: ["Shelby","Court","Van"]
};
var anotherPerson = Object.create(person,{
    name: {
        value: "Greg"
    }
});
alert(anotherPerson.name);   //"Greg"
```

### 寄生式继承
创建一个仅用于封装继承过程的函数，该函数在内部以某种方式来增强对象

```
function createAnother(original){
    var clone = Object(original);   //创建一个新对象，Object函数不是必需的
    clone.sayHi = function(){
        alert("Hi");
    };
    return clone;
}
```

### 寄生组合继承

```
fucntion inheritP(subType,superType){
    var prototype = Object(superType.prototype);
    prototype.constructor = subType;
    subType.prototype = prototype;
}
function SuperType(name){
    this.name = name;
    this.colors = ["red","blue","green"];
}
SuperType.prototype.sayName = function(){
    alert(this.name);
};
function SubType(name,age){
    SuperType.call(this.name);
    this.age = age;
}
inheritP(SubType,SuperType);
SubType.prototype.sayAge = function(){
    alert(this.age);
};
```

