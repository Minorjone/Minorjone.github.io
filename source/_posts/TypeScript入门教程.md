---
title: TypeScript入门教程
date: 2020-07-09 22:00:15
tags: 前端
categories: 前端
keywords: [前端,TS,Typescript]
description: 
---

> 本文是博主在学习TypeScript时跟着官方文档，自己写的例子，便于对知识点的掌握、梳理。

# 基础类型
在Javascript中，声明的变量在被赋值前可以是任何数据类型，TypeScript中对变量的数据类型做了校验，一旦设置了指定的类型，变量就不能再被赋值其他类型的数据，这样更便于后期对变量的维护和查错。

<!-- more -->
## 布尔值
```js
let flag: boolean = false;
```

## 数字
```js
let num1: number = 6;
let num2: number = 0xf00d;
let num3: number = 0b1010;
let num4: number = 0o744;
```

## 字符串
```js
let str: string = 'Hello';

let age: number = 1;
let info: string = `I'm ${age} years old`;
```

## 数组
```js
let list1: number[] = [1, 2, 3];
let list2: Array<number> = [1, 2, 3];
```

## 元组 Tuple
```js
let x: [string, number];
x = ['hello', 10];
```

## 枚举
默认情况下，从0开始为元素编号，但可以手动给编号，也可以通过编号获得他的名字
```js
enum Flag1 { success, fail }
let flag1:Flag1 = Flag1.fail;

enum Flag2 { success = 1, fail }
let flag2:Flag2 = Flag2.fail;

enum Flag3 { success = 1, fail = 3 }
let flag3:Flag3 = Flag3.fail;

enum Flag4 { success, fail }
let flag4: string = Flag4[1];
```

## 任意值
```js
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false;
```

## 空值
用于函数时，指函数没有返回值；用于变量时，变量只能被赋值null或undefined
```js
function warnUser(): void {
  alert("This is my warning message");
}
let unusable: void = undefined;
```

## Null 和 Undefined
```js
let u: undefined = undefined;
let n: null = null;
```

## Never
表示的是那些永不存在的值的类型。
```js
function error(message: string): never {
  throw new Error(message);
}
```

## 类型断言
相当于其他语言中的类型转换。
```ts
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;

let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```

# 函数
## 函数定义
```ts
function getInfo(name: string, age: number): string {
  return `${name}--${age}`;
}
var getInfo2 = function(name: string, age: number): string {
  return `${name}--${age}`;
}

// 书写完整函数类型
var getInfo3: (name: string, age: number) => string = function(a: string, b: number): string {
  return `${a}--${b}`;
}
```

## 可选参数
可选参数必须放在参数最后面。
```ts
var getDetail2 = function(name: string, age?: number): string {
  return `${name}--${age}`;
}
getDetail2('Bob');
```

## 默认参数
与普通可选参数不同的是，带默认值的参数不需要放在必须参数的后面。 如果带默认值的参数出现在必须参数前面，用户必须明确的传入undefined值来获得默认值。
```ts
function getTest(name: string, age: number = 20): string {
  return `${name}--${age}`;
}
getTest('Bob');

function getTest2(name: string = 'Bob', age: number): string {
  return `${name}--${age}`;
}
getTest2(undefined, 13);
```

## 剩余参数
```ts
function sum (a: number, ...res: number[]): number {
  var sum = 0;
  res.map(item => {
    sum += item;
  });
  return sum;
}
```

## 函数重载
```ts
function testInfo (name:string):string;
function testInfo (age:number):string;
function testInfo (str:any):any {
  if (typeof str === 'string') {
    return `Name:${str}`;
  } else {
    return `Age:${str}`;
  }
}

