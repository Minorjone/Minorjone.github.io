---
title: 'SASS,LESS,Stylus哪家强'
date: 2019-08-30 16:41:14
tags: [前端,LESS,SASS,Stylus]
categories: 前端
keywords: [前端,CSS,CSS3,LESS,SASS,Stylus]
description: 
---
俗话说得好，“人靠衣装，佛靠金装”，对于一个页面来说，美观与否，CSS起着重大的作用，而随着越来越多的使用，CSS也暴露出不少缺陷：
+ 语法不够强大，比如无法嵌套书写导致模块化开发中需要书写很多重复的选择器
+ 没有变量和合理的样式复用机制，使得逻辑上相关的属性值必须以字面量的形式重复输出，导致难以维护

于是，CSS预处理器就诞生了。CSS预处理器用一种专门的编程语言，进行Web页面样式设计，然后再编译成正常的CSS文件，以供项目使用。CSS预处理器为CSS增加一些编程的特性，无需考虑浏览器的兼容性问题。 
本文将主要介绍 Sass、Less 和 Stylus 这三种 css 预处理器，并比较分析他们的区别。

<!--more-->

Sass/Scss、Less、stylus是什么?
-------------
+ Sass：Sass是一种动态样式语言，语法属于缩排语法，是最早也是最成熟的CSS预处理器，拥有ruby社区的支持和compass这一最强大的css框架，目前受LESS影响，已经进化到了全面兼容CSS的SCSS。
+ Less：Less也是一种动态样式语言. 受Sass影响较大,对CSS赋予了动态语言的特性，如变量，继承，运算， 函数。Less既可以在客户端上运行 (支持IE 6+, Webkit, Firefox)，也可在服务端运行(借助 Node.js)。
+ Stylus：2010年产生于node社区， 主要用来给Node项目进行CSS预处理支持，人气不如前两者Stylus被称为是一种革命性的新语言，提供一个高效、动态、和使用表达方式来生成CSS，以供浏览器使用。Stylus同时支持缩进和CSS常规样式书写规则。需要安装nodeStylus的语法花样多一些，它的文件扩展名是“.styl”，Stylus也接受标准的CSS语法，但是他也像Sass老的语法规则，使用缩进控制，同时Stylus也接受不带大括号({})和分号的语法。

Sass/Scss、Less、stylus的区别
-------------
### 编译环境
Sass的安装需要Ruby环境，是在服务端处理的，而Less是需要引入less.js来处理Less代码输出css到浏览器，也可以在开发环节使用Less，然后编译成css文件，直接放到项目中，也有 Less.app、SimpleLess、CodeKit.app这样的工具，也有在线编译地址。Stylus需要安装node，然后安装最新的stylus包即可使用。

### 变量声明
#### Sass
变量以 `$` 符号开始，赋值像设置CSS属性那样。
```
$width: 5em;
#main {
  width: $width;
}
```
编译为：
```
#main {
  width: 5em;
}
```
变量支持块级作用域，嵌套规则内定义的变量只能在嵌套规则内使用（局部变量），不在嵌套规则内定义的变量则可在任何地方使用（全局变量）。将局部变量转换为全局变量可以添加 !global 声明。
```
#main {
  $width: 5em !global;
  width: $width;
}

#sidebar {
  width: $width;
}
```
编译为：
```
#main {
  width: 5em;
}

#sidebar {
  width: 5em;
}
```

#### Less
变量以 `@` 符号开始，赋值像设置CSS属性那样。
```
@width: 10px;
@height: @width + 10px;

#header {
  width: @width;
  height: @height;
}
```
编译为：
```
#header {
  width: 10px;
  height: 20px;
}
```
变量是延迟加载的，在使用前不一定要预先声明。在定义一个变量两次时，只会使用最后定义的变量，Less会从当前作用域中向上搜索。这个行为类似于CSS的定义中始终使用最后定义的属性值。
```
@var: 0;
.class {
  @var: 1;
  .brass {
    @var: 2;
    three: @var;
    @var: 3;
  }
  one: @var;
}
```
编译为：
```
.class {
  one: 1;
}
.class .brass {
  three: 3;
}
```

