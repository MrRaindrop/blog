---
layout: post
title:  "编写可维护的Javascript纪要"
date:   2013-07-13 20:12:51
categories: javascript
tags:
  - javascript
  - js
  - 代码规范
  - 最佳实践
  - 编程方法
author: "_danZ"
---

# 第一部分： 编程风格

在大型项目开发中，因为项目可读性规范性的需要（就像《编写可维护性的Javascript》一书作者Nicholas Zakas大神所说，他们团队所有成员写出的代码就像是经同一个人之手写出的一样），风格约定要大于个人喜好这一点毋庸置疑，不过什么样才是好的编程风格约定？下面推荐一些实践中沉淀下来的代码规范和最佳实践：

<!--more-->

## 缩进

* 缩进问题和编辑器问题一样是一个因为个人喜好和其他不管值得不值得争执的理由而存在争议的问题，目前存在两个流派，空格流和tab流。个人比较习惯于tab（4个空格距离），在这个问题上团队有协商的余地，不过关键还是两个字：一致。

* 一行截断成两行后，第2行开头缩进2个tab距离。
```javascript
// 截断缩进用2个tab
var str = "abcdefghijkl" +
        "mnopqrstuvwxyz";
```

* =号后面截断到下一行的开头要和前一行=号后面的第一个字母左对齐。

```javascript
// =截断和上一行=号对齐
var s1 = s2
       = s3;
```

* 函数用小写开头（Camel case），构造函数用大写开头(Pascal case)。
* 语句块的写法：()两边留空格，{}两边留空格。

```javascript
// 块语句的空格规范
if (flag === true) {
    alert('Yep!');
} else {
    alert('Nop!');
}
```

## 和NULL做比较

* 只在期待null值的时候和null做比较，比如在一个函数里返回一个null作为期待返回的对象的值。这时可以使用if (res === null) 或者 if( res !== null)来做判断。其他情况尽量不要用null来做检测。注意这里用的是===而不是==号来比较，因为undfined==null的结果是true，而null和undefined有着天壤之别，后面会提到。
* 尽量避免在代码中使用undefined，undefined表示变量未被初始化，或者变量未声明，也表示值是undefined的变量。而typeof运算符在检测以上三种情况时返回的都是undefined

```javascript
// 不好的用法：和undefined比较
var cat;
if(cat === undefined) {
    doSomething();
}
```

尽管上面的代码能够正常运行，但是好的方式是下面这种：

```javascript
// 好的用法，初始化为null
var cat = null;
if(cat === null) {
    doSomething();
}
```

这里不存在如上所述undefined的多种可能的混淆情形。

## 类型、属性判断

分清楚typeof和instanceof的适用情形

* 判断引用类型（比如Array和Date）时，不要用typeof，因为它返回总是object；应该用instanceof操作符，比如instanceof Array
* instanceof不能跨帧使用，因为不同的帧的同名构造函数缺省为不同的构造函数，而typeof是可以跨域使用的。

in的妙用：

* 检测某个DOM元素是否包含某个方法： if(“querySelctorAll” in document)
* 跨帧该怎么检查是否是数组？（注意前面已经说过instanceof不能跨帧使用）使用Object.prototype.toString这个函数，或者使用ECMAScript5引入的Array.isArray()方法：

```javascript
// 判断对象类型是否为数组
function isArray(val){
     if(typeof Array.isArray === 'function'){
          return Array.isArray(val);
     } else {
          return Object.prototype.toString.call(val) === '[object Array]';
     }
}
```

* 判断某个对象是否包含某个属性prop：不使用obj['prop'] == null/undefined，而是使用prop in obj（注意in会遍历原型链，如果只是判断是否是实例方法，则使用下面的hasOwnProperty方法）

hasOwnProperty是Object唯一不会遍历原型链的方法

* 判断某个实例属性用hasOwnProperty，但是因为IE8及更早版本的DOM并非继承自Object，调用DOM的hasOwnProperty方法时应先做判断：

```javascript
// obj为一般对象
if (obj.hasOwnProperty('prop')) {
    ;// ...
}
// obj为DOM对象
if('hasOwnProperty' in obj && obj.hasOwnProperty()) {
    ;// ...
}
```

## 变量提升

到目前为止javascript没有引入块语句作用域，所以有些块语句里声明的变量可能会带来语义上的混淆。因此好的实践是将函数作用域或者对象作用域内所有声明的变量按照隐式变量提升的方式显式放到代码的开头部分。

* 第一句总是声明所有的局部变量
* 合并成一个var声明语句，这样成本更低，代码更短，下载更快。

```javascript
// 所有局部变量合成为一个var声明语句
var s1, s2, s3,
    str1 = 'abcde',
    obj = { 1:'abc', 'xx':'oo' };
```

## ==和===比较