function testInfo2 (name:string):string;
function testInfo2 (name:string, age:number):string;
function testInfo2 (name:any, age?:any):any {
  if (age) {
    return `Name:${name},Age:${age}`;
  } else {
    return `Name:${name}`;
  }
}
```

# 类
## 定义
```ts
class Person {
  name: string;
  constructor (name: string) {
    this.name = name;
  }
  run (): void {
    console.log(`Name:${this.name}`);
  }
  getName (): string {
    return this.name;
  }
  setName (name: string): void {
    this.name = name
  }
}
var p = new Person('Amy');
p.setName('David')
```

## 继承
```ts
class Human {
  name: string;
  constructor (name: string) {
    this.name = name;
  }
  run (): string {
    return `Name:${this.name}`;
  }
}
class Tiny extends Human {
  constructor (name: string) {
    super(name);
  }
  work () {
    console.log(`Hello,${this.name}`)
  }
}
var t = new Tiny('Wang');
console.log(t.run())
t.work();
```

## 修饰符
+ public：当前类，子类和外部均可访问
+ protect: 当前类，子类可访问，外部不可访问
+ private: 只在当前类可访问，子类和外部都不可访问
```ts
class Person {
  public name: string;
  protect age: number;
  private gender: string;
  constructor (name: string, age: number, gender: string) {
    this.name = name;
    this.age = age;
    this.gender = gender;
  }
}

class Adult extends Person{
  constructor (name: string, age: number, gender: string) {
    super(name, age, gender);
  }
  print () {
    console.log(this.name);
    console.log(this.age);
    console.log(this.gender); // 报错，不能访问
  }
}
var a = new Adult('John', 13, 'female');
var p = new Person('John', 13, 'female');
console.log(a.name);
console.log(a.gender); // 报错，不能访问
console.log(p.age); // 报错，不能访问
console.log(p.gender); // 报错，不能访问
```

## readonly 修饰符
使用readonly关键字将属性设置为只读的。 只读属性必须在声明时或构造函数里被初始化。
```ts
class Person {
  readonly name: string;
  readonly age: number = 13;
  constructor (name: string) {
    this.name = name;
  }
}
let p = new Person('John');
p.name = 'Bob'; // 报错，属性只读
```

## 存取器
只带有 get不带有 set的存取器自动被推断为 readonly。
```ts
class Person {
  private _name: string;
  get name ():string {
    return this._name;
  }
  set name (name: string): void {
    this._name = name;
  }
}

let p = new Person();
p.name = "Bob";
if (p.name) {
    alert(p.name);
}
```

## 静态属性
类的实例成员仅当类被实例化的时候才会被初始化。 创建类的静态成员，这些属性存在于类本身上面而不是类的实例上。
```ts
class Base {
  public name: string;
  static age: number = 20;
  constructor (name: string) {
    this.name = name
  }
  run () {
    console.log(`${this.name} is running`)
  }
  work () {
    console.log(`${this.age} is working`) // 不能访问静态属性
  }
  static print () { // 静态方法只能访问类的静态属性
    console.log(`This is static func:${this.age}`);
  }
}
var b = new Base('Amy');
b.run();
console.log(Base.age); // 可以访问
Base.print(); // 可以访问
```

## 多态
继承类可以改写父类的方法。
```ts
class Human {
  name: string;
  constructor (name: string) {
    this.name = name;
  }
  run (): string {
    return `Name:${this.name}`;
  }
}
class Tiny extends Human {
  constructor (name: string) {
    super(name);
  }
  run (): string {
    return `This is Tiny's Name:${this.name}`;
  }
}
var t = new Tiny('Wang');
console.log(t.run()); // This is Tiny's Name:Wang
```

## 抽象类
抽象类做为其它派生类的基类使用，一般不会直接被实例化。不同于接口，抽象类可以包含成员的实现细节，abstract关键字是用于定义抽象类和在抽象类内部定义抽象方法。
```ts
abstract class Animal {
  public name: string;
  constructor (name:string) {
    this.name = name
  }
  abstract eat():any;
}

class Rabbit extends Animal {
  // 抽象类子类必须实现抽象方法
  constructor (name:string) {
    super(name);
  }
  eat () {
    console.log(`${this.name} eat grass`)
  }
  jump () {
    console.log(`I'm jumpping`)
  }
}

