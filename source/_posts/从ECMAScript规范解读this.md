---
title: 从ECMAScript规范解读this
date: 2019-10-08 22:07:04
tags: [前端,JS]
categories: 前端
keywords: [前端,JS,this,ECMAScript]
description: 
---

说起JavaScript中的 `this` ，一直是一个非常让人头痛的东西，我们常会见到相关书籍和文章中用这样一句话来概括 this 的指向。
> this 为引用函数当前执行的环境对象。

然而这样的描述却十分的抽象，最近拜读了某位大佬从ECMAScript规范对this的解读，有一种豁然开朗的感觉，本文更像是一篇笔记，里面也包含了一些我对this的理解，供分享学习。

<!--more-->

Types
-------------
> Types are further subclassified into ECMAScript language types and specification types.
An ECMAScript language type corresponds to values that are directly manipulated by an ECMAScript programmer using the ECMAScript language. The ECMAScript language types are Undefined, Null, Boolean, String, Number, and Object.
A specification type corresponds to meta-values that are used within algorithms to describe the semantics of ECMAScript language constructs and ECMAScript language types. The specification types are Reference, List, Completion, Property Descriptor, Property Identifier, Lexical Environment, and Environment Record.

ECMAScript 的类型分为语言类型和规范类型。
ECMAScript 语言类型是开发者直接使用 ECMAScript 可以操作的。其实就是我们常说的Undefined, Null, Boolean, String, Number, 和 Object。
而规范类型相当于 meta-values，是用来用算法描述 ECMAScript 语言结构和 ECMAScript 语言类型的。规范类型包括：Reference, List, Completion, Property Descriptor, Property Identifier, Lexical Environment, 和 Environment Record。

Reference
-------------
Reference 是一个 Specification Type，也就是 “只存在于规范里的抽象类型”。它们是为了更好地描述语言的底层行为逻辑才存在的，但并不存在于实际的 js 代码中。
> A Reference is a resolved name binding. 
A Reference consists of three components, the base value, the referenced name and the Boolean valued strict reference flag. 
The base value is either undefined, an Object, a Boolean, a String, a Number, or an environment record (10.2.1). 
A base value of undefined indicates that the reference could not be resolved to a binding. The referenced name is a String.

这段讲述了 Reference 的构成，由三个组成部分，分别是：
+ base value：就是属性所在的对象或者就是 EnvironmentRecord，它的值只可能是 undefined, an Object, a Boolean, a String, a Number, or an environment record 其中的一种。
+ referenced name：就是属性的名称。
+ strict reference

```
//示例1
var foo = 1;

// 对应的Reference是：
var fooReference = {
    base: EnvironmentRecord,
    name: 'foo',
    strict: false
};

//示例2
var foo = {
    bar: function () {
        return this;
    }
};

foo.bar(); // foo

// bar对应的Reference是：
var BarReference = {
    base: foo,
    propertyName: 'bar',
    strict: false
};
```

规范中还提供了获取 Reference 组成部分的方法，比如 GetBase 和 IsPropertyReference。
### GetBase
> GetBase(V). Returns the base value component of the reference V.

返回 reference 的 base value。

### IsPropertyReference
> IsPropertyReference(V). Returns true if either the base value is an object or HasPrimitiveBase(V) is true; otherwise returns false.

简单的理解：如果 base value 是一个对象，就返回true。

GetValue
-------------
GetValue 返回对象属性真正的值。
```
var foo = 1;

var fooReference = {
    base: EnvironmentRecord,
    name: 'foo',
    strict: false
};

GetValue(fooReference) // 1;
```

MemberExpression
-------------
+ PrimaryExpression // 原始表达式 可以参见《JavaScript权威指南第四章》
+ FunctionExpression    // 函数定义表达式
+ MemberExpression [ Expression ] // 属性访问表达式
+ MemberExpression . IdentifierName // 属性访问表达式
+ new MemberExpression Arguments    // 对象创建表达式

```
function foo() {
    console.log(this)
}

foo(); // MemberExpression 是 foo

function foo() {
    return function() {
        console.log(this)
    }
}

foo()(); // MemberExpression 是 foo()

var foo = {
    bar: function () {
        return this;
    }
}

foo.bar(); // MemberExpression 是 foo.bar
```
所以简单理解 MemberExpression 其实就是()左边的部分。

this 的确定步骤
-------------
+ 计算 MemberExpression 的结果赋值给 ref
+ 判断 ref 是不是一个 Reference 类型
> 1. 如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)
> 2. 如果 ref 是 Reference，并且 base value 值是 Environment Record, 那么this的值为 ImplicitThisValue(ref)
> 3. 如果 ref 不是 Reference，那么 this 的值为 undefined

