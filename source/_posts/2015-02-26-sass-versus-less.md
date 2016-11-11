---
layout: post
title:  "SASS/LESS语法简明对比"
date:   2015-02-26 14:36:00
categories: css
tags:
  - css
  - sass
  - scss
  - less
  - 样式表
  - css预处理
  - fe
  - 前端
  - front-end
  - 前端开发
author: "_danZ"
---

# 前言

网页的布局和样式是由css来决定的，当网站规模比较大元素比较多的时候，css的量级也会变得很庞大，css代码将变得难以维护，于是就出现了css预处理器，通过加入一些类似编程语言的特性，让编写css变得更高效更易于维护。和less一样，sass也是一种流行的css预处理器。sass总体上和less差不多，但是比less使用者更多，因为其对css兼容性更好。

<!--more-->

以下结合sass官方文档和less做一个简单的对比，每一小节如果两者用法有区别则用表格列出方便对比。

# 运行sass

首先推荐一个[sass在线编译器](http://sassmeister.com/)，可以用这个做测试

## 命令行

在本地命令行里运行sass，需要首先安装ruby（这里就不多做介绍了），ruby装好用gem安装sass包：

```shell
gem install sass
```

然后就可以使用sass了，比如：

```shell
sass foo.scss foo.css
```

上面的命令将``foo.scss``文件编译为``foo.css``样式文件；文件后缀名scss意思是sassy css，强调sass对css代码的完全兼容

sass的编译选项：``nested``（默认值）、``expanded``、``compact``、``compressed`` （可以望文生义一下）

设置编译选项：

```shell
sass --style compressed foo.sass foo.css
```

可以自动检测文件或文件夹变动并生成编译后的css

自动检测文件：

```shell
sass --watch foo.scss:foo.css
```

自动检测文件夹：

```shell
sass --watch app/sass:app/css
```

sass语法和scss语法可以互相转换：

```shell
sass-convert style.sass style.scss
```

```shell
sass-convert style.scss style.sass
```

# sass和scss

sass语法写起来更简洁，去掉了大括号和分号，而使用缩进和换行来处理语义间隔；这点类似coffeeScript或者python，看下面的sass的例子：

```css
.some-class
  color: red
  background-color: #fff
```

scss全称是sassy css，写起来跟更接近css，除此之外它和sass没有区别；上面的代码用scss写出来是这样的：

```css
.some-class {
  color: red;
  background-color: #fff;
}
```

可以看出，在没使用sass的扩展特性情况下，scss写法和css没有区别

**以下不做特别说明则sass均使用scss的书写方式**

# 变量

![变量](http://7u2loa.com1.z0.glb.clouddn.com/lessnsass-bianliang.png)

变量定义：``$my-color: #fff;`` （less用@定义和调用变量）

变量调用：

```css
div {
  color: $my-color;
}
```

## 作为字符串

变量用于字符串，使用#{$var-name}调用，和less的``@{var-name}``类似：

```css
$my-attr: "color";

// 使用变量
div {
  #{$my-attr}: #fff;
}
```

注意上面的``#{$my-attr}``会被编译为``color``，但如果是less则会保留``"color"``的双引号，变成``"color": #fff``

## 变量作用域

变量作用域是和层级相关的，内嵌的变量不能被外层的变量看到；除非为内嵌的变量加上``!global``全局声明，如：

```css
#mine {
  $width: 5em !global;    // 全局变量
  width: $width;
}

#div {
  width: $width;
}
```

## 设置默认值

使用``!default``为变量设置默认值，这样如果前面有赋值则不会使用该值，如果没有赋值则使用该值作为变量的值

```css
$content: '1st val';
$content: '2nd val' !default;
$new-content: '3rd val' !default;
```

# 注释

和less一样，可以使用css的块注释``/* comments */``，也可以使用一般编程语言中都用到的行注释``// commments``

块注释会被输出到css中，行注释不会

块注释不会被输出到compressed css中，除非块注释的第一个字符是``!``，如：

```css
/*! this is final output */
```

# 运算

运算和less没有区别

```css
div {
  padding: (5px * 2);
  width: (100px + 400px);
}
```

# 父元素和嵌套

![父元素和嵌套](http://7u2loa.com1.z0.glb.clouddn.com/lessnsass-fuyuansu.png)

## 父选择符&

和less一样，sass允许层级嵌套，并且可以引用父元素；在less中引用父元素使用``&``，在sass中引用父元素也是用``&``

&在less和sass中使用上的区别如上表所描述的，less可以将&插入到一个名字中的任何位置：

```css
.some-class {
  my-&-free: {
    color: red;
  }
}
```

上面这样的事情sass是做不到的，sass中&只能作为名字的前缀：``&-some-content``，而不能是``foo-&-bar``

## 合并值

对于font这样的符合属性，可以使用namespace font来实现

```css
.some {
  font-family: fantasy;
  font-size: 30em;
  font-weight: bold;
}
```

用sass可以这样写：

```css
.some {
  font {
    family: fantasy;
    size: 30em;
    weight: bold;
  }
}
```

名字空间也可以是已经赋了值的：

```css
.some {
  font: fantasy, 30em {
    weight: bold;
  }
}
```

上面的形式编译成css是这样的（保留已赋值部分，块里面的属性则每条单独用font-xxx的方式添加）：

```css
.some {
  font: fantasy, 30em;
  font-weight: bold;
}
```

## @midia嵌套

和less一样，@media可以嵌套在css规则集内部，也可以嵌套在另一个@media的里面（类似less的@media冒泡）

# 继承

![继承](http://7u2loa.com1.z0.glb.clouddn.com/lessnsass-jicheng.png)

sass和less都包含选择器继承，但是用法上稍有区别；less使用``:extend()``伪类作为继承方式，而sass中使用``@extend xxx;``语句

## 仅用于继承的选择器：place holder selectors

仅用于继承的选择器，不作为css输出（less中有仅用于继承的mixin）：

```css
.some-ct %some-class {
  color: red;
}

.mine {
  @extend %some-class;
}
```

编译成css为：

```css
.some-ct .mine {
  color: red;
}

```

## less和sass在继承内嵌选择器上的区别

less可以继承复杂选择器，而sass的继承对象不能是复杂选择器如``.a .b``或者``.a + .b``，因此less和sass在内嵌选择器的匹配规则上区别较大：

less在继承上的匹配是精确匹配，继承的时候必须指定继承的内嵌选择器的名字空间（父选择器）；而sass在匹配上则是尽量满足更多的可匹配选择器，包括自动去内嵌选择器中去寻找可能的匹配，比如：

```css
.some-ct .some-class {
  color: red;
}

.mine {
  @extend .some-class;
}
```

编译为：

```css
.some-ct .some-class, .some-ct .mine {
  color: red;
}
```

自动匹配了内嵌的``.some-class``，并做了替代。注意这里``@extend .some-class``如果放在less里（``:extend(.some-class)``）是匹配不到``.some-ct .some-class``的，只有``:extend(.some-ct .some-class)``才可以匹配到。而在sass里只能采取这种方式（``@extend .some-class;``而不是``@extend .some-ct .some-class``），原因是sass里不能继承``.a .b``或``.a + .b``这种带层级的选择器，因此如果用``@extend .some-ct .some-class``就会报错。

```css
.some-ct .some-class {
  color: red;
}

.mine {
  @extend .some-ct .some-class;
}
```

上面的代码会直接报错：``can't extend nested selectors``

另外，当继承目标选择器的时候，自身是在一个内嵌选择器序列中，则如果继承目标也是在内嵌序列中，则会发生**合并序列**，如下：

```css
#admin .tabbar a {
  font-weight: bold;
}
#demo .overview .fakelink {
  @extend a;
}
```

编译后：

```css
#admin .tabbar a,
#admin .tabbar #demo .overview .fakelink,
#demo .overview #admin .tabbar .fakelink {
  font-weight: bold; }
```

如果两者序列中包含重合的部分，则会合并重合的部分：

```css
#admin .tabbar a {
  font-weight: bold;
}
#admin .overview .fakelink {
  @extend a;
}
```

编译后：

```css
#admin .tabbar a,
#admin .tabbar .overview .fakelink,
#admin .overview .tabbar .fakelink {
  font-weight: bold; }
```

## 多重继承

```css
@extend .a;
@extend .b;

// or
@extend .a, .b;
```

## !optional

继承出错时可以避免报错：如当一个a元素去继承一个h1.xxx的元素，这时会因为a和h1冲突无法继承而报错；加上``!optional;``可以避免报错。

```css
#context h1.notice {
  color: blue;
  font-weight: bold;
  font-size: 2em; }

a.important {
  @extend .notice !optional;
}
```

## 在media内部使用继承

和less一样，在@media内部使用@extend只能匹配到该media块内部的选择器，而不能匹配到外部的选择器

## @at-root指令

将@at-root { ... }的规则放到根层级去

* @at-root { ... }
* @at-root (without: media) { ... } 放到@media指令外面去
* @at-root (without: media supports) { ... } 放到@media和@supports指令外面
* @at-root (without: rule) { ... } 不带上父选择器
* @at-root {with: rule} { ... } 带上父选择器
* @at-root (without: all) { ... } 放到所有指令和规则外

# mixin

![mixin](http://7u2loa.com1.z0.glb.clouddn.com/lessnsass-mixin.png)

less中只要是简单类选择器或者id选择器就可以拿来作为mixin在别的地方调用，调用方式也是简单的``mixin();``，而sass中使用了两个关键字，``@mixin``和``@include``，其中``@mixin``用于声明一个mixin，而``@include``用于调用一个mixin

和less一样，sass也可以向mixin传递参数，也可以设置默认参数值

## 独立规则集

向mixin传递的参数为独立规则集时，less和sass有点区别，less是当做一个参数传递的，在定义mixin时需要声明一个形参，并在mixin内部调动该形参即可，比如：

```css
.desktop-and-old-ie(@rules) {
  @media screen and (min-width: 1200) { @rules(); }
  html.lt-ie9 &                       { @rules(); }
}

header {
  background-color: blue;

  // 这里直接将{ background-color: red; }这个独立规则集传递给了mixin
  .desktop-and-old-ie({
  background-color: red;
  });
}
```

而同样的.desktop-and-old-ie(@rules)的在sass里是这样声明的：

```css
@mixin desktop-and-old-ie {
  // 使用@content调用独立规则集
  @media screen and (min-width: 1200) { @content; }
  html.lt-ie9 &                       { @content; }
}

header {
  background-color: blue;

  // 这里直接将{ background-color: red; }这个独立规则集传递给了mixin
  // 注意和less的区别，没有括号
  @include desktop-and-old-ie {
  background-color: red;
  };
}
```

注意传递独立规则集给mixin的时候，该独立规则集（这里是延用less的名词，在sass里这个叫做**block content**）内部的变量和mixin的作用域是定义该独立规则集的外围作用域，例如：

```css
$color: white;
@mixin colors($color: blue) {
  background-color: $color;
  @content;
  border-color: $color;
}
.colors {
  @include colors { color: $color; }
}
```

这里``@include colors { color: $color; }``的$color不会去找mixin内部的``$color:blue``而是找独立规则集外围的作用域``$color:white``，因此编译出来的css如下：

```css
.colors {
  background-color: blue;
  color: white;
  border-color: blue;
}
```

# 函数

![函数](http://7u2loa.com1.z0.glb.clouddn.com/lessnsass-function.png)

## 预置函数

less和sass中都提供了各种预置函数，比如``lighten()``，``darken()``等，调用的方法也是一样的，直接调用就可以了，比如：

```css
div {
  color: lighten(#000, 10%);
}
```

## 自定义函数

less使用mixin达到和函数类似的效果，而sass使用``@function``关键字来定义自定义函数：

```css
@function sum($a, $b) {
  @return $a + $b;
}

.some-class {
  $a: 10px;
  $b: 20px;
  width: sum($a, $b);
}
```

注意``@function``中使用``@return``返回结果

上面的例子在less中可以这样实现：

```css
.sum(@a, @b) {
  @sum: @a + @b;
}

.some-class {
  @a: 10px;
  @b: 20px;
  .sum(@a, @b);
  width: @sum;
}
```

建议：在定义函数时为了方便与变量或者mixin区别开来（避免名字冲突），可以给函数加前缀，比如你是3dobe公司的，定义sum函数，可以使用``-3dobe-sum``作为函数名

## 函数的参数

调用的时候使用命名参数方式调用，则参数顺序可以打乱，如``myFunc($var2: val2, $var1: val1);``；这点和less的mixin调用里明确参数名字的调用方式是一致的

# 导入（import）

和sass一样，也是使用``@import``来导入文件

## 四种直接编译为css @import的情况

* 导入文件后缀名为.css
* 文件路径以http://开头
* 文件路径为url(...)
* @import声明带有media query

这四种形式分别如下所示：

```css
@import "foo.css";
@import "http://foo.com/bar";
@import url(foo);
@import "foo" screen;
```

编译后：

```css
@import url(foo.css);
@import "http://foo.com/bar";
@import url(foo);
@import "foo" screen;
```

# 条件语句

![条件语句](http://7u2loa.com1.z0.glb.clouddn.com/lessnsass-condition.png)

less使用guards进行条件判断（包括mixin guards和css guards），sass使用``@if``、``@else``语句进行判断：

```css
$test: true;

div {
  @if ($test) {
    color: green;
  } @else {
    color: red;
  } 
}
```

**等于**符号和less的差别：less是``=``，而sass是``==``

# 循环

![循环](http://7u2loa.com1.z0.glb.clouddn.com/lessnsass-loop.png)

less里的循环是使用when判断条件，在mixin里先更新条件然后继续调用自身来实现的，类似一般编程语言中的while结构；而sass里实现循环则使用@for、@while或者@each关键字：

## @for指令

```css
@for $i from 1 through 10 {
  .class-#{$i} {
    background-color: darken(#eee, 10% * $i);
  }
}
```

@for循环有两种格式：

* ``@for $var from <start> through <end> { ... }``
* ``@for $var from <start> to <end> { ... }``

两者区别在于through会包括最后的``<end>``而to则不会包含最后的``<end>``

## @while指令

```css
$i: 0;
@while $i < 10 {
  $i: $i + 1;
  .class-#{$i} {
    background-color: darken(#fff, 10% * $i);
  }
}
```

## @each指令

```css
@each $i in 1,2,3,4,5,6,7,8,9,10 {
  .class-#{$i} {
    background-color: darken(#fff, 10% * $i);
  }
}
```

语法：``@each $var in <list or map> { ... }``

这里``<list or map>``是SassScript的两个类型：

* list：``(item1, item2, item3, ...)``，也可以是多值列表：``(item1a, item1b, item1c), (item2a, item2b, item2c), ...``；list除了用逗号``,``分隔，还可以用空格分隔，如``0 2px 0 2px``，注意空格的优先级高于逗号，如``1 2, 3``相当于``(1 2), 3``

list可以使用list functions做处理

来个多值列表的例子：

```css
@each $animal, $color, $cursor in (puma, black, default),
                (sea-slug, blue, pointer),
                (egret, white, move) {
  .#{$animal}-icon {
    background-image: url('/images/#{$animal}.png');
    border: 2px solid $color;
    cursor: $cursor;
  }
}
```

编译出来的css如下：

```css
.puma-icon {
  background-image: url("/images/puma.png");
  border: 2px solid black;
  cursor: default;
}

.sea-slug-icon {
  background-image: url("/images/sea-slug.png");
  border: 2px solid blue;
  cursor: pointer;
}

.egret-icon {
  background-image: url("/images/egret.png");
  border: 2px solid white;
  cursor: move;
}
```

* map：``(key1: val1, key2: val2, key3, val3)``；map可以用map functions，也可以当做``(key1 val1, key2 val2, key3 val3)``的list，调用list function

比如：

```css
@each $header, $size in (h1: 2em, h2: 1.5em, h3: 1.2em) {
  #{$header} {
    font-size: $size;
  }
}
```

# 其他指令

@debug 输出一个表达式的值，比如``@debug 10em + 12em;``

@warn 输出一条警告信息，比如``@warn "this is a warning"``

@error 抛出一条错误信息，比如``@error "this is a error"``