var ani: Animal; // 允许创建一个对抽象类型的引用
ani = new Animal('Bob'); // 错误: 不能创建一个抽象类的实例
ani = new Rabbit('Bob'); // 允许对一个抽象子类进行实例化和赋值
ani.eat();
ani.jump(); // 错误: 方法在声明的抽象类中不存在
```

# 接口
## 属性接口
属性接口就是对传入参数的约束。
```ts
function printLabel (labelInfo: { label: string }): void {
  console.log(labelInfo.label);
}
printLabel({ label: '123' });
```
我们传入的对象参数实际上会包含很多属性，使用对象传入约束的方式，编译器只会检查那些必需的属性是否存在，并且其类型是否匹配。
```ts
interface FullName {
  firstName: string;
  secondName: string;
}
function printName (name: FullName) {
  console.log(name.firstName + name.secondName);
}
var obj = {
  age: 20,
  firstName: 'Lei',
  secondName: 'Ming'
}
printName(obj);
printName({ firstName: 'Amy',secondName:' cooper' });
```

### 可选属性
```ts
interface Info {
  firstName: string;
  secondName?: string;
}
function getName(info:Info) {
  console.log(info.firstName)
}
getName({ firstName: 'Amy' });
```
结合可选属性，实现ajax函数：
```ts
interface Config {
  type: string;
  url: string;
  data?: string;
  dataType: string;
}
function ajax (config: Config) {
  var xhr = new XMLHttpRequest();
  xhr.open(config.type, config.url);
  xhr.send(config.data);
  xhr.onreadystatechange = function () {
    if (xhr.readyState === 4 && xhr.status === 200) {
      if (config.dataType === 'json') {
        return JSON.parse(xhr.responseText);
      }
      return xhr.responseText;
    }
  }
}
ajax({
  type: 'get',
  url: 'https://www.baidu.com',
  dataType: 'json'
})
```

### 只读属性
只读属性只能在对象刚刚创建的时候修改其值。
```ts
interface Point {
  readonly x: number;
  readonly y: number;
}
var p: Point = { x: 10, y: 20 };
p.x = 5; // 错误，只读属性不可修改
```
TypeScript具有ReadonlyArray<T>类型，它与Array<T>相似，只是把所有可变方法去掉了，因此可以确保数组创建后再也不能被修改。
```ts
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // 错误不能修改
ro.push(5); // 错误不能修改
ro.length = 100; // 错误不能修改
a = ro; // 错误不能修改
```
最简单判断该用readonly还是const的方法是看要把它做为变量使用还是做为一个属性。 做为变量使用的话用 const，若做为属性则使用readonly。

##  函数类型接口
```ts
interface encrypt {
  (key: string, value: string): string;
}
var md5: encrypt = function (key: string, value: string): string {
  return key + value;
}
```

## 可索引接口
TypeScript支持两种索引签名：字符串和数字。
```ts
interface UserArr {
  [index: number]: string;
}
var arr3: UserArr = ['aa', 'bb'];

