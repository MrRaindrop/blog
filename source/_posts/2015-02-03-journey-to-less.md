---
layout: post
title:  "LESS之旅：简明教程"
date:   2015-02-03 17:53:00
categories: css
tags:
  - css
  - less
  - 样式表
  - css预处理
  - fe
  - 前端
  - front-end
  - 前端开发
author: "_danZ"
---

说明：下图是我根据less 2.3.1官方文档画的less知识点脑图，看这个图可以对less的基本语法和概念有一个大致的了解和初步印象，当然也可以略过脑图直接看下面的文字；如果想看更加详细的脑图（全部结点都展开）可以点图片下面的链接

<!--more-->

![mindmap of learning less](http://7u2loa.com1.z0.glb.clouddn.com/lessMin.jpeg)

-->> [点击查看详细大图](http://7u2loa.com1.z0.glb.clouddn.com/less.jpeg) <<--

在正式介绍less之前，推荐一个Less在线测试器：[lesstester](http://lesstester.com/)，可以将less在线转换成css代码，使用起来很简单，把less代码贴到编辑区，按ctrl+enter编译出来的css就会出现在右边。如果代码有问题则右边会报错，也有可能默默的把错误吞了什么也不提示。下面正式开始less之旅

# 1. 注释（Comments）

和css不同，less可以使用行注释``// ...``，而css只能使用块注释``/* ... */``

# 2. 变量（Variables）

> 在一个地方管理常用的值。

## 2.1 变量用于数值管理

定义变量：``@var-name: value;`` （变量只能定义一次）

使用变量：用变量替换值，注意加@符号``attribute: @var-name;``， 如：

```css
@my-color: #fff;
body {
  color: @my-color;
}
```

使用变量可以使css变得易于维护，因为css里一般相同的值可能重复很多遍，抽离出来做为变量将很大程度方便修改

LESS的变量是**延迟加载（lazy loading）**的，即定义变量的语句可以是在使用变量之后，使用前不一定要预先声明

同一作用域内的同一变量定义如果重复了，后面定义的会覆盖前面定义的

变量插值（Variable Interpolation）除了用于数值管理，还可以用于选择器名称、属性名、URLs以及@import语句；注意用于选择器名、属性名、URL、@import语句（其实也是替换URL）都要在两边加大括号``@{var-name}``如：

## 2.2 变量用于选择器名

```css
// 用于选择器名
@test-class-name: classA;
#header {
  color: blue;
  .@{test-class-name} {   // 即.classA
    color: red;    
  }
}
```

## 2.3 变量用于属性名

```css
// 用于属性名
@property-name: color;
#footer {
  @{property-name}: blue;
  background-@{property-name}: #999;
}
```

## 2.4 变量用于URLs

```css
// 变量定义URL前缀
// 注意一定要加双引号
@images: "../img";

// 用法
body {
  color: #444;
  background: url("@{images}/white-sand.png");
  // 也可以去掉url("")里的双引号
  // background: url(@{images}/white-sand.png);
}
```

## 2.5 变量用于Import语句

```css
@themes: "../../src/themes";

// 用法
@import "@{themes}/my-theme.less"
body {
  color: #444;
  background: url("@{images}/white-sand.png");
  // 也可以去掉url("")里的双引号
  // background: url(@{images}/white-sand.png);
}
```

# 3. 运算（Operations）

任何数值、颜色、变量都可以运算

示例如下：

```css
/*变量运算*/
@base: 5%;
@filler: @base * 2;
@other: @base + @filler;

@var: 1px + 5;  /*结果为6px*/

/*颜色和数值运算*/
color: #888 / 4;
background-color: @base-color + #111;
height: 100% / 2 + @filler;
```

# 4. 函数（Functions）

Less提供了一些用于处理颜色、字符串和算术运算的函数

例子如下：

```css
@base: #f04615;
@width: 0.5;

.some-class {
  width: percentage(@width); /*percentage转换为百分数即50%*/
  color: saturate(@base, 5%); /*饱和度+5%*/
  background-color: spin(lighten(@base, 25%), 8); /*亮度提升25%，接着色相增加8*/
}
```

# 5. 混合（Mixins）

mixin: 将一系列属性从一个规则集引入另一个规则集，方式如下：

```css
.bordered {
  border: 1px solid black;
}

.mixin-class {
  color: #000;
  .bordered();
  // 或者不加括号“()”，如下：
  // .bordered;
}
```

编译出来是这样的（即相当于将``.bordered``里的规则复制到``.mixin-class``里了）：

```css
.bordered {
  border: 1px solid black;
}

.mixin-class {
  color: #000;
  border: 1px solid black;
}
```

可以混入的选择器只有**类选择器**和**id选择器**，混入id选择器的例子：

```css
#some-id {
  color: red;
}
.mixined {
  #some-id;
}
```

## 5.1 创建一个不能被输出的mixin

要让某个mixin只用来混入（纯粹用来做mixin的），而不用来作为单独的一个css规则集输出到编译好的css中，则在该mixin的选择器名字后面加上括号``()``即可，如：

```css
#output {
  color: red;
}
#not-output() { // 注意这里加了括号
  border: 1px solid black;
}
.mixined {
  #output;
  #not-output;
}
```

编译出来的css如下：

```css
#output {
  color: red;
}
.mixined {
  color: red;
  border: 1px solid black;
}
```

可以看到上面的#not-output选择器，如愿的没有出现在编译出来的css当中

## 5.2 mixin中内嵌选择器

mixin中嵌套的子选择器可以一并混入到目标选择器中，如下代码所示，将上面的代码稍作修改：

```css
#output {
  color: red;
  .inner {
    background-color: red;
  }
}
.mixined {
  color: red;
  border: 1px solid black;
}
```

这里``.inner``是会混入的，结果是增加了一条``#output .inner``的规则，即``#output .inner { background-color:red; }``，编译结果如下：

```css
#output {
  color: red;
}
#output .inner {
  background-color: red;
}
.mixined {
  color: red;
  border: 1px solid black;
}
```

## 5.3 使用名字空间

对于需要混入某个内嵌的选择器的规则，则混入的时候需要使用名字空间，如：

```css
#outer {
  .inner {
    color: red;
  }
}
.mixined {
  #outer > .inner; // 混入.inner的规则
}
```

注意以下这些方式（是否带括号，以及是否带``>``或者空格）效果是一致的：

```css
#outer > .inner;
#outer > .inner();
#outer .inner;
#outer .inner();
#outer.inner; // 两名字之间的空格居然可以去掉！
#outer.inner();
```

根据上面的规则我发现一个比较奇葩的现象：

```css
.a.b {
    color: blue;
}

.a {
    .b {
        color: green;
    }
}

.d {
    .a.b; // 或者改成.a > .b结果都一样
}
```

编译出来的css居然是：

```css
.a.b {
  color: blue;
}
.a .b {
  color: green;
}
.d {
  color: blue;  // 这条有点莫名其妙
  color: green;
}
```

结果很奇怪，向less项目[提了条issue](https://github.com/less/less.js/issues/2409)，希望后续能搞明白怎么回事，估计是个bug；总之尽量不要这么写就是了 **看issue的回复，这个不是bug而是设计的feature**

下一节嵌套规则也会讲到名字空间（名字空间和访问器）

## 5.4 mixin后面加上!important

在mixin调用语句的后面加上 ``!important`` ，将mixin的规则都加上!important

```css
.mix {
  color: red;
}

.some-class {
  .mix !important;
}
```

编译成css的结果如下：

```css
.mix {
  color: red;
}
.some-class {
  color: red !important;
}
```

## 5.5 带变量参数的mixin

mixin里可以加入变量参数，如下：

```css
.border-radius(@radius) {
  -webkit-border-radius: @radius;
    -moz-border-radius: @radius;
      border-radius: @radius;
}
```

在混入的时候可以向这个变量传值：

```css
#header {
  .border-radius(4px);
}
```

可以为``@radius``设置默认值，比如

```css
.border-radius(@radius: 4px) {
  -webkit-border-radius: @radius;
    -moz-border-radius: @radius;
      border-radius: @radius;
}
```

在用于混入的选择器右边加括号，但是不向括号里加变量参数，可避免在编译的css里输出这些规则，只是用于做mixin，即上面所述的**创建一个不能被输出的mixin**

在调用mixin的时候，多个参数可以使用分号``;``或者逗号``,``隔开，推荐使用分号``;``因为逗号会和css的列表混淆：

* 当使用了逗号，但没有用分号，则逗号按mixin分隔符号来解析
* 当同时使用了逗号和分号，则分号按mixin分隔符号来解析，逗号当做css list的分隔符号，例如：

```css
.name(1, 2, 3; something, else); // 相当于传递了2个变量，一个是"1,2,3"的值列表，一个是"something,else"的值列表
.name(1, 2, 3); // 传递了3个变量，分别是1, 2和3
.name(1, 2, 3;); // 传递了1个变量，即"1,2,3"的值列表
.name(@param1: red, blue); // 传递了1个变量
```

上面所说的css list典型的应用是用于transition这个属性：

```css
.transition(@property: all, @time: 1s, @timing: ease-in-out) {
  -moz-transition: @property @time @timing;
  -webkit-transition: @property @time @timing;
  -o-transition: @property @time @timing;
  transition: @property @time @timing;
}

a {
  // 如果要对color和opacity这两个属性运用transition，必须这样
  .transition(color, opacity; .5s);
  
  // 如果像下面用逗号分隔会产生与期待不符的结果
  // .transtion(color, opacity, .5s);
}
```

上面编译出来的两个结果分别是：

```css
/* 分号调用方式：*/
a {
  -moz-transition: color, opacity 0.5s ease-in-out;
  -webkit-transition: color, opacity 0.5s ease-in-out;
  -o-transition: color, opacity 0.5s ease-in-out;
  transition: color, opacity 0.5s ease-in-out;
}

/* 逗号调用方式（错误）：*/
a {
  -moz-transition: color opacity 0.5s;
  -webkit-transition: color opacity 0.5s;
  -o-transition: color opacity 0.5s;
  transition: color opacity 0.5s;
}
```

mixin还可以进行重载（类似函数重载），调用的时候会匹配所有可以匹配的mixin，把它们的规则都应用进来：

```css
.mixin(@color) {
  color-1: @color;
}
.mixin(@color; @padding: 2) {
  color-2: @color;
  padding-2: @padding;
}
.mixin(@color; @padding; @margin: 2) {
  color-3: @color;
  padding-3: @padding;
  margin: @margin @margin @margin @margin;
}
.some .selector div {
  .mixin(#008000);
}
```

编译出来的css代码：

```css
.some .selector div {
  color-1: #008000;
  color-2: #008000;
  padding-2: 2;
}
```

mixin在声明的参数列表里指定了参数名称，在调用的时候如果使用带参数名的调用方式，则不用考虑参数顺序：

```css
.mixin (@width, @style, @color) {
  border: @width @style @color;
}

// 调用的时候打乱参数顺序
.a {
  .mixin(@color: red; @width: 1px; @style:solid);
}
```

## 5.6 特殊的@arguments变量

@arguments包含所有调用mixin时传递进来的参数，包括默认参数，如：

```css
.box-shadow(@x: 0; @y: 0; @blur: 1px; @color: #000) {
  -webkit-box-shadow: @arguments;
     -moz-box-shadow: @arguments;
          box-shadow: @arguments;
}
.big-block {
  .box-shadow(2px; 5px);
}
```

编译成为：

```css
.big-block {
  -webkit-box-shadow: 2px 5px 1px #000;
     -moz-box-shadow: 2px 5px 1px #000;
          box-shadow: 2px 5px 1px #000;
}
```

## 5.7 不定数量参数

``...``用于指代后面省略不定数量参数，从0-N：

```css
.mixin(...) {        // 匹配 0-N 个参数
.mixin() {           // 匹配正好 0 个参数
.mixin(@a: 1) {      // 匹配 0-1 个参数
.mixin(@a: 1; ...) { // 匹配 0-N 个参数
.mixin(@a; ...) {    // 匹配 1-N 个参数
```

另外，如果直接将``...``加在某个参数名的后面，则后面不定数量的参数都保存在这个变量中，这里以``@rest``作为不定数量参数名举个例子：

```css
.mixin(@a; @rest...) {
   // @rest 保存了 @a 之后的所有参数
   // @arguments 仍然是保存了保存的参数，包括@a 和 @rest的值
}
```

## 5.8 mixin的模式匹配

根据传入的值与mixin声明的值是否一致来进行匹配（注意，不是与声明的变量名比较，因为变量名匹配任何值），或者根据传入的参数数量与声明的参数数量进行匹配（即前面讲的mixin重载）

```css
.mixin(dark; @color) { // 注意这里第一个形参是个值，不是变量名
  color: darken(@color, 10%);
}
.mixin(light; @color) { // 注意这里第一个形参是个值，不是变量名
  color: lighten(@color, 10%);
}
.mixin(@_; @color) {
  display: block;
}

// 以下为匹配上面某种模式的调用
@switch: light;

.class {
  .mixin(@switch; #888); // 即mixin(light, #888); 在上面找light的匹配mixin
}
```

编译结果为：

```css
.class {
  color: #a2a2a2; // lighten(#888, 10%)的结果
  display: block;
}
```

## 5.9 像使用函数一样使用mixin

因为mixin里定义的变量在调用mixin的选择器里是可见的，所以可以在调用mixin的地方使用这些变量：

```css
.average(@x, @y) {
  @average: ((@x + @y) / 2);
}

// 调用上面的取平均
.div {
  .average(16px, 50px); // 计算16px和50px的平均值
  padding: @average; // 使用.average这个mixin里的@average变量（作为.average的返回值）
}
```

被调用的mixin里定义的变量不会覆盖调用者的规则里定义的同名变量，但是会覆盖调用者的外面一层定义的同名变量

在mixin A里定义的minxin B可以作为mixin的返回变量使用，例如：

```css
.unlock(@value) { // 外层 mixin
  .doSomething() { // 内层 mixin
    declaration: @value;
  }
}

#namespace {
  .unlock(5); // 解锁内层的 doSomething
  .doSomething(); // 解锁以后内层的mixin就可以使用了
}
```

编译以后的css：

```css
#namespace {
  declaration: 5; 
}
```

# 6. Guards

Guards只能给mixin或者css选择器添加，声明一种匹配条件和策略

## 6.1 mixin guards

这里的mixin guards实际上是给mixin做一个访问控制，类似``if/else``，但是为了保留css声明性语言的特质，以类似``@media``的 media query的语法达到访问控制的效果

一个mixin guards的例子：

```css
.mixin (@a) when (lightness(@a) >= 50%) {   // 使用when关键字声明mixin guards，另外使用了lightness函数取亮度
  background-color: black;
}
.mixin (@a) when (lightness(@a) < 50%) {
  background-color: white;
}
.mixin (@a) {
  color: @a;
}
```

上面的例子同时用到了mixin可以重载的特性，只要是参数可以匹配就会将内容都混入进来，比如以下代码调用上面的mixin：

```css
.class1 { .mixin(#ddd); }
.class2 { .mixin(#555); }
```

上面的代码编译成css为：

```css
.class1 {
  background-color: black;
  color: #dddddd;
}
.class2 {
  background-color: white;
  color: #555555;
}
```

**guard比较运算符** ``>``, ``<``, ``>=``, ``<=``, ``=``

``true``是唯一表示真值的值，其他类型不能通过类型转换为true，比如

```css
.mixin(@val) when (@val = true) {
  color: red;
}
.mixin(@val) when (@val) {
  color: blue;
}

.some-class {
  .mixin(1);  // 1不是true
  .mixin(true); // 符合guard的条件
}
```

还可以在参数之间进行比较：

```css
.max (@a; @b) when (@a > @b) { width: @a }
.max (@a; @b) when (@a < @b) { width: @b }

.some-class {
  @a: 100;
  @b: 200;
  .max(@a, @b);
}
```

**guard逻辑运算符** ``and``, ``,``逗号（相当于or运算）, ``not``

几个例子：

```css
.mixin (@a) when (isnumber(@a)) and (@a > 0) { ... }
.mixin (@a) when (isnumber(@a)) and (@a > 10, @a < -10) { ... }
.mixin (@a) when not (isnumber(@a)) { ... }
```

**类型检查函数**

基本类型检查函数：

* ``iscolor``
* ``isnumber``
* ``isstring``
* ``iskeyword``
* ``isurl``

数值单位检查：

* ``ispixel``
* ``ispercentage``
* ``isem``
* ``isunit``

**default()** 默认处理

default函数用于处理匹配不到特定条件的情况，这是可以使用默认的样式，类似if/else里else的情况，或者switch/case里的default

```css
.mixin (@a) when (@a > 0) { ...  }
.mixin (@a) when (default()) { ... } // 匹配第一条条件没有匹配上的情况，即@a <= 0的情况
```

## 6.2 css guards

可以给css选择器添加guards，和给mixin添加guards的方式差不多，在1.5.0之前给css添加guards的方式和1.5.0及后续版本方式略有不同，新版本内部实现了一个语法糖，给css选择器建立一个立即执行的mixin

1.5.0之前的方式：（其实就是声明一个mixin，并在声明后面立即调用一次）

```css
.my-optional-style() when (@my-option = true) {
  button {
    color: white;
  }
}
.my-optional-style();
```

1.5.0的方式：（内部实现成一个立即执行mixin）

```css
button when (@my-option = true) {
  color: white;
}
```

如上面代码所示，这样mixin guards和css guards的声明方式就统一了

结合父选择器&可以有效组织多个guards

```css
button {
  & when ( ... ) { ... }
  & when ( ... ) { ... }
  // ... 继续添加
}
```

# 7. 循环调用

使用guards可以创造一种循环调用，在mixin的内部调用mixin自身，比如：

```css
.loop(@n) when (@n > 0) {
  .loop((@n - 1));
  width: (10px * @n);
}

// 调用
.some-class {
  .loop(10);
}
```

编译出来的css：

```css
.some-class {
  width: 10px;
  width: 20px;
  width: 30px;
  width: 40px;
  width: 50px;
  width: 60px;
  width: 70px;
  width: 80px;
  width: 90px;
  width: 100px;
}
```

一个常见的例子是创造一系列宽度的column：

```css
.generate-columns(@n, @i:1) when (@n >= @i) {
  .column-@{i} {
    width: (@i * 100% / @n);    // 这里也可以改成 width: percentage(@i / @n);
  }
  .generate-columns(@n, (@i+1));
}

// 初次调用
.generate-columns(4);
```

编译结果：

```css
.column-1 {
  width: 25%;
}
.column-2 {
  width: 50%;
}
.column-3 {
  width: 75%;
}
.column-4 {
  width: 100%;
}
```

# 8. 独立规则集

独立规则集（不带css选择器）可以赋值给某个变量，作为参数传递给某个需要接受参数的mixin，或者直接当做一个mixin被调用。独立规则集的内部可以包含css规则、内嵌的规则以及media query等内容。

## 8.1 作为变量

将一组规则集声明为某个变量的值（也可以说这就是一个独立规则集的定义方式），然后在其他地方调用该值（像调用mixin一样调用独立规则集）

```css
// 作为变量的独立规则集
@detached-ruleset: {
  background: red;
};    // 注意，这里的分号一定不能少，否则编译出错

// 在某个css选择器里调用这个规则集
.top {
  @detached-ruleset();  // 注意，这里的括号一定不能少，否则编译出错
}
```

变量赋值语句后面的分号``;``一定不能少，调用变量的时候，后面类似调用mixin的括号``()``一定不能少，否则都会编译出错。

独立规则集除了包含一组规则，还可以包含内嵌的规则集，如：
```css
@my-ruleset: {
  .my-selector {
    background-color: black;
  }
};
```

## 8.2 作为参数传递

```css
.desktop-and-old-ie(@rules) {
  @media screen and (min-width: 1200) { @rules(); }
  html.lt-ie9 &                       { @rules(); } // 这里使用了&符号指代父选择器(后面会做介绍)
}

header {
  background-color: blue;

  // 这里直接将{ background-color: red; }这个独立规则集传递给了mixin
  .desktop-and-old-ie({
    background-color: red;
  });
}
```

``{ background-color: red }``这样的独立规则集就像js的对象字面量一样可以传递给mixin作为实参；这里将特定的media query和某些不兼容的浏览器作为一个整体抽象出来单独设置针对它们的样式，省略了每次都去分别设置media query和不兼容浏览器的麻烦

上面的less编译出来的结果如下：

```css
header {
  background-color: blue;
}
@media screen and (min-width: 1200) {
  header {
    background-color: red;
  }
}
html.lt-ie9 header {
  background-color: red;
}
```

## 8.3 结合media query冒泡

```css
@my-ruleset: {
  .my-selector {
    @media tv {
      background-color: black;
    }
  }
};

@media (orientation:portrait) {
    @my-ruleset();
}
```

编译出来的css如下；

```css
@media (orientation: portrait) and tv {
  .my-selector {
    background-color: black;
  }
}
```

独立规则集内部的.my-selector选择器内嵌的media query ``@media tv``在独立规则集被调用的地方冒泡到外层和``@media (orientation: portrait)``组合成``@media (orientation: portrait) and tv``

## 8.4 调用返回值

前面说到调用mixin的返回值是该内部mixin的变量或者内嵌mixin（将这些变量或内嵌mixin解锁变为可用），而**独立规则集在调用的时候只返回内部的mixin，而不返回其内部定义的变量**

```css
// 定义一个独立规则集，内部包含一个内嵌mixin和一个变量
@detached-ruleset: { 
  @color: black;
  .mixin() {
      color: @color;
  }
};

// 调用该独立规则集，规则集内的mixin可用，而变量则不能使用（没有被返回）
.caller {
  @detached-ruleset(); // 调用独立规则集是不调用内部mixin的，但是会让内部mixin变得可用
  .mixin(); 
  background-color: @color; // 编译出错
}
```

## 8.5 规则作用域

独立规则集在其定义域内的变量，在独立规则集内部是可见的（独立规则集获得了该作用域的访问权）；在独立规则集的调用域内变量也是可见的，但是优先级低于定义域的变量

```css
@detached-ruleset: {
  caller-variable: @callerVariable; // 变量callerVariable在定义域里没有定义
  .callerMixin(); // callerMixin在定义域里没有定义
};

selector {
  // 使用独立规则集
  @detached-ruleset(); 

  // 上面的变量和mixin在调用域里定义了
  @callerVariable: value;
  .callerMixin() {
    variable: declaration;
  }
}
```

上例中独立规则集里用到的变量和mixin是在调用域里定义的，它们在独立规则集的内部是可见的。

```css
@variable: global;
@detached-ruleset: {
  // 使用全局的@variable即global，因为其在detached-ruleset定义时可见
  variable: @variable; 
};

selector {
  @detached-ruleset();
  @variable: value; // 调用域的@variable会被忽略，因为独立规则集的定义域已经有可见的@variable了
}
```

上例中定义域已经有@variable被定义可使用了，所有在调用域里定义的@variable会被直接忽略掉

在引用独立规则集时（如``@ruleset-a: @ruleset-b``定义@ruleset-a为@ruleset-b的引用），独立规则集不会获取该作用域的访问权；只有在独立规则集被定义或调用时，才会获得当前作用域的访问权

```css
@ruleset-1: { scope-detached: @one @two; };
.one {
  @one: visible;
  .two {
    @ruleset-2: @ruleset-1; // copying/renaming ruleset，这时的作用域并不会被获取到 
    @two: visible;
  }
}

.usePlace {
  .one > .two();  // 这一步让@two和@ruleset-2在此作用域可见
  @ruleset-2(); // 由于@one不可见，所以报错
}
```

上例的例子，在``@ruleset-2: @ruleset-1;``的时候没有获取到@one所在的作用域，而在.usePlace中也没有调用``.one()``，所以@one在调用@ruleset-2时是不可见的，因此编译会报错，如果在.usePlace中加上一个``.one();``调用语句，则编译没有问题，编译结果为：

```css
@ruleset-1: { scope-detached: @one @two; };
.usePlace {
  scope-detached: visible visible;
}
```

**解锁**的概念：调用一个mixin会将该mixin内的变量、mixin以及独立规则集解锁，在调用的作用域里变得可见：

```css
#space {
  .importer1() {
    @detached: { scope-detached: @variable; }; // 定义独立规则集
  }
}

.importer2() {
  @variable: value; // 在这个作用域里解锁的独立规则集可以看见这个变量
  #space > .importer1(); // 解锁mixin importer1，即解锁该mixin里的内容，包括独立规则集@detached
}

.usePlace {
  .importer2(); // 解锁mixin import2，同时解锁mixin importer1
   @detached(); // 调用已解锁的@detached独立规则集
}
```

以上less编译为css：

```css
.usePlace {
  scope-detached: value;
}
```

# 9. 嵌套规则（Nested Rules）和父选择器

## 9.1 嵌套规则

嵌套规则模仿html的标签嵌套结构，将选择器按层级进行嵌套：

```css
.parent {
  color: @parent-color;
  .child {
    color: @child-color;
  }
}
```

## 9.2 父选择器

``&``符号表示父选择器（用在嵌套规则中，常用于表示伪类），如：

```css
a {
  color: #000;
  // 注意，如果这里不写&表示的意义将大不同
  &:hover {
    color: #fff;
  }
}
```

另一个常用的场景是用``&``做父类的类名的替换：

```css
.btn {
  &-primary {
    background-color: blue;
  }
  &-success {
    background-color: green;
  }
}
```

**注意：加入``&``的选择将不被视为内嵌的选择器，而是和父选择器平级**；上面的例子即内层选择器``&-primary``的规则编译结果不会变成``.btn .btn-primary``，而是``.btn-primary``，与``.btn``是同一层级的

**多个&：**子选择器中还可以反复使用``&``，如``& > &``或者``& + &``等，其实相当于就是变量替换，只不过只能表示替换为父选择器的名字

另外注意``&``表示所有父选择器及父选择器之间的层级关系，比如在``.grand``中嵌套一个``.parent``，在``.parent``中引用的``&``将被替换为``.grand .parent``

**组合（爆发）**对于逗号分隔的父元素，``&``将代替逗号分隔的选择器其中的任何一个，如：

```css
p, a, ul, li {
  & + & {
    border: 1px solid blue;
  }
}
```

这里``& + &``会被编译为``p + a, p + ul, ...``这4x4=16种组合，会给这16种组合都应用指定规则

## 9.3 名字空间和访问器（Namespaces & Accessors）

（不要将它与[CSS ``@namespace``](http://www.w3.org/TR/css3-namespace/)以及[namespace选择器](http://www.w3.org/TR/css3-selectors/#typenmsp)搞混了）

引入某个mixin时，如果mixin在另一个选择器内部，可以将该选择器作为名字空间，将mixin作为该名字空间里的mixin引入，如：

```css
/*非#bundle空间内定义的.button规则*/
.button {
  border: none;
}

/*在#bundle空间内定义的.button规则*/
#bundle {
  .inner {
    .button {
      display: block;
      border: 1px solid black;
      background-color: grey;
    }
    &:hover {
      background-color: white
    }
  }
}

#header a {
  color: orange;
  #bundle > .inner > .button();
}

#header a {
  color: orange;
  #bundle > .inner > .button();
}
```

上面的``#bundle > .inner > .button()``也可以去掉``>``或``()``，如``#bundle .inner .button;``

这里的名字空间是二层，一般根据需要可以是任意层

**注：如果不是引用该名字空间的mixin ``#bundle .inner .button``，而是直接``.button()``则优先使用外层定义的``.button``的规则，即``border:none;``**

另外名字空间中（外层选择器中）定义的变量``@var``不能被用在该名字空间以外

**注：名字空间和访问器仅能用于mixin，不能用于独立规则集**，内嵌独立规则集有另外的引入方式：先通过``.outter();``或``#inner();``的方式（调用mixin）解锁内嵌的@inner独立规则集（变量），然后再调用它``@inner();``；如果不想同时引入outter里的规则，可以在定义outter时将其声明为不输出的mixin

## 9.4 作用域Scope

以上所谓名字空间有其对应的作用域Scope，与js中的作用域概念类似。在查找变量或mixin的时候，会沿着作用域链依次向上查找（本作用域没有，则查找父作用域），如：

```css
@my-color: red;

.some-class {
  @my-color: blue;
  #header {
    color: @my-color; // 结果为blue
  }
}
```

## 9.5 MediaQuery的嵌套和冒泡

@media也可以嵌套，如下：

```css
.some-class {
  @media screen {
    /*...*/
    @media (min-width: 768px) {
      /*...*/
    }
  }

  @media tv {
    /*...*/
  }
}
```

编译成css的结果如下：

```css
@media screen {
  /*...*/
}

@media screen and (min-width: 768px) {
  /*...*/
}

@media tv {
  /*...*/
}
```

之所以叫**media query冒泡**，如上所例子所示，内嵌的@media在编译成css的时候从内部冒泡到外层，作为外层@media的交集的一部分，通过``and [...]``的方式，将less层面的嵌套式media query变成了css的扁平的无交集的media query；这样内嵌的media query总是会冒泡到顶层来

# 10. 继承（Extend）

相比翻译成**扩展**我更喜欢把这里extend翻译成**继承**，与OOC  SS的理念更契合，也很好理解

> extend是一个Less伪类，它会合并它所在的选择器和它所匹配的引用。

如果``.a:extend(.b)``，那么.b的选择器会变为``.b, .a {...}``，这就是extend的编译方式

比如这段代码：

```css
nav ul {
  &:extend(.inline);
  background: blue;
}

.inline {
  color: red;
}
```

也可以使用这种继承形式：

```css
nav ul:extend(.inline) {
  background: blue;
}

.inline {
  color: red;
}
```

两者效果是一样的，编译出来的结果如下：

```css
nav ul {
  color: blue;
  background: blue;
}
.inline,
nav ul {
  color: red;
}
```

## 10.1 all关键字用于继承

加上``all``关键字表示欲继承的类的所有实例（这里实例的概念指.classA.classB是.classA的一个实例），如``:extend(.b all)``表示``:extend(.x.b)``、``:extend(.y.b)``等等

```css
.c:extend(.d all) {
  // 匹配".d"的所有实例，比如".x.d"或者".d.x"
}
.c:extend(.d) {
  // 匹配".d"选择器的唯一实例
}
```

注意**extend all**的工作方式是**部分替代**式的，即如果``.cls:extend(.a all)``作用到``.a.b``上，并不是粗暴的添加``.cls``到``.a.b``的规则集上去（``.a.b, .cls { ... }``）而是将``.a.b``中的``.a``替代为``.cls``再添加到``.a.b``规则集上去，即``.a.b, .cls.b { ... }``，例子如下：

```css
.a.b {
  color: orange;
}

.cls:extend(.a all) {
  border: 1px solid black;
}
```

编译结果为：

```css
.a.b,
.cls.b {  // 注意新加的选择器是对.a.b中的.a进行部分替换的结果
  color: orange;
}
.cls {
  border: 1px solid black;
}
```

## 10.2 多重继承

两种形式，一种使用``,``逗号分隔，一种使用多个``:extend``

```css
.e:extend(.f) {}
.e:extend(.g) {}

// 上面的代码与下面的做一样的事情
.e:extend(.f, .g) {}

// 上面的代码还有下面这种形式，效果都是一样的
.e:extend(.f):extend(.g) {}
```

**注意** ``:extend()``后面不能再跟上其他伪类了，但是第一个:extend之前可以有其他伪类，比如``:hover:extend(.a)``

## 10.3 继承嵌套的选择器

```css
.parent {
  tr {    // 扩展目标
    color: blue;
  }
}

.some-class {
  &:extend(.parent tr);
}
```

编译结果为：

```css
.bucket tr,
.some-class {
  color: blue;
}
```

## 10.4 精确匹配

匹配的精确性体现在多个方面：

* 体现在类选择器上，``:extend(.a)``不会匹配到``.a.b``，也不会匹配到``*.a``或者``div.a``，只会匹配到单独的``.a``

* 体现在伪类的顺序上，比如``:extend(a:visited:hover)``和``:extend(a:hover:visited)``就是不同的，虽然``a:visited:hover``和``a:hover:visited``选择的元素是一致的

* nth表达式，如``:extend(:nth-child(n+3))``就不能匹配到``:nth-child(1n+3)``，虽然``:nth-child(n+3)``和``:nth-child(1n+3)``是一致的

在属性选择器的引号上则比较宽松，可以是单引号、双引号或者没有引号：

```css
[title=identifier] {
  color: blue;
}
[title='identifier'] {
  color: blue;
}
[title="identifier"] {
  color: blue;
}

.noQuote:extend([title=identifier]) {}
.singleQuote:extend([title='identifier']) {}
.doubleQuote:extend([title="identifier"]) {}
```

编译结果：

```css
[title=identifier],
.noQuote,
.singleQuote,
.doubleQuote {
  color: blue;
}
[title='identifier'],
.noQuote,
.singleQuote,
.doubleQuote {
  color: blue;
}
[title="identifier"],
.noQuote,
.singleQuote,
.doubleQuote {
  color: blue;
}
```

## 10.5 当带有变量的选择器名遇到继承（Selector Interpolation with Extend）

可以给带有变量（变量插值）的选择器名添加``:extend``伪元素

但是extend()里面传递的参数不能是变量，extend()匹配的选择器名也不能是变量（匹配不到）

看下面的例子就好理解了：

```css
// 第一种，要匹配的目标选择器带变量插值
@variable: .bucket;
@{variable} { // interpolated selector
  color: blue;
}
.some-class:extend(.bucket) {} // does nothing, no match is found
```

```css
// 第二种，匹配所传递的选择器参数是变量插值
.bucket {
  color: blue;
}
.some-class:extend(@{variable}) {} // interpolated selector matches nothing
@variable: .bucket;
```

```css
// 第三种，要进行extend的选择器带变量插值
.bucket {
  color: blue;
}
@{variable}:extend(.bucket) {}
@variable: .selector;
```

只有第三种扩展是可以匹配到的

## 10.6 当@media遇到extend

在@media声明块里面的extend只匹配该@media内部的选择器，如：

```css
@media print {
  .screen-class:extend(.selector) {}
  // 上面的:extend只匹配下面的这个.selector
  .selector {
    color: #000;
  }
}
// 不会匹配外面这个.selector
.selector {
  color: #f7f7f7; 
}
```

在@media声明块的里的extend也不匹配内嵌@media块里的选择器，如：

```css
@media screen {
  .screen-class:extend(.selector) {}
  @media (min-width: 1023px) {
    .selector { // 上面的:extend匹配不到这里的.selector
      color: #000;
    }
  }
}
```

编译成css是这样的：

```css
@media screen and (min-width: 1023px) {
  .selector {
    color: #000;
  }
}
```

在@media外面的extend会匹配到@media内部所有可匹配的选择器，如：

```css
@media screen {
  .selector {
    color: #fff;
  }
  @media (min-width: 1023px) {
    .selector { // 上面的:extend匹配不到这里的.selector
      color: #000;
    }
  }
}

// top-level会继承@media中所有.selector的规则
.top-level:extend(.selector) {}
```

编译成css是这样的：

```css
@media screen {
  .selector,
  .top-level {
    color: #fff;
  }
}
@media screen and (min-width: 1023px) {
  .selector,
  .top-level {
    color: #000;
  }
}
```

## 10.7 Duplication Detection

less目前还没有做重复检测，所以继承的时候如果有重复的话不会自动清除，例如：

```css
.alert-info,
.widget {
  /* declarations */
}

.alert:extend(.alert-info, .widget) {}
```

这里``.alert-info``, ``.widget``的规则是一样的，但是``.alert``仍然会重复继承两者的规则，编译成css如下：

```css
.alert-info,
.widget,
.alert,
.alert {
  /* declarations */
}
```

## 10.8 应用场景

和子类继承父类的应用场景相同，选择器的继承就是为了获取所继承的选择器的规则，并有选择的进行改写。但是改写的话因为css后声明的规则覆盖前声明的规则，所以子类应该放在父类的下面，如下所示（html结构为``<a class="bear">Bear</a>``）：

```css
.animal {
  background-color: black;
  color: white;
}
.bear {
  &:extend(.animal);
  background-color: brown;
}
```

另一个场景是作为mixin的代替方法，可以用于某些mixin解决不了或者效果不好的场合：extend的实现方式是用逗号分隔开那些重复规则的选择器，可以比使用mixin生成更少的css代码；extend的选择器可以是复杂的选择器，比如``li.list``这样的，而mixin的选择器只能是id选择器或者类选择器（以及前面带名字空间修饰，比如``#some-id > mixin``）

# 11. 合并(Merge)

像transition、background这样的属性的值是由多个属性合并起来的值列表，并且像box-shadow这样的属性还可以用逗号``,``分隔多个值列表，合并提供一种手段方便向这些属性列表添加新的值，可以是逗号``,``分隔，也可以是空格分隔

## 11.1 逗号分隔

在需要逗号分隔的属性后面加一个加号``+``方便后续添加更多值列表

```css
.mixin() {
  box-shadow+: inset 0 0 10px #555;
}
.myclass {
  .mixin();
  box-shadow+: 0 0 20px black;
}
```

编译结果：

```css
.myclass {
  box-shadow: inset 0 0 10px #555, 0 0 20px black;
}
```

## 11.2 空格分隔

在需要空格分隔的属性后面加``+_``方便后续添加更多值

```css
.mixin() {
  transform+_: scale(2);
}
.myclass {
  .mixin();
  transform+_: rotate(15deg);
}
```

编译结果：

```css
.myclass {
  transform: scale(2) rotate(15deg);
}
```

# 12. 导入（Importing）

可以在less中导入另一个``.less``文件，然后这个文件中的所有变量都可用了，另外导入less文件时``.less``扩展名可以省略，如：

```css
@import "library"; // library.less
@import "typo.css";
```

css的@import语句必须放在所有css规则的顶部，然而less没有这个要求

## 12.1 扩展名

* ``.less``文件作为less文件引入
* ``.css``文件作为css导入
* 其他扩展名/没有扩展名均作为less文件导入

作为css文件导入时，不会对@import语句做处理，而是将该语句直接输出到最终css文件中，让浏览器去做处理（即**as-is**的方式）

## 12.2 导入选项

为了给予开发者更多的导入选择，使用``@import (<keyword>) "filename";``语句导入less/css

可用的选项关键词如下：

* ``reference`` 导入less文件，但是不输出其样式
* ``inline`` 导入文件但是不处理，直接输出，一般用来导入一些less不兼容的css（不标准的，或者hacked css）
* ``less`` 将导入的文件当做less处理，不管扩展名是什么
* ``css`` 将导入的文件当做css处理，不管扩展名是什么
* ``once`` 只引入一次（默认行为）该文件
* ``multiple`` 多次引用该文件
* ``optional`` 当文件未找到时继续编译

## 12.3 reference

用法：``@import (reference) "foo.less"``

引入的less文件中的规则集不会直接输出到output，而只能作为mixin或者被继承（:extend）

## 12.4 inline

用法： ``@import (inline) "not-less-compatible.css"``

一般用来导入一些less不兼容的css文件，less只兼容常用的标准css，有些地方的注释已经一些css hack是less处理不了的，因此这些css可以使用inline直接导入并输出到最终css文件中

## 12.5 less

用法： ``@import (less) "foo.css"``

把``foo.css``当做less引入进来

## 12.6 css

用法： ``@import (css) "foo.less"``

上面的语句会直接编译成如下css输出到最终的样式表中：

```css
@import "foo.less";
```

## 12.7 once

用法： ``@import (once) "foo.less"``

只引入一次，第二次声明引入该文件会自动忽略；这是@import语句的默认行为


## 12.8 multiple

用法：``@import (multiple) "foo.less"``

允许多次引入同一文件

## 12.9 optional

用法：``@import (optional) "foo.less"``

当使用optional时，该引入文件是可选的，若文件存在则引入之，若不存在也不会报错，而是直接忽略之