#### Stylus
没有特殊的要求,可以使用 `$` 符号开头。
```
font-size = 14px
font = font-size "Lucida Grande", Arial
body
  font font, sans-serif
```
编译为：
```
body {
  font: 14px "Lucida Grande", Arial, sans-serif;
}
```

### 插值语句
#### Sass
通过 `#{}` 插值语句可以在选择器或属性名中使用变量。
```
$name: foo;
$attr: border;
p.#{$name} {
  #{$attr}-color: blue;
}
```
编译为:
```
p.foo {
  border-color: blue; 
}
```

#### Less
通过 `{}` 插值语句可以在选择器或属性名中使用变量。
```
@mySelector: banner;
.@{mySelector} {
  font-weight: bold;
  line-height: 40px;
  margin: 0 auto;
}
```
编译为：
```
.banner {
  font-weight: bold;
  line-height: 40px;
  margin: 0 auto;
}
```

#### Stylus
Stylus支持通过使用{}字符包围表达式来插入值，其会变成标识符的一部分。
```
vendor(prop,args)
  -webkit-{prop} args
  -moz-{prop} args
  {prop} args
border-radius()
  vendor('border-radius',arguments)
button
  border-radius 1px 2px / 3px 4px
```
编译为：
```
button {
  -webkit-border-radius: 1px 2px / 3px 4px;
  -moz-border-radius: 1px 2px / 3px 4px;
  border-radius: 1px 2px / 3px 4px;
}
```

### 嵌套
在嵌套上，三种选择器基本一致，没有太大区别。
```
a {
  font-weight: bold;
  text-decoration: none;
  &:hover { text-decoration: underline; }
  body.firefox & { font-weight: normal; }
}
```
编译为：
```
a {
  font-weight: bold;
  text-decoration: none; 
}
a:hover {
  text-decoration: underline; 
}
body.firefox a {
  font-weight: normal; 
}
```
Sass还提供一种属性嵌套功能。
```
.funky {
  font: {
    family: fantasy;
    size: 30em;
    weight: bold;
  }
}
```
编译为：
```
.funky {
  font-family: fantasy;
  font-size: 30em;
  font-weight: bold; 
}
```

### 继承
#### Sass
```
.error {
  border: 1px #f00;
  background-color: #fdd;
}
.attention {
  font-size: 3em;
  background-color: #ff0;
}
.seriousError {
  @extend .error;
  @extend .attention;
  border-width: 3px;
}
```
编译为：
```
.error, .seriousError {
  border: 1px #f00;
  background-color: #fdd; 
}
.attention, .seriousError {
  font-size: 3em;
  background-color: #ff0; 
}
.seriousError {
  border-width: 3px; 
}
```

#### Less
```
nav ul {
  &:extend(.inline);
  background: blue;
}
.inline {
  color: red;
}
```
编译为：
```
nav ul {
  background: blue;
}
.inline,
nav ul {
  color: red;
}
```

#### Stylus
```
form
  input[type=text]
  padding: 5px
  border: 1px solid #eee
  color: #ddd

textarea
  @extends form input[type=text]
  padding: 10px
```
编译为：
```
form input[type=text],
textarea {
  padding: 5px;
  border: 1px solid #eee;
  color: #ddd;
}
textarea {
  padding: 10px;
}
```

### 混入 (Mixins)
#### Sass
```
@mixin sexy-border($color, $width: 1in) {
  border: {
    color: $color;
    width: $width;
    style: dashed;
  }
}
p { @include sexy-border(blue); }
h1 { @include sexy-border(blue, 2in); }
```
编译为：
```
p {
  border-color: blue;
  border-width: 1in;
  border-style: dashed; 
}
h1 {
  border-color: blue;
  border-width: 2in;
  border-style: dashed; 
}
```