interface UserObj {
  [index: string]: string;
}
var objArr: UserObj ={name:'Jon', age: '2'}
```

## 类类型接口
```ts
interface Animal {
  name: string;
  eat (str: string): void;
}
class Pig implements Animal {
  name: string;
  constructor(name: string) {
    this.name = name
  }
  eat () {
    console.log(`This is pig`)
  }
}
class Elephan implements Animal {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  eat (food: string) {
    console.log(`Eat ${food}`);
  }
}
```

## 继承接口
一个接口可以继承多个接口，创建出多个接口的合成接口。
```ts
interface Animal {
  name: string;
}
interface Cute {
  level: string;
}
interface Rabbit extends Animal, Cute {
  age: number;
}
var rab = <Rabbit>{};
rab.name = 'Haru';
rab.level = 'cutest';
rab.age = 1
```

## 接口继承类
```ts
interface PP {
  eat(): void;
}
interface P2 extends PP {
  work(): void;
}
class Programmer {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  coding(code: string) {
    console.log(this.name + code);
  }
}
class Web extends Programmer implements P2{
  constructor(name: string) {
    super(name);
  }
  eat() {
    console.log(`Eating`)
  }
  work() {
    console.log(`Working`);
  }
}
var ww = new Web('Sara');
ww.eat()
ww.work()
ww.coding(' python')
```

# 泛型
## 使用
```ts
function getData<T> (value: T): T {
  return value;
}
getData<number>(123);
```

## 泛型类
```ts
class MinClass<T> {
  public list: T[] = [];
  add (num: T): void {
    this.list.push(num)
  }
  min (): T {
    var minNum = this.list[0]
    for (var i = 0; i < this.list.length; i++) {
      if (minNum > this.list[i]) {
        minNum = this.list[i];
      }
    }
    return minNum;
  }
}
var m = new MinClass<number>();
m.add(9);
m.add(3);
m.add(5);
console.log(m.min());
```

## 泛型接口
```ts
// 方法一：
interface ConfigFn {
  <T>(val: T): T;
}
var getDa: ConfigFn = function<T>(val: T): T {
  return val;
}
getDa<number>(13);

// 方法二：
interface ConfigFn2<T> {
  (val: T): T;
}
function getDa2<T>(val: T): T {
  return val;
}
var myGetDa: ConfigFn2<string> = getDa2;
var myGetDa2: ConfigFn2<string> = function<T>(val: T): T {
  return val;
}
myGetDa('13');
myGetDa2('12');

// 在泛型中继承接口
interface ConfigFn3 {
  length: number;
}
function myGetDa3<T extends ConfigFn3>(arg: T): T {
  return arg;
}
myGetDa3(3); // 错误，传入参数有限制
myGetDa3({length: 10, value: 3}); // 成功
```

## 实例
利用泛型实现一个数据库类。
```ts
interface DBI<T> {
  add(info:T): boolean;
  update(info:T, id:number): boolean;
  delete(id:number): boolean;
  get(id:number):any[];
}
export class MysqlDb<T> implements DBI<T> {
  constructor() {
    console.log(`Connecting DB`);
  }
  add(info: T): boolean {
    console.log(info);
    return true;
  }
  update(info: T, id: number): boolean {
    throw new Error("Method not implemented.");
  }
  delete(id: number): boolean {
    throw new Error("Method not implemented.");
  }
  get(id: number): any[] {
    throw new Error("Method not implemented.");
  }
}
export class MssqlDb<T> implements DBI<T> {
  add(info: T): boolean {
    throw new Error("Method not implemented.");
  }
  update(info: T, id: number): boolean {
    throw new Error("Method not implemented.");
  }
  delete(id: number): boolean {
    throw new Error("Method not implemented.");
  }
  get(id: number): any[] {
    throw new Error("Method not implemented.");
  }
}
```

# 高级类型
## 交叉类型
使用`&`合并两个类，这让我们可以把现有的多种类型叠加到一起成为一种类型，它包含了所需的所有类型的特性。
```ts
function extend<T, U>(first: T, second: U): T & U {
  let result = <T & U>{};
  for (let id in first) {
    (<any>result)[id] = (<any>first)[id];
  }
  for (let id in second) {
    if (!result.hasOwnProperty(id)) {
      (<any>result)[id] = (<any>second)[id];
    }
  }
  return result;
}

class Person {
  constructor(public name: string) { }
}
interface Loggable {
  log(): void;
}
class ConsoleLogger implements Loggable {
  log() {}
}
var jim = extend(new Person("Jim"), new ConsoleLogger());
var n = jim.name;
jim.log();
```

## 联合类型
```ts
let uniType: string | number;
uniType = 1;  // 正确
uniType = '2'; // 正确
uniType = true; // 错误
```
如果一个值是联合类型，我们只能访问此联合类型的所有类型里共有的成员。
```ts
interface Bird {
  fly();
  layEggs();
}

interface Fish {
  swim();
  layEggs();
}