* 对象obj和数字做==比较时，会调用valueOf自动转换成数字进行比较，比如Boolean对象true和2比较 true==2返回false，因为true会首先转换成1
* 字符串和数字比较时，字符串使用Number()转换成数字，比如”0×19″ == 25
* javascript的==隐式转换比较混乱，推荐总是使用===和!==提高代码可读性和可维护性

## 临时包装对象的特性

给变量用内置类型的字面量赋值以后，每次使用这个变量会产生一个临时包装对象，但是这个包装对象是每次用后即弃的。

```javascript
// 临时包装对象用后即弃
var str = "abc";
str.flag = true;
console.log(str.flag);    // undefined
```

上面代码给str添加了一个flag属性并赋值为true，但是这个str临时包装对象赋完值立即弃掉了。因此下一句打印是打印不出这个属性的值的。如果使用new操作符来显式创建这个对象，结果还是一样的：

```javascript
// 不好的写法：显式new一个内置类型包装对象
var str = new String("abc");
str.flag = true;
console.log(str.flag);    // undefined
```

上面的代码可能对开发者产生疑惑，因为第一句貌似是给str赋值了一个String对象。因此，为了避免不必要的代码歧义，建议不要使用new来创建内置类型对象。

# 第二部分：编程实践

前端代码应该尽量避免html/css/javascript代码之间的依赖性，提高各自的独立性。

* 避免使用css的express表达式：（因为浏览器会高频率的计算该表达式，严重影响性能。）

```css
/* css里使用express表达式严重降低性能 */
.box {
    width: expression(document.body.offsetWidth + 'px');
}
```

* 不应在javascript代码中直接修改css样式，而是将所有css样式放在css文件中，并组织在类中，只在javascript代码中修改DOM的className属性，通过类的添加和删除来改变元素样式。
* 避免0级DOM绑定（在html中直接给元素onxxx属性赋值）： 1.可能onclick=”doSomething”的这个doSomething函数还没有加载，此时会报错或者点击没有响应； 2.加深了两个UI层（html和javascript）的耦合性，典型的紧耦合代码
* 一个全兼容的事件绑定函数：

```javascript
// 全兼容的事件绑定函数
function addListener(target, type, handler) {
    if (target.addEventListener) {
        target.addEventListener(type, handler, false);
    } else if (target.attachEvent) {    // IE
        target.attachEvent('on' + type, handler);
    } else {
        target['on' + type] = handler;
    }
}
```

不要去破坏别人的模块。这里推荐一个无破坏性的命名空间定义方式：

```javascript
// 无破坏性的命名空间定义方式
var yourGlobal = {
    namespace: function(ns) {
        var parts = ns.split('.'),
            object = this,
            i, len; 
        for (i = 0, len = parts.length; i < len; i++) {
            if (!object[parts[i]]) {
                object[parts[i]] = {};
            }
            object = object[parts[i]];
        }
        return object;
    }
}
```

* 隔离应用逻辑：

下面的代码直接将处理放到了handle函数里，如果在没有点击的情况下想要调用它就麻烦了：

```javascript
// 不好的写法
function handleClick(event) {
    console.log(event.clientX, event.clientY);
}
```

将应用逻辑分离后的代码：

```javascript
// 应用逻辑和事件处理隔离
function handleClick(event) {
    logPosition(event);
} 
function logPosition(event) {
    console.log(event.clientX, event.clientY);
}
```

不要分发事件对象

上面的代码处理还是有问题，因为logPosition的参数带有应用逻辑不需要的信息，因此继续改进：

```javascript
// 改进：避免分发事件对象
function handleClick(event) {
    logPosition(event.clientX, event.clientY);
} 
function logPosition(x, y) {
    console.log(x, y);
}
```

对象的保护：Object的方法preventExtension、seal和freeze（保护强度递增）

```javascript
// 对象的保护
'use strict'
// 不能增加属性
Object.preventExtension(cat);
console.log(Object.isExtensible(cat));
cat.age = 1;    // error.
// 不能增删属性
Object.seal(cat);
console.log(Object.isExtensible(cat));
console.log(Object.isSealed(cat));
delete cat.name;    // error.
cat.age = 1;    // error.
// 不能增删改
Object.freeze(cat);
console.log(Object.isExtensible(cat));
console.log(Object.isSealed(cat));
console.log(Object.isFrozen(cat));
cat.name = "Kitty";    // error.
delete cat.name;    // error.
cat.age = 1;    // error.
```


注意：以上error如果不使用strict模式的话会被浏览器默默吞掉。

* 尽量使用特性检测代替userAgent检测，比如if(document.getElementById()) { … } 而不是去判断用户代理if(navigator.userAgent.indexof(‘MSIE 7′) > -1)；特性检测的优点在于不依赖于浏览器。

常用文件结构：

* **src, build, lib, doc, release, test** （一般src和test里的文件名存在一一对应的关系）
