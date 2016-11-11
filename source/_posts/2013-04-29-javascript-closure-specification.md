---
layout: post
posted: 'test'
title:  "Javascript闭包详解"
date:   2013-04-29 10:09:13
categories: javascript
tags:
  - closure
  - javascript
  - js
  - 闭包
author: "_danZ"
---

# 闭包的概念

之前微博上有人提了个问题，到底什么是闭包？
javascript秘密花园中有这么一段

> 闭包是 JavaScript 一个非常重要的特性，这意味着当前作用域总是能够访问外部作用域中的变量。 因为函数 是 JavaScript 中唯一拥有自身作用域的结构，因此闭包的创建依赖于函数。

<!--more-->

维基闭包词条阐释了其本质

> 在计算机科学中，闭包（Closure）是词法闭包（Lexical Closure）的简称，是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。所以，有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的实体。

维基的解释或许有点抽象，但是闭包作为一个概念理解起来并不困难，你可以看做是内部函数的作用域延伸到了外部环境中。

# javascript中的闭包

js是词法作用域语言，查找内部函数名字是总是从当前作用域开始，沿着作用域链一层层的向上查找。但是在函数外部想要修改函数内部的变量一般是不可能的，因为作用域链不能向下查找，所以它找不到要访问的函数内部变量的名字。而闭包就是把这种不可能变为可能，把函数内部的作用域扩展到函数外部来。

来看这样一个例子：

```javascript
function init() {
    var name = "DEFINITELY MAYBE";
    function displayName() {
        alert(name);
    }
    displayName();
}
init();
```

init()执行时调用内部的函数displayName()，displayName()用到了name这个变量，执行结束以后，name变量不会继续在内存里驻留，而是被GC回收。这段代码和闭包有什么关系呢？它本身不是一个闭包，它本身只是一个功能函数。我们可以把它变成一个闭包，只需要把定义在init函数内部的displayName函数当做一个变量返回：

```javascript
function makeFunc() {
    var name = "DEFINITELY MAYBE";
    function displayName() {
        alert(name);
    }
    return displayName;
}
var myFunc = makeFunc();
myFunc();
```

makeFunc()执行完毕，返回的函数displayName就是一个闭包，因为它包含它所在作用域的上下文信息，它是displayName这个函数以及它的相关引用环境组合而成的实体。上一段代码在makeFunc()执行结束以后，它内部定义的变量name会被GC清理，但是闭包因为包含了上下文信息，所以name这个变量会一直存在于内存中。所以在最后一句myFunc()的调用中，alert(name);能够成功执行，因为这个name并没有被gc回收，而是一直伴随着闭包而存在。

在js中，闭包可以实现类定义中私有属性的隐藏。我们看以下一段代码，它定义了一个类Monster，其中的隐藏字段可以通过get和set方法访问，但是用户代码无法直接访问它（undefinded）。

```javascript
function Monster(){
    var _name;
    function getName(){
        return _name;
    }
    function setName(name){
        _name = name;
    }
    return {
        getName: getName,
        setName: setName,
    }
}
 
var godzilla = Monster();
godzilla.setName('Godzilla');
```

用户可以通过godzilla变量所引用的闭包访问其作用域上下文环境的name属性，但是不能通过用户代码直接访问name变量，这样从面向对象的角度看是隐藏了内部属性。

通过函数直接量构造一个闭包产生一个singletonMonster单例：

```javascript
var singletonMonster = (function(){
    var _name = 'Godzilla';
    function getName(){
        return _name;
    }
    function setName(name){
        _name = name;
    }
    return {
        getName: getName,
        setName: setName,
    }
})();
```

虽然以上代码每次返回的实例内部的属性是相同的引用，改变某个实例的属性会同时改变另一个，但是却并不是正确意义上的单例的构造方式，因为每次调用返回的对象实例引用不是同一个引用。正确的带私有属性的单例构造方式可以参考以下两种：

方式一：

```javascript
function Singleton() {
    var _instance;
    var _name = 'Godzilla';
    Singleton = function Singleton () {
        return _instance;
    }
    Singleton.prototype = this;
    _instance = new Singleton();
    _instance.constructor = Singleton;
    _instance.setName = function(name) {
        _name = name;
    };
    _instance.getName = function() {
        return _name;
    }
    return _instance;
}
```

方式二：

```javascript
var Singleton;
(function() {
    var _instance; 
    var _name = 'Godzilla';
    Singleton = function Singleton() {
        if (_instance) {
            return _instance;
        }
        _instance = this;
        this.getName = function() {
            return _name;
        };
        this.setName = function(name) {
            _name = name;
        };
    }
})();
```

以后有机会再详细讨论一下javascript中单例的各种构建方式及其优缺点。

# 闭包的内存效率

闭包并没有多么高深，实际上闭包在js中需要谨慎使用，一般在类/对象编程中使用prototype而不是闭包，因为闭包是直接将属性添加到构造函数中，在构造实例时，会把构造函数的所有属性都实例化一遍，同一个类的同一个方法在每个实例中都存储了一个相同的版本，这样会造成资源的严重浪费。解决这个问题就是把实例方法放到prototype中，而实例属性则放在类的构造函数中。这里暂不讨论prototype的作用，prototype是js另一个重要特性，和基于class的语言不同，js是基于prototype的语言，类的继承必须通过prototype实现。