function getSmallPet(): Fish | Bird {}
let pet = getSmallPet();
pet.layEggs(); // 正确
pet.swim();    // 错误
```

## 类型别名
类型别名会给一个类型起个新名字。 类型别名有时和接口很像，但是可以作用于原始值，联合类型，元组以及其它任何你需要手写的类型。

起别名不会新建一个类型 - 它创建了一个新名字来引用那个类型。
```ts
// 可以使用类型别名来在属性里引用自己
type Tree<T> = {
  value: T;
  left: Tree<T>;
  right: Tree<T>;
}

// 与交叉类型一起使用
type LinkedList<T> = T & { next: LinkedList<T> };
interface Person {
  name: string;
}
var people: LinkedList<Person>;
var s = people.name;
var s = people.next.name;
var s = people.next.next.name;
var s = people.next.next.next.name;
```
使用注意：
+ 类型别名不能被 extends和 implements（自己也不能 extends和 implements其它类型）。
+ 如果你无法通过接口来描述一个类型并且需要使用联合类型或元组类型，这时通常会使用类型别名。

### 字符串字面量类型
```ts
type Easing = "ease-in" | "ease-out" | "ease-in-out";
class UIElement {
  animate(dx: number, dy: number, easing: Easing) {
    if (easing === "ease-in") {}
    else if (easing === "ease-out") {}
    else if (easing === "ease-in-out") 
    else {}
  }
}
let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy");
```

### 数字字面量类型
```ts
function rollDie(): 1 | 2 | 3 | 4 | 5 | 6 {}
```

### 可辨识联合
它具有3个要素：
+ 具有普通的单例类型属性— 可辨识的特征。
+ 一个类型别名包含了那些类型的联合— 联合。
+ 此属性上的类型保护。

```ts
interface Square {
  kind: "square";
  size: number;
}
interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}
interface Circle {
  kind: "circle";
  radius: number;
}

type Shape = Square | Rectangle | Circle | Triangle;
function assertNever(x: never): never {
  throw new Error("Unexpected object: " + x);
}
function area(s: Shape) {
  switch (s.kind) {
    case "square": return s.size * s.size;
    case "rectangle": return s.height * s.width;
    case "circle": return Math.PI * s.radius ** 2;
    default: return assertNever(s);
  }
}
```

## 多态的this类型
使用了this类型，你可以继承它，新的类可以直接使用之前的方法，不需要做任何的改变。
```ts
class BasicCalculator {
  public constructor(protected value: number = 0) { }
  public currentValue(): number {
    return this.value;
  }
  public add(operand: number): this {
    this.value += operand;
    return this;
  }
  public multiply(operand: number): this {
    this.value *= operand;
    return this;
  }
}

class ScientificCalculator extends BasicCalculator {
  public constructor(value = 0) {
    super(value);
  }
  public sin() {
    this.value = Math.sin(this.value);
    return this;
  }
}

let v = new ScientificCalculator(2)
        .multiply(5)
        .sin()
        .add(1)
        .currentValue();
```

## 索引类型
```ts
// JS版本
function pluck(o, names) {
  return names.map(n => o[n]);
}

// TS版本
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);

interface Person {
  name: string;
  age: number;
}
let person: Person = {
  name: 'Jarid',
  age: 35
};
let strings: string[] = pluck(person, ['name']);
```
TS中为了检测属性是否存在类中，引入了几个类型操作符：
+ keyof T：索引类型查询操作符。
+ T[K]：索引访问操作符。

## 映射类型
TypeScript提供了从旧类型中创建新类型的一种方式 — 映射类型。 在映射类型里，新类型以相同的形式去转换旧类型里每个属性。
```ts
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
}
type Partial<T> = {
  [P in keyof T]?: T[P];
}
```

# 命名空间
```ts
namespace A {
  export class UserInfo {
    username: string | undefined;
    password: string | undefined;
  }
}
var user = new A.UserInfo()
```

## 别名
这里的语法是为指定的符号创建一个别名。 你可以用这种方法为任意标识符创建别名，也包括导入的模块中的对象。
```ts
namespace Shapes {
  export namespace Polygons {
    export class Triangle { }
    export class Square { }
  }
}