#### Less
```
.my-mixin {
  color: black;
}
.my-other-mixin() {
  background: white;
}
.class {
  .my-mixin;
  .my-other-mixin;
}
```
编译为：
```
.my-mixin {
  color: black;
}
.class {
  color: black;
  background: white;
}
```

#### Stylus
```
border-radius(n)
  -webkit-border-radius n
  -moz-border-radius n
  border-radius n
form input[type=button]
  border-radius(5px)
```
编译为：
```
form input[type=button] {
  -webkit-border-radius: 5px;
  -moz-border-radius: 5px;
  border-radius: 5px;
}
```

### @import
三种语言基本一致。
```
@import "foo.css";
@import "foo" screen;
@import "http://foo.com/bar";
@import url(foo);
```
Sass还提供一种分音。如果需要导入 SCSS 或者 Sass 文件，但又不希望将其编译为 CSS，只需要在文件名前添加下划线，这样会告诉 Sass 不要编译这些文件，但导入语句中却不需要添加下划线。
```
//将文件命名为 _colors.scss，便不会编译 _colours.css 文件。
@import "colors";
```


### 输出设置
Sass 提供了四种输出格式，可以通过 :style option 选项设定，或者在命令行中使用 --style 选项。
+ :nested：Nested （嵌套）样式，默认的输出格式。选择器与属性等单独占用一行，缩进量与 Sass 文件中一致，每行的缩进量反映了其在嵌套规则内的层数。
+ :expanded：输出更像是手写的样式，选择器、属性等各占用一行，属性根据选择器缩进，而选择器不做任何缩进。
+ :compact：每条 CSS 规则只占一行，包含其下的所有属性。嵌套过的选择器在输出时没有空行，不嵌套的选择器会输出空白行作为分隔符。
+ :compressed：删除所有无意义的空格、空白行、以及注释，力求将文件体积压缩到最小，同时也会做出其他调整，比如会自动替换占用空间最小的颜色表达方式。

### 控制指令
#### Sass
```
$type: monster;
p {
  @if $type == ocean {
    color: blue;
  } @else if $type == matador {
    color: red;
  } @else if $type == monster {
    color: green;
  } @else {
    color: black;
  }
}
```
编译为：
```
p {
  color: green; 
}
```

#### Less
在1.5.0以前，使用 `when` 的语法。
```
.my-optional-style() when (@my-option = true) {
  button {
    color: white;
  }
}
.my-optional-style();
```
1.5.0以后也可以使用 `& when` 的方式模拟 `if` 。
```
& when (@my-option = true) {
  button {
    color: white;
  }
  a {
    color: blue;
  }
}
```
也可以使用 `if` 。
```
@dr: if(@my-option = true, {
  button {
    color: white;
  }
  a {
    color: blue;
  }
});
@dr();
```

#### Stylus
更接近Sass。
```
overload-padding = true
if overload-padding
  padding(y, x)
    margin y x
body
  padding 5px 10px
```

总结
-------------
+ Sass的优势得益于对函数和库的支持，但是因为基于Ruby环境，并不是特别友好。
+ Less相对清晰，也更接近CSS本身的写法，由于受Sass的极大的影响，我们也能看到，其中一些长期被开发者诟病的写法也得到了改善，对于Less的前景还是相对看好的。
+ Stylus可以说是集合了Sass和Less的优点，并进行极大的自由化，虽然目前Stylus的使用程度不如前两者广泛，但是鉴于其基于Node环境，语法大多参考Sass，在未来将是一个十分有生命力的CSS预处理器。


> 参考链接：
> [浅谈css预处理器，Sass、Less和Stylus-知乎](https://zhuanlan.zhihu.com/p/23382462)
> [关于sass（scss）、less、postcss、stylus等的用法与区别-掘金](https://juejin.im/post/5c9b17cbf265da60c95b7c3a)