案例分析
-------------
```
var value = 1;

var foo = {
  value: 2,
  bar: function () {
    return this.value;
  }
}

//示例1
console.log(foo.bar());
//示例2
console.log((foo.bar)());
//示例3
console.log((foo.bar = foo.bar)());
//示例4
console.log((false || foo.bar)());
//示例5
console.log((foo.bar, foo.bar)());
```

### foo.bar()
+ 判断MemberExpression 计算结果为 foo.bar
+ foo.bar的Reference
```
var Reference = {
  base: foo,
  name: 'bar',
  strict: false
};
```
+ IsPropertyReference(ref) 是 true
+ 返回GetBase()，为 `foo`

### (foo.bar)()
+ () 并没有对 MemberExpression 进行计算
+ 判断MemberExpression 计算结果为 foo.bar
+ foo.bar的Reference
```
var Reference = {
  base: foo,
  name: 'bar',
  strict: false
};
```
+ IsPropertyReference(ref) 是 true
+ 返回GetBase()，为 `foo`

### (foo.bar = foo.bar)()
+ 关于赋值运算符 `=` ，使用了 GetValue 进行赋值，所以返回值不为 Reference 类型
+ this 为 undefined
+ 非严格模式下，this 的值为 undefined 的时候，其值会被隐式转换为全局对象

### (false || foo.bar)()
+ 关于逻辑运算符，使用了 GetValue 进行求值，所以返回值不为 Reference 类型
+ this 为 undefined
+ 非严格模式下，this 的值为 undefined 的时候，其值会被隐式转换为全局对象

### (foo.bar, foo.bar)()
+ 关于逗号运算符，使用了 GetValue 进行运算，所以返回值不为 Reference 类型
+ this 为 undefined
+ 非严格模式下，this 的值为 undefined 的时候，其值会被隐式转换为全局对象

### 最终结果
```

var value = 1;

var foo = {
  value: 2,
  bar: function () {
    return this.value;
  }
}

//示例1
console.log(foo.bar()); // 2
//示例2
console.log((foo.bar)()); // 2
//示例3
console.log((foo.bar = foo.bar)()); // 1
//示例4
console.log((false || foo.bar)()); // 1
//示例5
console.log((foo.bar, foo.bar)()); // 1
//示例6
```

### 补充
```
function foo() {
    console.log(this)
}

foo();
```
+ MemberExpression 是 foo
+ foo 的 Reference
```
var fooReference = {
    base: EnvironmentRecord,
    name: 'foo',
    strict: false
};
```
+ base value 是 Environment Record，调用 ImplicitThisValue(ref)
+ ImplicitThisValue 方法始终返回 undefined。

改变this指向
-------------
### 箭头函数
箭头函数的 this 始终指向函数定义时的 this，而非执行时。箭头函数中没有 this 绑定，必须通过查找作用域链来决定其值，如果箭头函数被非箭头函数包含，则 this 绑定的是最近一层非箭头函数的 this，否则，this 为 undefined。
```
var name = "windowsName";

var a = {
    name : "Cherry",

    func1: function () {
        console.log(this.name)     
    },

    func2: function () {
        setTimeout( () => {
            this.func1()
        },100);
    }
};

a.func2()     // Cherry
```

### 在函数内部使用 _this = this
利用 _this 保存 this 对象。
```
var name = "windowsName";

var a = {

    name : "Cherry",

    func1: function () {
        console.log(this.name)     
    },

    func2: function () {
        var _this = this;
        setTimeout( function() {
            _this.func1()
        },100);
    }
};

a.func2()       // Cherry
```

### 使用 apply、call、bind
#### apply
```
var a = {
    name : "Cherry",

    func1: function () {
        console.log(this.name)
    },

    func2: function () {
        setTimeout(  function () {
            this.func1()
        }.apply(a),100);
    }

};

a.func2()            // Cherry
```

#### call
```
var a = {
    name : "Cherry",

    func1: function () {
        console.log(this.name)
    },

    func2: function () {
        setTimeout(  function () {
            this.func1()
        }.call(a),100);
    }

};

a.func2()            // Cherry
```

#### bind
```
var a = {
    name : "Cherry",

    func1: function () {
        console.log(this.name)
    },

    func2: function () {
        setTimeout(  function () {
            this.func1()
        }.bind(a)(),100);
    }

};

a.func2()            // Cherry
```

apply、call、bind 区别
-------------
### apply 和 call 的区别
apply 和 call 基本类似，他们的区别只是传入的参数不同。

apply 的语法为：
```
fun.apply(thisArg, [argsArray])
```

call 的语法为：
```
fun.call(thisArg[, arg1[, arg2[, ...]]])
```

### bind 和 apply、call 区别
bind 是创建一个新的函数，我们必须要手动去调用
```
var a ={
    name : "Cherry",
    fn : function (a,b) {
        console.log( a + b)
    }
}

var b = a.fn;
b.bind(a,1,2)()           // 3
```

> 参考链接：[JavaScript深入系列15篇（冴羽）](https://juejin.im/post/590159d8a22b9d0065c2d918)