import polygons = Shapes.Polygons;
let sq = new polygons.Square();
```

# 装饰器
类中不同声明上的装饰器将按以下规定的顺序应用：
1. 参数装饰器，然后依次是方法装饰器，访问符装饰器，或属性装饰器应用到每个实例成员。
2. 参数装饰器，然后依次是方法装饰器，访问符装饰器，或属性装饰器应用到每个静态成员。
3. 参数装饰器应用到构造函数。
4. 类装饰器应用到类。


## 类装饰器
类装饰器应用于类构造函数，可以用来监视，修改或替换类定义。
```ts
// 无参数
function logClass(params: any) {
  console.log(params);
  params.prototype.apiUrl = 'http://xxx.com';
}
@logClass
class HttpClient {
  constructor () {}
  getData () {}
}
var http = new HttpClient();
console.log(http.apiUrl); // http://xxx.com

// 传参数
function logClass2(params: any) {
  return function(target: any) {
    target.prototype.apiUrl = params;
  }
}
@logClass2('hello')
class HelloClient {
  constructor() {}
  getData() {}
}
var hell = new HelloClient();
console.log(hell); // { apiUrl: 'hello', getData: f() }
```

## 方法装饰器
被应用到方法的属性描述符上，可以用来监视，修改或者替换方法定义。

可以接受下列3个参数：
1. 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
2. 成员的名字。
3. 成员的属性描述符。
```ts
class Greet {
  greeting: string;
  constructor (message: string) {
    this.greeting = message;
  }
  @enumerable(false)
  greet(): string {
    return `Greeting ${this.greeting}`;
  }
}
function enumerable (val: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = val;
  }
}
var g = new Greet('John');
console.log(Object.getOwnDescriptorProperty(g, 'greet')); 
// {
//   enumerable: false
// }
```

## 访问器装饰器
访问器装饰器应用于访问器的 属性描述符并且可以用来监视，修改或替换一个访问器的定义，使用和方法装饰器相同。

> 注意：TypeScript不允许同时装饰一个成员的get和set访问器。取而代之的是，一个成员的所有装饰的必须应用在文档顺序的第一个访问器上。

```ts
class Point {
  private _x: number;
  private _y: number;
  constructor(x: number, y: number) {
    this._x = x;
    this._y = y;
  }
  @configurable(false)
  get x() { return this._x; }
  @configurable(false)
  get y() { return this._y; }
}
function configurable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.configurable = value;
  };
}
```

## 属性装饰器
```ts
function logProperty(params: any) {
  return function(target: any, attr: any) {
    target[attr] = params
  }
}
class Https {
  @logProperty('http://2222.com')
  public url: any | undefined;
}
var https = new Https();
console.log(https); // { url: "http://2222.com" }
```

## 参数装饰器
参数装饰器声明在一个参数声明之前（紧靠着参数声明）。可以传入三个参数：
+ 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
+ 成员的名字。
+ 参数在函数参数列表中的索引。

```ts
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
  @validate
  greet(@required name: string) {
    return "Hello " + name + ", " + this.greeting;
  }
}
```

# 三斜线指令
三斜线指令仅可放在包含它的文件的最顶端。 一个三斜线指令的前面只能出现单行或多行注释，这包括其它的三斜线指令。 如果它们出现在一个语句或声明之后，那么它们会被当做普通的单行注释，并且不具有特殊的涵义。
```ts
/// <reference path="..." />
```
三斜线引用告诉编译器在编译过程中要引入的额外的文件。

使用注意：
+ 预处理输入文件：编译器会对输入文件进行预处理来解析所有三斜线引用指令。
+ 错误：引用不存在的文件会报错。
+ 使用 --noResolve：如果指定了--noResolve编译选项，三斜线引用会被忽略。