最近在网上看到一个说法，优化闭包的内存效率，意思只要这个闭包存在，闭包所包含的上下文环境中的每个变量都不会被GC收集，都需要手动清空为null或undefined，实在是大雾。GC的收集机制不是看取值，而是看是否被引用，所以对于以下这个例子，在Foo函数体内返回语句之前加上_noUse = null实际上没有意义。GC只要看到某个局部变量没有被引用了，这个函数执行结束就在下次垃圾回收时自动将该变量清理掉。

```javascript
function Foo(){
    var _bar = 'bar';
    var _noUse = 2;
    function getFoo(){
        return _bar;
    }
    function setFoo(sm){
        _bar = sm;
    }
    _noUse = null;  // no need to add this statement.
    return {
        getFoo: getFoo,
        setFoo: setFoo,
    }
}
```

# 常见的错误用法

援引mozilla开发者网络上的一个典型例子

```html
<p id="help">Helpful notes will appear here</p>
<p>E-mail: <input type="text" id="email" name="email"></p>
<p>Name: <input type="text" id="name" name="name"></p>
<p>Age: <input type="text" id="age" name="age"></p>
```

js代码:

```javascript
function showHelp(help){
    document.getElementById('help').innerHTML = help;
}
function setupHelp(){
    var helpText = [
        {'id': 'email', 'help': 'Your e-mail address'},
        {'id': 'name', 'help': 'Your full name'},
        {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];
    for(var i = 0, l = helpText.length; i < l; i++){
        var item = helpText[i];
        document.getElementById(item.id).onfocus = function(){
            showHelp(item.help);
        }
    }
}
setupHelp();
```

上面这个例子，也许你会认为对于每个input都绑定了自己的一个事件响应，显示相对应的提示信息。代码的执行结果也许会让你失望，所有的input标签都绑定到同一个事件响应，三个input的提示信息是一样的，都是最后一条。

记住在循环中使用闭包是需要谨慎小心的，这里onfocus实际上是用一个闭包赋值，这个闭包所引用的执行环境就是SetupHelp()函数的作用域，三个绑定函数所引用的执行环境实际上都是这个SetupHelp()内部作用域，这时闭包里的执行代码showHelp(item.help)每次调用都是SetupHelp()函数中定义的item变量，而item在SetupHelp()执行结束的时候已经被赋值为helpText[2]了，所以每次显示的都是helpText[2].help的内容。

上面强调的是闭包，如果我们从执行角度来看上面代码的问题，可能来得更清晰。从执行过程来看，在绑定的事件处理的过程中包裹showHelp()的匿名函数并没有执行，所以showHelp的实参并没有确定，而是到执行的时候去确定，这时根据闭包的原理，item用的是已经改变为helpText[2]的item，这时不论哪个事件绑定函数取到的实参都是这个值，导致了错误上述结果。

要解决这个问题，我们可以为每个事件绑定一个单独的闭包，这样它们各自的执行环境不同，引用的item值也不同。

```javascript
function showHelp(help){
    document.getElementById('help').innerHTML = help;
}
function makeHelpCallback(help){
    return function(){
        showHelp(help);
    };
}
function setupHelp(){
    var helpText = [
        {'id': 'email', 'help': 'Your e-mail address'},
        {'id': 'name', 'help': 'Your full name'},
        {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];
    for(var i = 0, l = helpText.length; i < l; i++){
        var item = helpText[i];
        document.getElementById(item.id).onfocus = makeHelpCallback(item.help);
    }
}
setupHelp();
```

从执行角度观察，makeHelpCallback()在onfocus绑定过程中执行，返回的函数就是所要绑定的函数，是一个闭包, 这个闭包的上下文环境是makeHelpCallback(item.help)的作用域，每次绑定事件时对应返回的闭包都是一个不同的makeHelpCallback()作用域（因为item.help不同）。item.help在执行过程中已经被传递到makeHelpCallback内层的匿名函数闭包中，这时help是不变的，不会因为后面循环代码中item改变而改变。

作为一个简单的解决方案，也可以直接给ShowHelp加上一个匿名包裹函数：

```javascript
document.getElementById(item.id).onfocus = function(e){
    return function(){
        showHelp(e);
    }
}(item.help);
```

# 闭包实现函数工厂

上面的makeHelpCallback实际上实现了一个函数工厂，传递不同的参数返回对应的函数。每个闭包执行函数都是独特的，因为每次调用的时候传递的参数不同。

```javascript
function makeHunter(monster){
    return function(){
        console.log('Hah, you\'re a ' + monster + 'hunter!');
    }
}
var gozillaHunter = makeHunter('gozilla');
var zombieHunter = makeHunter('zombie');
console.log(gozillaHunter());
console.log(zombieHunter());
```

上面的函数工厂根据不同的参数产生不同的Hunter函数。注意虽然产生的闭包共享同一个函数体，即makeHunter函数体，但是每一个闭包对应的上下文坏境是不同的，在闭包gozillaHunter的上下文中，monster的值是’gozilla’，而在闭包zombieHunter的对应上下文中，monster的值是’zombie’，它们是两个完全不同的上下文坏境。
